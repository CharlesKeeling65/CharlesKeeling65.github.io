---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "AI Agent 工程化系统手册 (2026 版)"
postSlug: agent-engineering-system
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: AI Agent 工程化系统手册 (2026 版)"
---

---
tags: [LLM, Agent, Index, 阿里面试]
created: 2026-05-22
---

# AI Agent 工程化系统手册 (2026 版)

> **面试核心定位**：面向**阿里 AI 应用工程师（工程落地）**的实战知识体系。  
> 所有理论必须能映射到工程实现和高并发业务场景。

---

## 📚 知识体系导航

本手册已按主题拆分为专项深度笔记，每篇包含**原理拓展 + Python 示例代码**：

| 序号 | 主题 | 核心内容 | 阿里面试相关度 |
|------|------|---------|--------------|
| 01 | [[agent-01-planning-reasoning]] | CoT / ReAct / Plan-and-Execute / Reflection / ToT | ⭐⭐⭐⭐⭐ |
| 02 | [[agent-02-memory-context]] | Sliding Window / Summary / RAG / Hybrid Search / Rerank | ⭐⭐⭐⭐⭐ |
| 03 | [[agent-03-tool-use-action]] | Function Calling / Pydantic校验 / 并发工具 / MCP / 熔断降级 | ⭐⭐⭐⭐⭐ |
| 04 | [[agent-04-eval-system]] | LLM-as-Judge / RAGAS / Eval Runner / CI/CD 门禁 | ⭐⭐⭐⭐⭐ |
| 05 | [[agent-05-infra-performance]] | KV Cache / PagedAttention / Continuous Batching / SSE 流式 | ⭐⭐⭐⭐⭐ |
| 06 | [[agent-06-observability]] | Tracing / Token成本 / 归因分析 / 告警 | ⭐⭐⭐⭐ |

---

## 🗺️ 核心架构速览

```
┌─────────────────────────────────────────────────────────┐
│                    Agent 系统架构                         │
│                                                          │
│  User Input                                              │
│      ↓                                                   │
│  ┌─────────────────────────────────────────┐            │
│  │  Planning Layer（规划层）                │            │
│  │  ReAct / Plan-and-Execute / Reflection   │            │
│  └─────────────────────────────────────────┘            │
│      ↓                    ↓                              │
│  ┌──────────┐      ┌──────────────┐                     │
│  │ Memory   │      │ Tool / Action │                     │
│  │ RAG/滑窗 │      │ Function Call │                     │
│  └──────────┘      └──────────────┘                     │
│      ↓                    ↓                              │
│  ┌─────────────────────────────────────────┐            │
│  │  Eval 体系（评测 + CI/CD 质量门禁）       │            │
│  └─────────────────────────────────────────┘            │
│  ┌─────────────────────────────────────────┐            │
│  │  Observability（Trace + Metrics + Cost） │            │
│  └─────────────────────────────────────────┘            │
│  ┌─────────────────────────────────────────┐            │
│  │  Infra（vLLM + Continuous Batching）     │            │
│  └─────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────┘
```

---

## ⚡ 高频面试问题速查

### Planning 相关
- **Q: ReAct 有什么缺点？** → [[agent-01-planning-reasoning#三、ReAct]]  
  上下文膨胀（每步携带完整历史）+ 容错率低（一步错步步错）
- **Q: 如何解决 Agent 死循环？** → 设置 `max_steps`；Reflection 闭环自我纠错

### Memory / RAG 相关
- **Q: 如何提高 RAG 检索准确率？** → [[agent-02-memory-context#二、长期记忆]]  
  混合检索（BM25 + 向量） + Cross-Encoder Reranker
- **Q: Lost in the Middle 是什么？** → 长上下文中间部分被模型忽略；减少 Top-K 或重排文档位置

### Tool Use 相关
- **Q: 如何提高 Function Calling 的参数填充正确率？** → [[agent-03-tool-use-action]]  
  精化 Schema description + 加 example + Pydantic 校验 + Auto-Correction 重试

### Infra 相关
- **Q: vLLM 的 PagedAttention 原理？** → [[agent-05-infra-performance#二、KV Cache 与 PagedAttention]]  
  借鉴操作系统分页机制，动态分配 KV Cache，消除显存碎片，Batch Size 提升 2-4x
- **Q: 什么是 Continuous Batching？** → 请求完成后立刻插入新请求，GPU 始终满载

### Eval 相关
- **Q: 什么是 Faithfulness？怎么计算？** → [[agent-04-eval-system#四、RAGAS 指标实现]]  
  将答案拆解为 Claims，逐一验证是否有 Context 支撑，支撑比例即 Faithfulness

---

## 🎯 面试防守策略：如何用 CodeBuddy 回答概念问题

**策略：将具体项目作为抽象概念的落地证明。**

- **当问"意图识别"时**：  
  "在 CodeBuddy 中，通过前置路由层将'解释代码'和'重构代码'分流到不同小 Agent，提高并发能力和响应速度。"

- **当问"Agent 幻觉"时**：  
  "引入 Reflection 闭环：模型生成修改计划后，校验卡点对比 AST，错误信息反馈给模型纠正。"

- **当问"长上下文优化"时**：  
  "没有把整个文件树喂给模型，而是只注入相关函数签名（摘要），按需工具读取完整内容。"

- **当问"系统如何保证稳定性"时**：  
  "三层保障：Pydantic 强校验 + Auto-Correction 重试 + 规则引擎兜底；同时有熔断器监控失败率。"

---

*同目录：[[prompt-engineering]]*
