---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "Agent Tool Use & Action 工具编排"
postSlug: agent-03-tool-use-action
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: Agent Tool Use & Action 工具编排"
---

---
tags: [LLM, Agent, ToolUse, FunctionCalling, MCP]
created: 2026-05-22
---

# Agent Tool Use & Action 工具编排

> Agent 通过工具连接现实世界。核心工程问题：**如何提高参数填充正确率、处理工具失败、控制并发副作用？**

---

## 一、Function Calling 工作原理

### 流程

```
User Message
    ↓
LLM → tool_calls: [{name, arguments}]   ← 模型决定调用哪些工具
    ↓
执行工具（本地/远程）
    ↓
Tool Result → 重新送入 LLM
    ↓
LLM → 最终自然语言回答
```

### 提高参数填充正确率的技巧

1. **Schema 描述要精确**：`description` 字段比 `name` 更重要，模型靠它理解工具用途
2. **在 description 中给示例**：`"example: 'Shanghai' not '上海'"`
3. **用 enum 约束选项**：能用 enum 就不用 string
4. **Few-shot tool examples**：在 system prompt 中演示工具调用范例

---

## 二、完整 Tool Call Loop 实现

```python
import json
import asyncio
from openai import OpenAI, AsyncOpenAI

client = OpenAI()
async_client = AsyncOpenAI()

# ── 工具定义（Schema） ─────────────────────────────────────
TOOLS_SCHEMA = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "搜索互联网获取最新信息。适合查询时事、最新文档、不确定的事实。",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词，使用英文效果更好。example: 'vLLM PagedAttention'"
                    },
                    "num_results": {
                        "type": "integer",
                        "description": "返回结果数量",
                        "enum": [3, 5, 10],
                        "default": 5,
                    }
                },
                "required": ["query"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "查询指定城市的实时天气。city 必须是英文城市名。",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市英文名。example: 'Beijing' not '北京'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "温度单位",
                    }
                },
                "required": ["city"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "执行数学计算。适合精确的算术运算，避免 LLM 计算错误。",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "合法的 Python 数学表达式。example: '(100 * 2000 * 3600 * 8 * 30) / 1e6 * 2.5'"
                    }
                },
                "required": ["expression"],
            },
        },
    },
]


# ── 工具实现（mock） ──────────────────────────────────────
def search_web(query: str, num_results: int = 5) -> str:
    """模拟搜索工具"""
    return f"搜索'{query}'的结果（共{num_results}条）：[1] vLLM 官方文档... [2] PagedAttention 论文..."


def get_weather(city: str, unit: str = "celsius") -> str:
    """模拟天气工具"""
    temp = {"Beijing": 28, "Shanghai": 32, "Hangzhou": 30}.get(city, 25)
    unit_symbol = "°C" if unit == "celsius" else "°F"
    return f"{city}: {temp}{unit_symbol}，晴，湿度 60%"


def calculator(expression: str) -> str:
    """安全的计算器（只允许数学运算）"""
    import ast as ast_module
    try:
        # 使用 AST 解析，避免 eval 注入风险
        tree = ast_module.parse(expression, mode="eval")
        # 白名单：只允许数字、运算符、内置数学函数
        allowed_nodes = {
            ast_module.Expression, ast_module.BinOp, ast_module.UnaryOp,
            ast_module.Num, ast_module.Constant, ast_module.Add, ast_module.Sub,
            ast_module.Mult, ast_module.Div, ast_module.Pow, ast_module.Mod,
            ast_module.USub, ast_module.UAdd,
        }
        for node in ast_module.walk(tree):
            if type(node) not in allowed_nodes:
                return f"不允许的表达式类型: {type(node).__name__}"
        result = eval(compile(tree, "<string>", "eval"))
        return f"计算结果: {result}"
    except Exception as e:
        return f"计算失败: {e}"


TOOL_REGISTRY = {
    "search_web": search_web,
    "get_weather": get_weather,
    "calculator": calculator,
}


def run_tool(tool_name: str, arguments: dict) -> str:
    """统一工具执行入口，带异常捕获"""
    if tool_name not in TOOL_REGISTRY:
        return f"未知工具: {tool_name}"
    try:
        return TOOL_REGISTRY[tool_name](**arguments)
    except Exception as e:
        return f"工具执行失败: {e}"


def tool_agent(user_query: str, max_iterations: int = 5) -> str:
    """
    完整的工具调用 Agent 循环
    支持 parallel tool calls（OpenAI 会在一次响应中返回多个 tool_calls）
    """
    messages = [
        {"role": "system", "content": "你是一个助手，可以使用工具回答问题。优先用工具获取准确信息。"},
        {"role": "user", "content": user_query},
    ]

    for i in range(max_iterations):
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=TOOLS_SCHEMA,
            tool_choice="auto",
            parallel_tool_calls=True,  # 允许一次调用多个工具
        )
        msg = response.choices[0].message
        messages.append(msg)

        # 无工具调用 → 最终回答
        if not msg.tool_calls:
            return msg.content

        # 处理所有工具调用（可能是并行的）
        for tool_call in msg.tool_calls:
            name = tool_call.function.name
            args = json.loads(tool_call.function.arguments)
            print(f"  [Tool] {name}({args})")
            result = run_tool(name, args)

            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result,
            })

    return "已达最大迭代次数"


# 测试
answer = tool_agent("北京现在天气怎么样？顺便帮我算一下 100 * 2000 * 24 * 30 / 1e6 * 2.5 是多少？")
print(answer)
```

---

## 三、Pydantic 校验 + Auto-Correction 闭环

⭐ **阿里面试高频考点**：工程上如何保证模型输出的结构化数据可用？

```python
import json
from pydantic import BaseModel, Field, ValidationError
from openai import OpenAI

client = OpenAI()


class CodeReviewResult(BaseModel):
    """代码审查结果的结构化模型"""
    severity: str = Field(..., pattern="^(critical|major|minor|info)$")
    issues: list[str] = Field(..., min_length=0)
    score: float = Field(..., ge=0.0, le=10.0)
    refactored_code: str | None = None
    summary: str


def extract_with_autocorrect(
    raw_text: str,
    schema_class: type[BaseModel],
    max_retries: int = 3,
) -> BaseModel:
    """
    Pydantic 校验 + Auto-Correction 闭环
    
    失败时将错误信息回传给 LLM 自动修复，最多重试 max_retries 次
    这是生产级 Agent 必备的容错机制
    """
    messages = [
        {
            "role": "system",
            "content": f"你需要将输出格式化为以下 JSON Schema：\n{schema_class.model_json_schema()}",
        },
        {"role": "user", "content": raw_text},
    ]

    for attempt in range(max_retries):
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            temperature=0,
            response_format={"type": "json_object"},
        )
        output = response.choices[0].message.content

        try:
            data = json.loads(output)
            return schema_class(**data)
        except (json.JSONDecodeError, ValidationError) as e:
            error_msg = str(e)
            print(f"[Attempt {attempt + 1}] 校验失败: {error_msg[:100]}")

            if attempt < max_retries - 1:
                # 关键：将错误信息反馈给 LLM，让其自行修复
                messages.append({"role": "assistant", "content": output})
                messages.append({
                    "role": "user",
                    "content": (
                        f"输出校验失败，错误信息：{error_msg}\n\n"
                        "请修正 JSON 格式，严格符合 Schema 要求后重新输出："
                    ),
                })

    raise ValueError(f"经过 {max_retries} 次尝试仍无法生成合法的结构化输出")


# 测试
result = extract_with_autocorrect(
    raw_text="帮我审查这段 Python 代码：def foo(x): return x*x\n评分 8.5，有个小问题：没有类型注解",
    schema_class=CodeReviewResult,
)
print(result.model_dump_json(indent=2))
```

---

## 四、异步并行 Tool Calling

当多个工具调用之间没有依赖关系时，应当**并发执行**，而非顺序等待。

```python
import asyncio
import json
from openai import AsyncOpenAI

async_client = AsyncOpenAI()


async def run_tool_async(tool_name: str, arguments: dict) -> tuple[str, str]:
    """异步执行工具，返回 (tool_call_id, result)"""
    # 模拟异步 IO（如数据库查询、HTTP 请求）
    await asyncio.sleep(0.1)
    result = run_tool(tool_name, arguments)
    return result


async def parallel_tool_agent(user_query: str) -> str:
    """
    并行工具调用 Agent
    当 LLM 返回多个 tool_calls 时，用 asyncio.gather 并发执行
    相比顺序执行，延迟 = max(单个工具延迟) 而非 sum(所有工具延迟)
    """
    messages = [
        {"role": "system", "content": "你是一个助手，尽量同时调用多个工具提高效率。"},
        {"role": "user", "content": user_query},
    ]

    for _ in range(5):
        response = await async_client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=TOOLS_SCHEMA,
            tool_choice="auto",
            parallel_tool_calls=True,
        )
        msg = response.choices[0].message
        messages.append(msg)

        if not msg.tool_calls:
            return msg.content

        # 并发执行所有工具调用
        tasks = []
        for tc in msg.tool_calls:
            args = json.loads(tc.function.arguments)
            tasks.append(run_tool_async(tc.function.name, args))

        results = await asyncio.gather(*tasks, return_exceptions=True)

        for tc, result in zip(msg.tool_calls, results):
            if isinstance(result, Exception):
                result = f"工具执行异常: {result}"
            messages.append({
                "role": "tool",
                "tool_call_id": tc.id,
                "content": str(result),
            })

    return "已达最大迭代次数"


# asyncio.run(parallel_tool_agent("查询北京天气，同时计算 1000 * 2.5"))
```

---

## 五、规则引擎降级（Fallback）

⭐ **阿里视角**：生产系统必须有"保底"机制，LLM 不稳定时不能直接报错给用户。

```python
from enum import Enum
from dataclasses import dataclass, field


class AgentState(Enum):
    LLM_NORMAL = "llm_normal"
    LLM_DEGRADED = "llm_degraded"   # LLM 连续失败，降级到规则引擎
    CIRCUIT_OPEN = "circuit_open"    # 熔断，暂停 LLM 调用


@dataclass
class CircuitBreaker:
    """
    熔断器：追踪 LLM 工具调用失败次数
    连续失败 N 次 → 触发降级
    """
    failure_threshold: int = 3
    recovery_timeout: float = 60.0  # 秒，熔断后多久尝试恢复
    _failure_count: int = field(default=0, init=False)
    _last_failure_time: float = field(default=0.0, init=False)
    _state: AgentState = field(default=AgentState.LLM_NORMAL, init=False)

    def record_failure(self):
        import time
        self._failure_count += 1
        self._last_failure_time = time.time()
        if self._failure_count >= self.failure_threshold:
            self._state = AgentState.LLM_DEGRADED
            print(f"⚠ 熔断器触发：LLM 连续失败 {self._failure_count} 次，降级到规则引擎")

    def record_success(self):
        self._failure_count = 0
        self._state = AgentState.LLM_NORMAL

    def should_use_llm(self) -> bool:
        import time
        if self._state == AgentState.LLM_NORMAL:
            return True
        # 超过恢复时间，尝试半开放
        if time.time() - self._last_failure_time > self.recovery_timeout:
            print("熔断器半开放，尝试 LLM...")
            return True
        return False


# 规则引擎（确定性兜底）
RULE_ENGINE_RESPONSES = {
    "天气": "抱歉，天气查询服务暂时不可用，请稍后重试或访问天气网站。",
    "计算": "抱歉，计算服务暂时不可用，请使用计算器工具。",
    "搜索": "抱歉，搜索服务暂时不可用，请直接访问搜索引擎。",
}


def rule_based_response(user_query: str) -> str:
    """规则引擎：基于关键词匹配，给出保底回复"""
    for keyword, response in RULE_ENGINE_RESPONSES.items():
        if keyword in user_query:
            return response
    return "抱歉，系统暂时遇到问题，请稍后重试。"


circuit_breaker = CircuitBreaker(failure_threshold=3)


def resilient_tool_agent(user_query: str) -> str:
    """带熔断降级的 Agent"""
    if not circuit_breaker.should_use_llm():
        print("[Fallback] 规则引擎处理中...")
        return rule_based_response(user_query)

    try:
        result = tool_agent(user_query)
        circuit_breaker.record_success()
        return result
    except Exception as e:
        print(f"[Error] LLM 调用失败: {e}")
        circuit_breaker.record_failure()
        return rule_based_response(user_query)
```

---

## 六、MCP (Model Context Protocol) 概念

MCP 是 Anthropic 提出的标准化协议，解决"如何让 LLM 统一访问异构数据源"的问题。

```
┌─────────────────────────────────────┐
│           MCP Host (Claude)         │
│  ┌─────────────────────────────┐   │
│  │        MCP Client           │   │
│  └──────┬──────────────────────┘   │
└─────────┼───────────────────────────┘
          │ MCP Protocol (JSON-RPC)
    ┌─────┼──────┐
    ↓     ↓      ↓
  DB    File   HTTP API
 Server Server  Server
```

**核心概念**：
- **Resources**：LLM 可读取的数据（文件、数据库记录）
- **Tools**：LLM 可调用的函数
- **Prompts**：可复用的 Prompt 模板

```python
# MCP Server 的简化实现思路（伪代码，展示协议结构）
from dataclasses import dataclass
from typing import Any


@dataclass
class MCPTool:
    name: str
    description: str
    input_schema: dict
    handler: callable


class SimpleMCPServer:
    """
    简化版 MCP Server 实现思路
    真实场景使用 mcp Python SDK：pip install mcp
    """
    def __init__(self, name: str):
        self.name = name
        self._tools: dict[str, MCPTool] = {}

    def register_tool(self, tool: MCPTool):
        self._tools[tool.name] = tool

    def list_tools(self) -> list[dict]:
        """列出所有可用工具（供 LLM 选择）"""
        return [
            {
                "name": t.name,
                "description": t.description,
                "inputSchema": t.input_schema,
            }
            for t in self._tools.values()
        ]

    def call_tool(self, tool_name: str, arguments: dict) -> Any:
        """执行工具调用"""
        if tool_name not in self._tools:
            raise ValueError(f"Tool not found: {tool_name}")
        return self._tools[tool_name].handler(**arguments)


# 注册工具示例
server = SimpleMCPServer("code-assistant-server")
server.register_tool(MCPTool(
    name="read_file",
    description="读取代码文件内容",
    input_schema={
        "type": "object",
        "properties": {"path": {"type": "string"}},
        "required": ["path"],
    },
    handler=lambda path: open(path).read(),
))
```

---

## 七、Tool Use 工程陷阱清单

| 陷阱 | 现象 | 解决方案 |
|------|------|---------|
| **Schema 描述不清** | 模型选错工具或参数填错 | 精化 description，加 example |
| **并发副作用** | 重复写入数据库、重复发邮件 | 工具设计为幂等；写操作加分布式锁 |
| **无限循环** | Agent 反复调用同一工具 | 设置 `max_iterations`；检测重复调用 |
| **工具超时阻塞** | HTTP 工具无超时导致卡死 | 所有 IO 设置 timeout；async + asyncio.wait_for |
| **参数注入** | 用户输入包含特殊字符破坏 SQL/命令 | 使用参数化查询；白名单校验工具参数 |
| **Token 暴增** | 工具返回超大内容（如整个文件） | 工具层截断输出；只返回摘要 + 分页 |
| **缺少降级** | LLM 挂掉时直接 500 | 熔断器 + 规则引擎兜底 |

---

*← [[agent-02-memory-context]] | 下一篇 → [[agent-04-eval-system]]*
