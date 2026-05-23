---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "AI Infra 高并发与推理性能优化"
postSlug: agent-05-infra-performance
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: AI Infra 高并发与推理性能优化"
---

---
tags: [LLM, Infra, Performance, vLLM, Streaming]
created: 2026-05-22
---

# AI Infra 高并发与推理性能优化

> 阿里一面必问：**"如果流量翻 100 倍，底层模型推理和系统怎么撑住？"**  
> 本文从原理到代码，覆盖推理性能优化的核心知识体系。

---

## 一、LLM 推理的两个阶段

```
Prefill 阶段（并行）          Decode 阶段（自回归）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
输入所有 token → 并行计算    逐个 token 生成（必须串行）
  KV Cache 首次生成            利用已有 KV Cache
  延迟 = TTFT                  延迟 = TPS × 输出长度
  计算密集型（GPU 利用率高）    显存密集型（KV Cache 占用大）
```

**TTFT（Time to First Token）**：用户感知的"响应速度"  
**TPS（Tokens per Second）**：生成流畅度

---

## 二、KV Cache 与 PagedAttention

### 2.1 传统 KV Cache 的问题

每次推理时，Attention 机制需要保存所有历史 token 的 Key 和 Value 张量（KV Cache）。

**传统方案（预分配连续显存）**：

```
显存布局（假设 max_length=2048）：

请求 A: [░░░░░░░░░░░░████████████████] 预分配2048，实际用了800（内部碎片：1248）
请求 B: [████████████████████░░░░░░░░] 预分配2048，实际用了1200（内部碎片：848）
空闲区: [     (碎片化，无法分配给新请求)    ]
```

**显存利用率 < 30%** → 同时处理的请求数（Batch Size）极低 → 吞吐量差。

### 2.2 PagedAttention 解决方案

借鉴操作系统的**虚拟内存分页**：将 KV Cache 划分为固定大小的 Block（页），按需动态分配。

```
物理显存（Block Pool）：
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ B0  │ B1  │ B2  │ B3  │ B4  │ B5  │ B6  │ B7  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

请求 A 的逻辑 KV Cache → [B0, B2, B5]    （非连续物理页）
请求 B 的逻辑 KV Cache → [B1, B3, B4]
新请求 C 可以使用        [B6, B7]

Block 内部碎片 ≤ 1个 Block 大小（极小），外部碎片为零
```

**工程收益**：
- Batch Size 提升 **2-4x** → 吞吐量（Throughput）大幅提升
- Beam Search 多个候选序列可共享相同历史页（Copy-on-Write）

---

## 三、Continuous Batching（动态批处理）

```
传统 Static Batching：
时间线: [请求A──────────完成][请求B──────完成][请求C──完成]
                                              ↑ GPU 空闲等待

Continuous Batching：
时间线: [请求A──────────完成]
        [请求B──────完成]     ← A完成后立刻插入新请求D
        [请求C──完成]
        [请求D────────]       ← GPU 始终满负载
```

**核心**：当某个请求生成了 EOS token，立刻在下一个 iteration 插入新请求，无需等待整个 Batch 完成。

---

## 四、流式输出（SSE）+ 背压控制

```python
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
from openai import AsyncOpenAI
import asyncio

app = FastAPI()
async_client = AsyncOpenAI()


async def stream_generator(prompt: str, request: Request):
    """
    SSE 流式输出生成器
    关键：检测客户端断连（背压控制）
    """
    try:
        stream = await async_client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            stream=True,
        )

        async for chunk in stream:
            # 检测客户端是否已断开连接（背压的核心）
            if await request.is_disconnected():
                print("客户端已断开，停止生成")
                await stream.close()  # 关键：通知服务端停止生成，节省 GPU 资源
                break

            delta = chunk.choices[0].delta.content
            if delta:
                # SSE 格式：data: <content>\n\n
                yield f"data: {delta}\n\n"

        yield "data: [DONE]\n\n"

    except asyncio.CancelledError:
        print("请求被取消")
        raise


@app.post("/stream")
async def stream_endpoint(request: Request):
    body = await request.json()
    prompt = body.get("prompt", "")

    return StreamingResponse(
        stream_generator(prompt, request),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "X-Accel-Buffering": "no",  # 禁用 Nginx 缓冲
        },
    )
```

---

## 五、Token Bucket 限流器

```python
import asyncio
import time
from dataclasses import dataclass, field


@dataclass
class TokenBucket:
    """
    令牌桶限流算法
    
    - 每秒固定产生 rate 个令牌
    - 桶容量为 capacity（允许短时间突发）
    - 请求到达时消耗令牌，无令牌则等待或拒绝
    """
    rate: float       # 每秒产生的令牌数
    capacity: float   # 桶的最大容量
    _tokens: float = field(init=False)
    _last_refill: float = field(init=False)
    _lock: asyncio.Lock = field(init=False)

    def __post_init__(self):
        self._tokens = self.capacity
        self._last_refill = time.monotonic()
        self._lock = asyncio.Lock()

    def _refill(self):
        """根据时间流逝补充令牌"""
        now = time.monotonic()
        elapsed = now - self._last_refill
        self._tokens = min(self.capacity, self._tokens + elapsed * self.rate)
        self._last_refill = now

    async def acquire(self, tokens: int = 1, timeout: float = 10.0) -> bool:
        """
        获取令牌（异步等待）
        
        tokens: 本次请求消耗的令牌数（可按请求大小动态计算）
        timeout: 最大等待时间（秒）
        返回 False 表示超时拒绝
        """
        deadline = time.monotonic() + timeout
        async with self._lock:
            while True:
                self._refill()
                if self._tokens >= tokens:
                    self._tokens -= tokens
                    return True
                # 计算还需要等待多久
                wait_time = (tokens - self._tokens) / self.rate
                if time.monotonic() + wait_time > deadline:
                    return False  # 超时，拒绝请求
                await asyncio.sleep(min(wait_time, 0.1))


# 全局限流器：每秒允许 10 个 LLM 请求，最大突发 20
llm_rate_limiter = TokenBucket(rate=10, capacity=20)


async def rate_limited_llm_call(prompt: str) -> str:
    """带限流的 LLM 调用"""
    allowed = await llm_rate_limiter.acquire(tokens=1, timeout=5.0)
    if not allowed:
        raise Exception("服务繁忙，请稍后重试（限流）")

    response = await async_client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
    )
    return response.choices[0].message.content
```

---

## 六、Agent 状态持久化（中断恢复）

长时间运行的 Agent（如后台分析任务）必须支持**中断和恢复**，不能依赖内存。

```python
import json
import os
from dataclasses import dataclass, field, asdict
from datetime import datetime
from enum import Enum
from typing import Any


class StepStatus(str, Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"
    FAILED = "failed"


@dataclass
class AgentStep:
    step_id: int
    description: str
    status: StepStatus = StepStatus.PENDING
    result: Any = None
    error: str | None = None
    started_at: str | None = None
    finished_at: str | None = None


@dataclass
class AgentCheckpoint:
    """Agent 执行状态的完整快照"""
    session_id: str
    goal: str
    steps: list[AgentStep]
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
    last_updated: str = field(default_factory=lambda: datetime.now().isoformat())
    metadata: dict = field(default_factory=dict)


class PersistentAgent:
    """
    支持持久化和恢复的 Agent
    每个步骤完成后立即写入磁盘，中断后可从上次断点继续
    
    思路类似 LangGraph 的 Checkpoint 机制
    """

    def __init__(self, checkpoint_dir: str = ".agent_checkpoints"):
        self.checkpoint_dir = checkpoint_dir
        os.makedirs(checkpoint_dir, exist_ok=True)

    def _checkpoint_path(self, session_id: str) -> str:
        return os.path.join(self.checkpoint_dir, f"{session_id}.json")

    def save(self, checkpoint: AgentCheckpoint):
        """保存检查点（每步执行后调用）"""
        checkpoint.last_updated = datetime.now().isoformat()
        path = self._checkpoint_path(checkpoint.session_id)
        with open(path, "w") as f:
            json.dump(asdict(checkpoint), f, ensure_ascii=False, indent=2, default=str)

    def load(self, session_id: str) -> AgentCheckpoint | None:
        """加载已有检查点（恢复中断的任务）"""
        path = self._checkpoint_path(session_id)
        if not os.path.exists(path):
            return None
        with open(path) as f:
            data = json.load(f)
        steps = [AgentStep(**s) for s in data.pop("steps")]
        return AgentCheckpoint(steps=steps, **data)

    def run_or_resume(
        self,
        session_id: str,
        goal: str,
        step_definitions: list[str],
        step_executor,  # Callable[[str, dict], Any]
    ) -> AgentCheckpoint:
        """
        运行 Agent，自动处理断点恢复
        
        - 如果存在已有 checkpoint，从上次中断处继续
        - 如果不存在，从头开始
        """
        # 尝试加载已有 checkpoint
        checkpoint = self.load(session_id)
        if checkpoint:
            print(f"[Resume] 从检查点恢复，session={session_id}")
            # 找到第一个未完成的步骤
            completed = sum(1 for s in checkpoint.steps if s.status == StepStatus.DONE)
            print(f"  已完成 {completed}/{len(checkpoint.steps)} 步")
        else:
            print(f"[New] 创建新会话，session={session_id}")
            checkpoint = AgentCheckpoint(
                session_id=session_id,
                goal=goal,
                steps=[
                    AgentStep(step_id=i, description=desc)
                    for i, desc in enumerate(step_definitions)
                ],
            )
            self.save(checkpoint)

        # 执行未完成的步骤
        context = {}  # 步骤间传递的上下文
        for step in checkpoint.steps:
            if step.status == StepStatus.DONE:
                context[step.step_id] = step.result
                continue  # 跳过已完成的步骤

            print(f"\n[Step {step.step_id}] {step.description}")
            step.status = StepStatus.RUNNING
            step.started_at = datetime.now().isoformat()
            self.save(checkpoint)  # 标记为运行中

            try:
                result = step_executor(step.description, context)
                step.result = result
                step.status = StepStatus.DONE
                context[step.step_id] = result
                print(f"  ✓ 完成")
            except Exception as e:
                step.status = StepStatus.FAILED
                step.error = str(e)
                print(f"  ✗ 失败: {e}")
                self.save(checkpoint)
                raise  # 失败时不继续，保留检查点供恢复

            step.finished_at = datetime.now().isoformat()
            self.save(checkpoint)  # 每步完成后立即持久化

        return checkpoint


# 测试
def mock_executor(description: str, context: dict) -> str:
    """模拟步骤执行（实际中会调用 LLM 或工具）"""
    import time
    time.sleep(0.1)
    return f"完成：{description}"


agent = PersistentAgent()
result = agent.run_or_resume(
    session_id="analysis-20260522",
    goal="分析代码库并生成报告",
    step_definitions=["扫描代码文件", "提取关键函数", "生成文档", "写入报告"],
    step_executor=mock_executor,
)
print(f"\n所有步骤状态：{[s.status for s in result.steps]}")
```

---

## 七、并发请求管理器（Semaphore 控制）

```python
import asyncio
from openai import AsyncOpenAI

async_client = AsyncOpenAI()


class ConcurrentLLMPool:
    """
    并发 LLM 请求池
    - 用 Semaphore 限制最大并发数，避免超出 API Rate Limit
    - 带队列等待和超时处理
    """
    def __init__(self, max_concurrent: int = 10, timeout: float = 30.0):
        self._semaphore = asyncio.Semaphore(max_concurrent)
        self.timeout = timeout
        self._active = 0
        self._total = 0

    async def call(self, prompt: str, **kwargs) -> str:
        """带并发控制的 LLM 调用"""
        async with self._semaphore:
            self._active += 1
            self._total += 1
            try:
                return await asyncio.wait_for(
                    self._do_call(prompt, **kwargs),
                    timeout=self.timeout,
                )
            except asyncio.TimeoutError:
                raise TimeoutError(f"LLM 调用超时（>{self.timeout}s）")
            finally:
                self._active -= 1

    async def _do_call(self, prompt: str, **kwargs) -> str:
        response = await async_client.chat.completions.create(
            model=kwargs.get("model", "gpt-4o-mini"),
            messages=[{"role": "user", "content": prompt}],
            temperature=kwargs.get("temperature", 0.7),
        )
        return response.choices[0].message.content

    async def batch_call(self, prompts: list[str]) -> list[str]:
        """批量并发调用，自动限流"""
        tasks = [self.call(p) for p in prompts]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        # 将异常转换为错误字符串，避免单个失败影响整体
        return [
            str(r) if isinstance(r, Exception) else r
            for r in results
        ]


# 测试：批量处理 50 个请求，最多 10 个并发
async def main():
    pool = ConcurrentLLMPool(max_concurrent=10)
    prompts = [f"用一句话解释概念#{i}" for i in range(20)]
    results = await pool.batch_call(prompts)
    print(f"处理完成 {len(results)} 个请求")

# asyncio.run(main())
```

---

## 八、性能优化优先级矩阵

| 优化手段 | 收益 | 实现难度 | 推荐优先级 |
|---------|------|---------|-----------|
| **Continuous Batching** | 极高（吞吐 3-5x） | 使用 vLLM 即可 | ⭐⭐⭐⭐⭐ |
| **Streaming + 背压** | 高（用户体验） | 中 | ⭐⭐⭐⭐⭐ |
| **KV Cache 复用**（相同前缀） | 高（延迟降低） | 低（Prompt Cache） | ⭐⭐⭐⭐⭐ |
| **限流（Rate Limiting）** | 中（稳定性） | 低 | ⭐⭐⭐⭐ |
| **INT8 量化** | 高（显存减半） | 中（需验证精度） | ⭐⭐⭐⭐ |
| **Speculative Decoding** | 高（延迟降低 2x） | 高（需要草稿模型） | ⭐⭐⭐ |
| **Tensor Parallelism** | 极高（多卡） | 高（需要多 GPU） | ⭐⭐⭐ |
| **状态持久化** | 中（可靠性） | 中 | ⭐⭐⭐ |
| **INT4/GPTQ 量化** | 高（显存极省） | 高（精度损失风险） | ⭐⭐ |

---

*← [[agent-04-eval-system]] | 下一篇 → [[agent-06-observability]]*
