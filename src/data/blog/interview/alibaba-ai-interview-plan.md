---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "阿里巴巴 AI 应用工程师（一面）冲刺计划"
postSlug: alibaba-ai-interview-plan
featured: false
draft: false
tags:
  - Alibaba
  - Interview
  - Plan
description: "Imported knowledge: 阿里巴巴 AI 应用工程师（一面）冲刺计划"
---


## 🎯 核心目标
在 5.25 中午前面向阿里 JD，完成“AI 应用工程师”专属维度的知识收敛。重点不再是单一项目的堆砌，而是**以《AI Agent 工程化系统手册》为主轴，将 CodeBuddy 等项目作为底层概念（Memory/Planning/Tool Use/Eval）的落地案例**。

---

## 📅 时间表与任务分配 (5.21 - 5.25 中午)

### Day 1: 5.21 (今日) - Agent 核心架构与方法论抽象
**JD 对齐**：职责 2 (架构设计)、职责 4 (核心能力实现)。

*   **1. 建立系统级 Agent 认知**：
    *   **复习资料**：精读 `01-Atomic-Knowledge/LLM-AI/agent-engineering-system.md`（第一部分：核心抽象架构）。
    *   **核心攻坚**：深刻理解 Agent 的四大支柱：Profile, Memory, Planning, Action。能白板画出 **ReAct**、**Plan-and-Execute** 和 **Reflection (反思纠错)** 的底层状态机流转。
*   **2. 将 CodeBuddy 转化为“落地案例” (Case Study)**：
    *   **复习资料**：`02-Projects/CodeBuddy-AI-Sidepane/project-defense.md`
    *   **策略转变**：弱化“这是一个侧边栏插件”，强化“这是一个包含**路由 (Routing)**、**状态机记忆 (Memory)** 和 **沙盒工具调用 (Tool Use)** 的 AI 智能体应用”。
    *   **实战防守**：用 CodeBuddy 的经验回答“你是如何处理 LLM 输出 JSON 格式损坏的？”（Schema 校验与重试兜底）。
*   **3. 环境与知识构建 (Context Engineering)**：
    *   **MCP 协议深度理解**：阿里 JD 明确要求的加分项。掌握 MCP 如何解耦 LLM 与本地环境。
    *   **RAG 进阶**：从单纯的 Vector 检索，升级到对 Hybrid Search、Rerank 和 Chunking 策略（如 AST Chunking）的工程化理解。

### Day 2: 5.22 - 评测、AI Infra 与高并发系统设计
**JD 对齐**：职责 5 (系统迭代/自动化评测)、职责 6 (性能优化)。

*   **1. 自动化评测 (Eval) - 从单点验证到规模化**：
    *   **复习资料**：`01-Atomic-Knowledge/LLM-AI/agent-engineering-system.md`（第二部分）。
    *   **重点攻克**：如何利用 LLM-as-a-Judge 构建黄金数据集？如何量化评估 RAG 的召回率与准确率（如 RAGAS 指标）？
*   **2. 高并发与性能优化 (AI Infra)**：
    *   **阿里 System Design 必考**：“如果流量翻 100 倍，你的 Agent 系统怎么撑住？”
    *   **核心考点**：流式输出 (SSE) 优化、分布式 KV Cache (PagedAttention) 管理、长任务的异步化处理与状态持久化（类似 LangGraph）。
*   **3. 阿里 2026 前沿模型适配**：
    *   了解 Qwen 2.5/3 系列模型的特性。在工程中，如何针对特定模型进行 Prompt 调优和规避其短板？

### Day 3: 5.23 - 算法手撕与全栈基石
**JD 对齐**：专业能力 (扎实的代码能力)。

*   **复习资料**：
    *   `01-Atomic-Knowledge/Algorithms/fullstack-interview-handwrite.md`
    *   `01-Atomic-Knowledge/Architecture/network-protocols.md`
*   **重点攻克**：
    1.  **AI 业务场景手撕题**：给定一段 Agent 思考日志（Markdown/JSON 混合），手写逻辑提取 `Action` 和 `Action Input`。
    2.  **经典手撕题**：LRU 缓存实现（Agent 记忆管理核心）、最小覆盖子串（LeetCode 76）、编辑距离（文本相似度）。
    3.  **网络与 OS 底层**：TCP/UDP、IO 多路复用、死锁原理。这是阿里考察“工程兜底能力”的必问项。

### Day 4: 5.24 - 生产级 Prompt 与极客软素质准备
**JD 对齐**：能力特质 (AI 编程工具重度玩家)。

*   **1. 生产级 Prompt 工程**：
    *   **重点攻克**：防御 Prompt 注入；通过 Few-shot 和 Schema 强约束保障结构化输出的 100% 成功率。
*   **2. AI 生产力方法论**：
    *   **实战防守**：总结你在使用 Cursor、Claude Code 等工具时的套路。你是如何通过 Agentic Workflow 让他们写出生产级代码的？（体现 10x 程序员特质）。
*   **3. 自我驱动故事**：
    *   准备一个符合 STAR 原则的案例，证明你“能快速啃透最新论文并转化为工程代码”或“拒绝纸上谈兵”。

### Day 5: 5.25 (上午) - 冲刺模考与心态调整
*   **早上 9:00 - 11:00**：对着大纲进行快速“车轮战”自问自答，特别是“Agent 四大支柱”和“高并发 System Design”。
*   **11:00 之后**：平复心态，检查网络与设备，迎接面试。

---

## 🚀 执行与验证方式 (Verification)
1.  **每日 Check**：每天晚上睡前，尝试在白板上画出当日复习模块的架构图（如 ReAct 流程图、RAG 检索重排架构图）。
2.  **文档完善**：计划已与 `agent-engineering-system.md` 深度绑定，请优先消化该手册。
