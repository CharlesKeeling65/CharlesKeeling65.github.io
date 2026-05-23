---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "Agent Planning & Reasoning 规划与推理"
postSlug: agent-01-planning-reasoning
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: Agent Planning & Reasoning 规划与推理"
---

---
tags: [LLM, Agent, Planning, ReAct, CoT]
created: 2026-05-22
---

# Agent Planning & Reasoning 规划与推理

> 规划能力是 Agent 的"大脑"。核心工程问题：**如何让模型拆解复杂目标，并在执行过程中不跑偏、不死循环？**

---

## 一、Zero-shot / Few-shot Prompting

### 原理

最基础的指令形式。Zero-shot 直接给任务描述；Few-shot 在 Prompt 中插入若干 `(Input, Output)` 样例，让模型"学"输出格式。

**工程核心**：Few-shot 的质量 > 数量。3 个精心挑选的例子往往优于 10 个随意的例子。

### 适用场景
- 格式固定的提取任务（实体识别、分类）
- 不需要多步推理的单轮任务

```python
from openai import OpenAI

client = OpenAI()

# Few-shot：让模型输出结构化 JSON
FEW_SHOT_PROMPT = """
你是一个意图分类器，将用户输入分类为以下类别之一：
code_explain | code_refactor | code_debug | other

示例：
用户: 帮我解释这段代码是什么意思
输出: {"intent": "code_explain", "confidence": 0.95}

用户: 这段代码太乱了，帮我重构一下
输出: {"intent": "code_refactor", "confidence": 0.90}

用户: 我的代码报错了，TypeError: ...
输出: {"intent": "code_debug", "confidence": 0.92}

现在分类以下输入，只输出 JSON：
用户: {user_input}
输出:"""

def classify_intent(user_input: str) -> dict:
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "user", "content": FEW_SHOT_PROMPT.format(user_input=user_input)}
        ],
        temperature=0,  # 分类任务用 0 温度，保证稳定性
    )
    import json
    return json.loads(response.choices[0].message.content)

# 测试
result = classify_intent("帮我把这个函数改成异步的")
print(result)  # {"intent": "code_refactor", "confidence": 0.88}
```

---

## 二、Chain of Thought (CoT)

### 原理

通过 `"Think step by step"` 或显式步骤 Prompt，强制模型输出**中间推理过程**再给出答案。研究表明，CoT 能显著提升复杂推理任务的准确率。

**工程代价**：
- 输出 Token 数大幅增加（通常 3-5x），延迟上升
- TTFT（Time To First Token）不变，但 Total Latency 增加

### Self-Consistency CoT（进阶）

同一问题多次采样（temperature > 0），对多个 CoT 路径的答案投票。牺牲延迟换准确率，适合离线批处理任务。

```python
from openai import OpenAI
from collections import Counter

client = OpenAI()

def cot_single(question: str) -> str:
    """单次 CoT 推理"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": "你是一个严谨的分析助手。解答问题时，请先一步步思考推理过程，最后在 [答案] 标记后给出最终答案。"
            },
            {"role": "user", "content": question}
        ],
        temperature=0.7,
    )
    return response.choices[0].message.content


def self_consistency_cot(question: str, n_samples: int = 5) -> str:
    """
    Self-Consistency CoT：多次采样后投票
    适合高精度要求的离线任务，延迟 = n_samples × 单次延迟
    """
    answers = []
    for _ in range(n_samples):
        full_response = cot_single(question)
        # 提取 [答案] 后的内容
        if "[答案]" in full_response:
            answer = full_response.split("[答案]")[-1].strip()
        else:
            answer = full_response.strip()
        answers.append(answer)
    
    # 多数投票
    vote_result = Counter(answers).most_common(1)[0]
    print(f"投票结果: {vote_result[0]}（{vote_result[1]}/{n_samples} 票）")
    return vote_result[0]


# 测试
result = self_consistency_cot(
    "一个 Agent 系统每秒处理 100 个请求，每个请求平均消耗 2000 tokens，"
    "gpt-4o 的价格是 $2.5/1M tokens，每天运行 8 小时，每月成本是多少美元？"
)
```

---

## 三、ReAct (Reason + Act)

### 原理

**ReAct = Reason（思考） + Act（行动）**，模式为：

```
Thought: 我需要先查询用户的订单状态
Action: query_order_status(order_id="12345")
Observation: 订单状态为"已发货"，预计明天到达
Thought: 已获取信息，可以回答用户了
Action: final_answer("您的订单已发货，预计明天到达")
```

### 工程局限（重点）
1. **上下文膨胀**：每次循环都要携带完整历史，token 消耗随步数线性增长
2. **容错率低**：第 N 步的错误会污染后续所有步骤（错误传播）
3. **最大步数限制**：必须设置 `max_steps`，否则可能无限循环

```python
import json
from openai import OpenAI
from typing import Callable

client = OpenAI()

# ── 工具定义 ──────────────────────────────────────────────
def search_docs(query: str) -> str:
    """模拟文档检索工具"""
    # 真实场景：调用向量数据库
    return f"检索到关于'{query}'的文档：vLLM 使用 PagedAttention 管理 KV Cache..."

def get_code_context(file_path: str) -> str:
    """模拟代码读取工具"""
    return f"# {file_path} 的内容\ndef example_function():\n    pass"

def run_test(test_name: str) -> str:
    """模拟测试运行工具"""
    return f"测试 {test_name} 通过 ✓"

# 工具注册表
TOOLS: dict[str, Callable] = {
    "search_docs": search_docs,
    "get_code_context": get_code_context,
    "run_test": run_test,
}

TOOLS_SCHEMA = [
    {
        "type": "function",
        "function": {
            "name": "search_docs",
            "description": "搜索技术文档",
            "parameters": {
                "type": "object",
                "properties": {"query": {"type": "string", "description": "搜索关键词"}},
                "required": ["query"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_code_context",
            "description": "读取指定文件的代码内容",
            "parameters": {
                "type": "object",
                "properties": {"file_path": {"type": "string", "description": "文件路径"}},
                "required": ["file_path"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "run_test",
            "description": "运行指定的测试用例",
            "parameters": {
                "type": "object",
                "properties": {"test_name": {"type": "string", "description": "测试名称"}},
                "required": ["test_name"],
            },
        },
    },
]


def react_agent(user_query: str, max_steps: int = 10) -> str:
    """
    ReAct Agent 实现
    - 使用 OpenAI Function Calling 实现 Thought/Action/Observation 循环
    - max_steps 防止死循环
    """
    messages = [
        {
            "role": "system",
            "content": (
                "你是一个工程助手 Agent。遇到需要查询信息的问题，请调用工具获取信息后再回答。"
                "思考过程要简洁，每次只调用最必要的工具。"
            ),
        },
        {"role": "user", "content": user_query},
    ]

    for step in range(max_steps):
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=TOOLS_SCHEMA,
            tool_choice="auto",
        )
        msg = response.choices[0].message
        messages.append(msg)  # 把 assistant 的回复加入历史

        # 没有工具调用 → 模型已给出最终答案
        if not msg.tool_calls:
            print(f"[ReAct] 共执行 {step + 1} 步")
            return msg.content

        # 处理工具调用
        for tool_call in msg.tool_calls:
            func_name = tool_call.function.name
            func_args = json.loads(tool_call.function.arguments)

            print(f"  [Step {step + 1}] Action: {func_name}({func_args})")

            # 执行工具（关键：捕获异常，避免单个工具失败崩溃整个 Agent）
            try:
                observation = TOOLS[func_name](**func_args)
            except Exception as e:
                observation = f"工具执行失败: {e}"

            print(f"  [Step {step + 1}] Observation: {observation[:80]}...")

            # 将工具结果注入消息历史
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": observation,
            })

    return "已达到最大步数限制，无法完成任务"


# 测试
answer = react_agent("vLLM 的 PagedAttention 原理是什么？请查阅文档后解释。")
print(answer)
```

---

## 四、Plan-and-Execute（规划与执行）

### 原理

将"想"和"做"解耦为两个独立的 Agent：

```
Planner Agent → [Task 1, Task 2, Task 3]
                        ↓
            Executor Agent（逐个执行）
                        ↓
               每步结果汇总 → 最终答案
```

**优势（vs ReAct）**：
- **可追踪**：执行计划是结构化的，可以记录每步状态（Observability 友好）
- **可并行**：无依赖关系的 Task 可以并发执行
- **容错**：单步失败不影响其他步骤

**适用**：复杂的多步骤任务，如代码重构、报告生成

```python
import json
import asyncio
from dataclasses import dataclass, field
from openai import OpenAI

client = OpenAI()


@dataclass
class Task:
    id: int
    description: str
    depends_on: list[int] = field(default_factory=list)
    result: str | None = None
    status: str = "pending"  # pending | running | done | failed


def planner(user_goal: str) -> list[Task]:
    """
    Planner Agent：将复杂目标分解为有序任务列表
    输出严格的 JSON，便于后续解析
    """
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": """你是一个任务规划专家。将用户目标分解为 3-6 个具体可执行的子任务。
输出格式（严格 JSON）：
{
  "tasks": [
    {"id": 1, "description": "任务描述", "depends_on": []},
    {"id": 2, "description": "任务描述", "depends_on": [1]}
  ]
}
depends_on 填写前置任务的 id，没有前置则为空数组。""",
            },
            {"role": "user", "content": f"目标：{user_goal}"},
        ],
        temperature=0,
        response_format={"type": "json_object"},
    )
    data = json.loads(response.choices[0].message.content)
    return [Task(**t) for t in data["tasks"]]


def executor(task: Task, context: dict[int, str]) -> str:
    """
    Executor Agent：执行单个任务
    context 包含前置任务的结果，用于信息传递
    """
    # 构建前置任务的上下文
    context_str = "\n".join(
        [f"任务{k}的结果：{v}" for k, v in context.items()]
    ) if context else "无前置信息"

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": "你是一个执行专家，负责完成具体的子任务。给出详细可用的结果。",
            },
            {
                "role": "user",
                "content": f"前置任务结果：\n{context_str}\n\n当前任务：{task.description}",
            },
        ],
        temperature=0.3,
    )
    return response.choices[0].message.content


def plan_and_execute(user_goal: str) -> str:
    """完整的 Plan-and-Execute 流程"""
    print(f"[Planner] 分解目标：{user_goal}")
    tasks = planner(user_goal)

    for t in tasks:
        print(f"  Task {t.id}: {t.description} (依赖: {t.depends_on})")

    results: dict[int, str] = {}

    # 按依赖顺序执行（简单拓扑排序）
    completed = set()
    while len(completed) < len(tasks):
        for task in tasks:
            if task.id in completed:
                continue
            # 检查依赖是否都完成
            if all(dep in completed for dep in task.depends_on):
                context = {dep: results[dep] for dep in task.depends_on}
                print(f"\n[Executor] 执行 Task {task.id}: {task.description}")
                task.result = executor(task, context)
                results[task.id] = task.result
                completed.add(task.id)
                print(f"  结果：{task.result[:100]}...")

    # 汇总最终结果
    final_context = "\n\n".join([f"## Task {i}: {t.description}\n{t.result}" for i, t in enumerate(tasks, 1)])
    summary_response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "根据各子任务的结果，综合给出最终答案。"},
            {"role": "user", "content": f"目标：{user_goal}\n\n各步骤结果：\n{final_context}"},
        ],
    )
    return summary_response.choices[0].message.content


# 测试
result = plan_and_execute(
    "分析 Python 的 asyncio 事件循环原理，并写出一个包含超时控制的并发请求示例"
)
print("\n[最终结果]\n", result)
```

---

## 五、Reflection（反思纠错闭环）

### 原理

在 Executor 之后加入 **Critic Agent**（裁判），对执行结果评分。若分数不达标，将批评意见反馈给 Executor 重试。

```
User Goal
    ↓
Executor Agent → Draft Output
    ↓
Critic Agent → {score, feedback}
    ↓ (score < threshold)
Executor Agent (修订) → Revised Output
    ↓ (score >= threshold 或达到最大轮次)
Final Output
```

**阿里视角**：这是"闭环能力"的核心体现。Critic 的 Prompt 设计质量决定了整个系统的上限。

```python
import json
from dataclasses import dataclass
from openai import OpenAI

client = OpenAI()


@dataclass
class CriticResult:
    score: float          # 0.0 ~ 1.0
    passed: bool
    feedback: str
    dimensions: dict[str, float]


def executor_agent(task: str, previous_feedback: str = "") -> str:
    """执行 Agent，可接收上轮的批评意见"""
    revision_hint = (
        f"\n\n上一轮的评审意见（请针对性改进）：\n{previous_feedback}"
        if previous_feedback else ""
    )
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "你是一个高质量的代码和方案生成专家。"},
            {"role": "user", "content": task + revision_hint},
        ],
        temperature=0.7,
    )
    return response.choices[0].message.content


def critic_agent(task: str, output: str) -> CriticResult:
    """
    Critic Agent：多维度评分
    使用 CoT + 结构化输出，避免"只说好/坏"的无效评审
    """
    critic_prompt = f"""
你是一个严格的技术评审专家。请按以下维度评审输出，先写推理过程，最后输出 JSON。

任务要求：{task}

待评审输出：
{output}

评审维度：
1. correctness（正确性）：逻辑是否正确，代码是否可运行 (0-1)
2. completeness（完整性）：是否覆盖了任务的所有要求 (0-1)
3. code_quality（代码质量）：可读性、注释、边界处理 (0-1)

请先写出你的推理过程（中文），然后输出以下格式的 JSON：
{{
  "dimensions": {{"correctness": 0.9, "completeness": 0.7, "code_quality": 0.8}},
  "overall_score": 0.8,
  "passed": true,
  "feedback": "具体的改进建议（如果 passed=true 可为空）"
}}

passed = true 的条件：overall_score >= 0.75
"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": critic_prompt}],
        temperature=0,
    )
    content = response.choices[0].message.content

    # 提取 JSON 部分（模型可能在前面输出了推理文字）
    json_start = content.rfind("{")
    json_end = content.rfind("}") + 1
    result_data = json.loads(content[json_start:json_end])

    return CriticResult(
        score=result_data["overall_score"],
        passed=result_data["passed"],
        feedback=result_data["feedback"],
        dimensions=result_data["dimensions"],
    )


def reflection_pipeline(task: str, max_rounds: int = 3) -> str:
    """
    完整的 Reflection 闭环
    max_rounds 防止无限循环（通常 2-3 轮就能收敛）
    """
    feedback = ""
    for round_num in range(1, max_rounds + 1):
        print(f"\n[Round {round_num}] Executor 生成输出...")
        output = executor_agent(task, previous_feedback=feedback)

        print(f"[Round {round_num}] Critic 评审中...")
        critic_result = critic_agent(task, output)

        print(
            f"  评分: {critic_result.score:.2f} | "
            f"通过: {critic_result.passed} | "
            f"维度: {critic_result.dimensions}"
        )

        if critic_result.passed:
            print(f"✓ 第 {round_num} 轮通过评审")
            return output

        feedback = critic_result.feedback
        print(f"  Feedback: {feedback[:150]}...")

    print(f"⚠ 达到最大轮次 {max_rounds}，返回最后一次输出")
    return output


# 测试
result = reflection_pipeline(
    task="用 Python 实现一个线程安全的 LRU Cache，要求：1. 支持 get/put 操作 "
         "2. 使用 OrderedDict 实现 3. 包含完整的类型注解和单元测试",
    max_rounds=3,
)
print("\n[最终输出]\n", result[:300])
```

---

## 六、Tree of Thought (ToT)（进阶补充）

### 原理

将推理过程建模为**树结构**，在每个节点生成多个候选思路，并用评估函数剪枝，选择最优路径。适合需要穷举探索的创意/规划问题。

**工程代价极高**：API 调用次数 = 节点数 × 候选数，通常只用于离线场景。

```
                Goal
               /  |  \
           思路A 思路B 思路C
           /\       |
         A1 A2    B1（剪枝）
         |
        A1.1（最优路径）
```

```python
from dataclasses import dataclass, field
from openai import OpenAI

client = OpenAI()


@dataclass
class ThoughtNode:
    thought: str
    score: float = 0.0
    children: list["ThoughtNode"] = field(default_factory=list)
    depth: int = 0


def generate_thoughts(problem: str, parent_thought: str, n: int = 3) -> list[str]:
    """在当前节点生成 n 个候选思路"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "user",
                "content": f"问题：{problem}\n\n当前思路：{parent_thought}\n\n"
                           f"请生成 {n} 个不同的下一步思路，每行一个：",
            }
        ],
        temperature=0.9,  # 高温度保证多样性
    )
    lines = response.choices[0].message.content.strip().split("\n")
    return [l.strip().lstrip("0123456789.-) ") for l in lines if l.strip()][:n]


def evaluate_thought(problem: str, thought: str) -> float:
    """评估单个思路的价值（0-1）"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "user",
                "content": f"问题：{problem}\n思路：{thought}\n\n"
                           f"评估这个思路解决问题的可能性（0-10的整数），只输出数字：",
            }
        ],
        temperature=0,
    )
    try:
        score = int(response.choices[0].message.content.strip()) / 10.0
    except ValueError:
        score = 0.5
    return score


def tree_of_thought(problem: str, breadth: int = 3, depth: int = 2) -> str:
    """
    简化版 ToT：BFS 展开 + 贪心选择
    breadth: 每层候选数  depth: 树的深度
    """
    # 初始化根节点
    root = ThoughtNode(thought="开始分析问题", score=1.0)
    current_level = [root]

    for d in range(depth):
        next_level = []
        for node in current_level:
            # 生成子思路
            child_thoughts = generate_thoughts(problem, node.thought, n=breadth)
            for ct in child_thoughts:
                score = evaluate_thought(problem, ct)
                child = ThoughtNode(thought=ct, score=score, depth=d + 1)
                node.children.append(child)
                next_level.append(child)
                print(f"  [Depth {d+1}] Score={score:.1f}: {ct[:60]}...")

        # 剪枝：只保留 top-breadth 的节点继续展开
        next_level.sort(key=lambda x: x.score, reverse=True)
        current_level = next_level[:breadth]

    # 返回最高分路径的最终思路
    best_node = max(current_level, key=lambda x: x.score)
    return best_node.thought


# ToT 适合复杂创意任务（如系统设计），实际生产中谨慎使用（成本高）
# result = tree_of_thought("设计一个高并发 AI 网关系统", breadth=3, depth=2)
```

---

## 七、工程速查表

| 方案 | 适用场景 | Token 消耗 | 延迟 | 容错性 | 工程复杂度 |
|------|---------|-----------|------|--------|-----------|
| Zero-shot | 简单指令、格式化提取 | ⭐ | 极低 | N/A | ⭐ |
| Few-shot | 格式固定的分类/提取 | ⭐⭐ | 低 | N/A | ⭐ |
| CoT | 数学推理、逻辑分析 | ⭐⭐⭐ | 中 | 中 | ⭐⭐ |
| Self-Consistency | 高精度离线推理 | ⭐⭐⭐⭐⭐ | 极高 | 高 | ⭐⭐ |
| ReAct | 简单工具调用、信息检索 | ⭐⭐⭐ | 中 | 低 | ⭐⭐⭐ |
| Plan-and-Execute | 复杂多步任务、可并行 | ⭐⭐⭐⭐ | 中-高 | 中 | ⭐⭐⭐⭐ |
| Reflection | 质量敏感任务（代码生成） | ⭐⭐⭐⭐ | 高 | 高 | ⭐⭐⭐⭐ |
| ToT | 创意探索、复杂规划（离线） | ⭐⭐⭐⭐⭐ | 极高 | 高 | ⭐⭐⭐⭐⭐ |

> **阿里面试核心考点**：ReAct 的上下文膨胀问题、Plan-and-Execute 的解耦优势、Reflection 闭环的工程实现。

---

*← [[agent-engineering-system]] | 下一篇 → [[agent-02-memory-context]]*
