---
author: Charles Keeling
pubDatetime: 2026-06-01T08:00:00Z
title: 华泰 Fintech 星战营 B站视频详细筛选表
slug: huatai-fintech-bilibili-video-screening
featured: false
draft: false
tags:
  - bilibili
  - fintech
  - interview
  - 金融AI
  - 大模型
description: B站视频详细筛选表，华泰 Fintech 星战营面试冲刺。
---

# B站视频详细筛选表

> 数据来源：2026-06-01 使用内部浏览器打开 B站搜索页逐项检索。时长、播放量、UP 主和日期来自搜索页可见卡片；少数旧候选来自上一轮搜索计划。B站搜索页会混入标题党和弱相关内容，本表已按 7 天面试冲刺目标做二次筛选。

## 总体观看策略

优先级定义：

- **P0 必看**：当天主线，直接支撑面试表达。
- **P1 选看**：用于补案例、补金融场景、补追问。
- **P2 查阅**：长课或噪音较多，只按章节查，不完整观看。

推荐总顺序：

```text
Day1 自我叙事
→ Day2 Transformer + 训练范式 + LoRA
→ Day3 RAG + Agent + RAG面试追问
→ Day4 因子/风控/知识图谱/NLP
→ Day5 AI研报助手/智能投顾产品设计
→ Day6 SHAP/联邦学习/合规/Flink架构
→ Day7 模拟面试题
```

## Day 1：自我定位与行业认知

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [面试自我介绍：面试官让你自我介绍时，到底想听到什么？](https://www.bilibili.com/video/BV1jy4y1j7N6/) | 鹅姐职场 / 2021-07-18 | 09:52 | 讲自我介绍的底层逻辑、常见雷区和案例结构。 | 直接转成 1/2/3 分钟个人叙事模板。 |
| 2 | P1 | [一面试就挂？把这段自我介绍背下来！](https://www.bilibili.com/video/BV1Yx4y1p7fd/) | 小王养成日记 / 2024-05-27 | 03:22 | 快速给出普通求职自我介绍话术。 | 只借结构，不要照背。 |
| 3 | P1 | [手把手教你读研报！我的研报阅读经验分享](https://www.bilibili.com/video/BV1TP4y1H7vw/) | 嗨皮土豆888 / 2021-12-16 | 51:28 | 研报选择、研报阅读、宏观/行业/公司研报理解。 | 为“AI研报助手”补真实用户任务。 |
| 4 | P1 | [AI已从银行的“增效工具”变成“核心生产力”](https://www.bilibili.com/video/BV1gb6cB9EqC/) | 大王AI应用报告 / 01-29 | 20:55 | 银行业 AI 应用从效率工具到业务生产力的变化。 | 用来回答“金融行业为什么需要 AI”。 |
| 5 | P2 | [7大AI智能投研平台剖析对比](https://www.bilibili.com/video/BV1Xw1HB8EED/) | 资管业务与科技 / 2025-11-05 | 01:28 | 快速展示智能投研平台类别。 | 只作为产品灵感入口。 |

Day 1 产出：一张个人叙事卡，必须包含“AI 技术背景、金融场景理解、为什么华泰 AI”三段。

## Day 2：Transformer 与大模型训练

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [【闪客】一小时从函数到 Transformer](https://www.bilibili.com/video/BV1NCgVzoEG9/) | 飞天闪客 / 2025-07-21 | 01:09:17 | 从函数、表示、注意力逐步讲到 Transformer。 | 作为主线，建立“为什么 Transformer 适合大模型”的直觉。 |
| 2 | P0 | [【学大模型必看】AI大模型是怎么炼成的？](https://www.bilibili.com/video/BV1HhoTBeERS/) | 卢菁博士_北大AI博士后 / 04-24 | 46:23 | 预训练、SFT、RLHF、量化、蒸馏、Transformer、Token、Prompt、LoRA。 | 看完后整理“训练三阶段 + 金融微调理由”。 |
| 3 | P0 | [20分钟带你快速弄懂SFT、RLHF、DPO](https://www.bilibili.com/video/BV1HYBWBaEE3/) | AI敲代码的阿Q / 2025-12-22 | 19:23 | 讲 SFT、RLHF、DPO 的定义和适用边界。 | 面试要能比较 RLHF 与 DPO。 |
| 4 | P0 | [LoRA是什么？5分钟讲清楚LoRA的工作原理](https://www.bilibili.com/video/BV17i421X7q7/) | 是花子呀_ / 2024-05-18 | 05:40 | 用短视频解释 LoRA 的低秩适配思想。 | 最适合转化成口述答案。 |
| 5 | P1 | [什么是LoRA 大模型微调是怎么回事](https://www.bilibili.com/video/BV1PvwYzxE9D/) | 隔壁的程序员老王 / 03-19 | 13:35 | 解释大模型微调和 LoRA 的关系。 | 作为 LoRA 补充。 |
| 6 | P1 | [10分钟，让你彻底理解Transformer](https://www.bilibili.com/video/BV1rY81zpEMa/) | 交叉科学 / 2025-07-31 | 11:35 | 快速复盘 Transformer 机制。 | Day 2 晚上复盘用。 |
| 7 | P2 | [【2026/Minimind】一集通关SFT、LoRA、PPO、DPO、GRPO](https://www.bilibili.com/video/BV1XWfUBTEHe/) | 木乔_Mokio / 02-25 | 01:33:30 | 代码和训练流程实操。 | 面试冲刺不用完整看，只查概念章节。 |

Day 2 产出：Transformer + 训练范式知识卡。口述重点是 Q/K/V、并行训练、预训练、SFT、RLHF/DPO、LoRA。

## Day 3：RAG / Agent / 评估

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [十分钟了解RAG基本原理](https://www.bilibili.com/video/BV1C69iBgEMk/) | 林爻linyao / 05-04 | 10:17 | RAG 基本流程和概念。 | 主线视频，直接画流程图。 |
| 2 | P0 | [RAG是什么？和大模型有什么关系？](https://www.bilibili.com/video/BV1AKRLY4ECg/) | 小生凡一 / 2025-03-11 | 04:48 | 解释 RAG、向量数据库和大模型的关系。 | 用来补“为什么不是只靠模型记忆”。 |
| 3 | P0 | [别再死记 AI 名词了：用一条线讲清 Prompt、RAG、MCP、Agent](https://www.bilibili.com/video/BV1WTRzBWEsz/) | 飞行认知 / 05-05 | 27:15 | 串联 Prompt、RAG、MCP、Agent 的关系。 | 用来建立系统视角，适合面试说“技术选型”。 |
| 4 | P1 | [【Agent入门】工具调用实现原理](https://www.bilibili.com/video/BV1ymrCBeESA/) | 花里胡哨的汤无际 / 01-16 | 39:38 | 讲 Agent 工具调用机制。 | 看核心工具调用部分即可。 |
| 5 | P1 | [我用7层架构重写RAG后准确率从40%飙到92%](https://www.bilibili.com/video/BV1NwV86hEYB/) | 费曼学AI / 05-30 | 14:30 | 讲 RAG 准确率、召回、架构优化。 | 作为“如何降低幻觉/提升召回”的案例。 |
| 6 | P1 | [【沉浸式大模型面试】RAG项目拷问](https://www.bilibili.com/video/BV1sV2PYCEo3/) | 丁师兄大模型 / 2024-10-09 | 04:19 | RAG 面试追问：流程、优化、评估、幻觉。 | Day 3 或 Day 7 都要看。 |
| 7 | P2 | [黑马程序员大模型RAG与Agent智能体项目实战教程](https://www.bilibili.com/video/BV1yjz5BLEoY/) | 黑马程序员 / 01-21 | 15h+ | RAG + Agent + LangChain 项目课。 | 超长资料库，只查章节。 |

Day 3 产出：RAG 工作流程图。口述重点是“金融知识更新频繁，所以 RAG 是必选项；微调解决风格和行为，RAG 解决实时知识”。

## Day 4：金融 AI 算法

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [《因子投资-方法与实践》第1章 因子投资基础](https://www.bilibili.com/video/BV1VxZcBqEsj/) | 量衍金工 / 04-26 | 13:32 | 因子投资基础概念。 | 先看，建立“因子是什么”。 |
| 2 | P0 | [Fama French三因子策略深度解析](https://www.bilibili.com/video/BV1Py4y1j7x5/) | Devils-Advocate / 2021-07-23 | 16:24 | 解释经典三因子模型。 | 用来回答“传统因子 vs AI 因子”。 |
| 3 | P1 | [量化多因子模型建模全流程](https://www.bilibili.com/video/BV1oB4y167ur/) | 科大金工 / 2022-08-12 | 15:26 | 多因子建模流程。 | 看流程，不陷入公式细节。 |
| 4 | P1 | [Python因子回测：一个因子的诞生](https://www.bilibili.com/video/BV1uj421S7Rf/) | 大导演哈罗德 / 2024-02-27 | 13:29 | 从因子想法到回测。 | 用来补“AI 在量化不是预测涨跌”。 |
| 5 | P0 | [B站最系统的金融风控课程，信用评分卡模型、互联网金融风控模型](https://www.bilibili.com/video/BV1RL8kzTErw/) | python风控模型 / 2025-07-31 | 05:24:08 | 信用评分卡、互联网金融风控、企业风险建模。 | P2 长课，但风控流程章节必查。 |
| 6 | P0 | [模型评估：Accuracy / Precision / Recall / F1](https://www.bilibili.com/video/BV1vt4y117Zz/) | 小萌Annie / 2020-05-19 | 06:03 | 机器学习评估指标。 | 用来解释风控模型不能只看准确率。 |
| 7 | P0 | [知识图谱在金融风控场景的应用](https://www.bilibili.com/video/BV1Zm4y1w7xF/) | NebulaGraph / 2022-10-26 | 42:50 | 金融风控中的知识图谱行业案例。 | 行业案例价值高，建议看。 |
| 8 | P1 | [邦盛科技：关联图谱在金融反欺诈领域的应用与实践](https://www.bilibili.com/video/BV1FY4y147Pw/) | NebulaGraph / 2022-06-06 | 01:07:55 | 关联图谱和反欺诈实践。 | 时间够再看，重点听团伙欺诈识别。 |
| 9 | P1 | [基于大模型做信息抽取方法介绍](https://www.bilibili.com/video/BV1rJ4m1s7Nm/) | AI大实话 / 2024-02-21 | 26:45 | 大模型信息抽取方法。 | 用来支撑财报/研报结构化抽取。 |
| 10 | P1 | [信息抽取统一框架 UIE](https://www.bilibili.com/video/BV1LW4y1U7ch/) | breezedeus / 2022-07-02 | 31:41 | 通用信息抽取框架。 | 选看，补 NLP/NER/关系抽取背景。 |

Day 4 产出：金融 AI 算法速查卡。核心观点是“量化找统计规律，风控重评估与可解释，反欺诈适合图谱/关系网络”。

## Day 5：AI 产品设计

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [7大AI智能投研平台剖析对比](https://www.bilibili.com/video/BV1Xw1HB8EED/) | 资管业务与科技 / 2025-11-05 | 01:28 | 快速扫智能投研平台形态。 | 作为“AI研报助手”竞品灵感。 |
| 2 | P0 | [多Agent + 爬虫：AI自动化研报/新闻摘要生成平台](https://www.bilibili.com/video/BV1ucVn6gEhk/) | 程序员花菜 / 10小时前 | 11:16 | 多 Agent、爬虫、研报/新闻摘要平台。 | 适合转化为“AI研报助手”的功能流程。 |
| 3 | P0 | [【AI金融】智能投顾平台](https://www.bilibili.com/video/BV1MJFbeXEuD/) | AICDA / 2025-01-29 | 05:12 | 智能投顾平台展示。 | 用于产品设计中的用户场景和合规边界。 |
| 4 | P1 | [产品经理如何用AI赋能从需求分析到产品设计](https://www.bilibili.com/video/BV1bU6nBrE3d/) | AI产品经理养成记 / 02-04 | 08:15 | AI 辅助需求分析和产品设计流程。 | 用来组织 PRD/功能文档表达。 |
| 5 | P1 | [我是怎么拿到7家产品经理offer的？](https://www.bilibili.com/video/BV1Gy4y1r7jf/) | 斯前想后来 / 2020-10-18 | 10:17 | 产品经理面试表达和追问。 | Day 5 写完文档后看，练表达。 |
| 6 | P2 | [AI大模型解决方案专家：从0到1拆解大模型落地方法论](https://www.bilibili.com/video/BV1HErwBVEep/) | 产品先锋 / 01-09 | 07:18:16 | 大模型落地方法论、方案、售前/产品视角。 | 长课，只看“场景拆解/落地方法论”章节。 |
| 7 | P2 | [AI产品经理大模型教程](https://www.bilibili.com/video/BV1V181z5E75/) | AI大模型码农 / 2025-08-01 | 11:53:18 | AI 产品经理入门到实战。 | 标题党倾向明显，只作章节备查。 |

Day 5 产出：500 字“AI研报助手”功能设计文档。比看视频更重要的是写出用户分层、能力边界、可解释、合规、兜底机制。

## Day 6：可解释 AI / 负责任 AI / 合规 / 架构

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [全流程：机器学习之可解释性分析-SHAP值](https://www.bilibili.com/video/BV1Xk4ZeCEX6/) | 代码解析与论文精读 / 2024-09-13 | 16:49 | SHAP 图、特征重要性、特征交互。 | 主线视频，直接转化为风控解释话术。 |
| 2 | P0 | [初学者的 SHAP 介绍视频，它的含义及其应用](https://www.bilibili.com/video/BV1oEUyYEEai/) | 茶桁 / 2024-11-21 | 04:38 | SHAP 含义和应用。 | 快速复盘。 |
| 3 | P1 | [李宏毅：机器学习模型的可解释性](https://www.bilibili.com/video/BV1Kh411v7cv/) | 爱学习的凉饭爷 / 2021-05-11 | 01:11:56 | Explainable ML 系统讲解。 | 时间够再看，补理论。 |
| 4 | P0 | [什么是联邦学习？](https://www.bilibili.com/video/BV1x6421Z7t8/) | Ph-D-Vlog / 2024-06-18 | 50:26 | 联邦学习概念、数据不出域、协同训练。 | 用来回答金融数据孤岛和隐私问题。 |
| 5 | P1 | [联邦学习技术介绍、应用和FATE开源框架](https://www.bilibili.com/video/BV1WE411s73n/) | 机器之心官方 / 2020-03-07 | 01:08:32 | 联邦学习应用和 FATE 框架。 | 只看应用场景部分。 |
| 6 | P0 | [《生成式人工智能服务管理暂行办法》解读](https://www.bilibili.com/video/BV15u411J7Aj/) | 爱智法商 / 2023-08-16 | 06:37 | 国内生成式 AI 管理办法要点。 | 必看，补监管表达。 |
| 7 | P1 | [基于 Flink 构建大规模实时风控系统在阿里巴巴的落地](https://www.bilibili.com/video/BV1ga411p7GW/) | Apache_Flink / 2022-07-06 | 24:18 | Flink 在实时风控中的落地。 | 用来解释实时风控架构。 |
| 8 | P1 | [Flink CEP 新特性进展与在实时风控场景的落地](https://www.bilibili.com/video/BV1Y8411j7Fm/) | Apache_Flink / 2022-11-30 | 38:19 | CEP 和实时规则匹配在风控中的应用。 | 时间够再看。 |

Day 6 产出：2 分钟“可解释 AI 在金融风控”口述稿。建议结构：为什么必须解释、SHAP 怎么解释、监管/合规怎么兜底、系统架构怎么留痕。

## Day 7：模拟面试与查漏补缺

| 顺序 | 重要度 | 视频 | UP/日期 | 时长 | 大致内容 | 观看建议 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | P0 | [【沉浸式大模型面试】RAG项目拷问](https://www.bilibili.com/video/BV1sV2PYCEo3/) | 丁师兄大模型 / 2024-10-09 | 04:19 | RAG 项目高频追问。 | 先看，马上自问自答。 |
| 2 | P0 | [12个RAG落地全链路必踩的致命雷区](https://www.bilibili.com/video/BV1VUVJ6EE6o/) | java架构师百里 / 8小时前 | 07:11 | RAG 落地问题和坑点。 | 用来补“幻觉、切片、召回、评估”。 |
| 3 | P0 | [AI大模型面试：RAG如何设计切片策略？](https://www.bilibili.com/video/BV1GXVJ61Etw/) | 扶安AI小课堂 / 8小时前 | 07:07 | RAG 切片策略追问。 | 高频追问，建议看。 |
| 4 | P1 | [AI产品经理｜某电商大厂面试拷打录音](https://www.bilibili.com/video/BV1SJUWBqEP8/) | AI产品经理-笑川 / 2025-11-22 | 28:02 | 产品经理面试追问节奏。 | 只学习回答结构，不照搬行业内容。 |
| 5 | P1 | [我是怎么拿到7家产品经理offer的？](https://www.bilibili.com/video/BV1Gy4y1r7jf/) | 斯前想后来 / 2020-10-18 | 10:17 | 产品经理面试表达。 | Day 7 下午模拟前看。 |
| 6 | P2 | [2025最新大模型面试题，RAG夺命10连问](https://www.bilibili.com/video/BV15HxkzbE1K/) | 楼兰教你学AI / 2025-10-09 | 05:14:02 | RAG 面试长课。 | 不完整看，只查十连问目录。 |
| 7 | P2 | [Langgraph+LangChain+Agent+RAG 大模型面试题合集](https://www.bilibili.com/video/BV1WhdcBjE5k/) | 大模型饼饼 / 05-07 | 20:04:29 | 大模型面试题合集。 | 超长资料库，只按问题检索。 |

Day 7 产出：8 个必练题录音复盘。每题控制 2 分钟，优先修正“是否金融化、是否具体、是否提到合规/可解释/人工兜底”。

## 7天最小必看清单

时间紧时，只看下面 15 个：

| 顺序 | 视频 | 时长 | 覆盖能力 |
| --- | --- | --- | --- |
| 1 | [面试自我介绍：面试官想听什么](https://www.bilibili.com/video/BV1jy4y1j7N6/) | 09:52 | 自我叙事 |
| 2 | [【闪客】一小时从函数到 Transformer](https://www.bilibili.com/video/BV1NCgVzoEG9/) | 01:09:17 | Transformer |
| 3 | [AI大模型是怎么炼成的](https://www.bilibili.com/video/BV1HhoTBeERS/) | 46:23 | 训练范式 |
| 4 | [SFT、RLHF、DPO](https://www.bilibili.com/video/BV1HYBWBaEE3/) | 19:23 | 对齐 |
| 5 | [LoRA是什么](https://www.bilibili.com/video/BV17i421X7q7/) | 05:40 | 微调 |
| 6 | [十分钟了解RAG基本原理](https://www.bilibili.com/video/BV1C69iBgEMk/) | 10:17 | RAG |
| 7 | [Prompt、RAG、MCP、Agent 一条线讲清](https://www.bilibili.com/video/BV1WTRzBWEsz/) | 27:15 | 技术路线 |
| 8 | [RAG项目拷问](https://www.bilibili.com/video/BV1sV2PYCEo3/) | 04:19 | 面试追问 |
| 9 | [因子投资基础](https://www.bilibili.com/video/BV1VxZcBqEsj/) | 13:32 | 量化 |
| 10 | [Fama French三因子](https://www.bilibili.com/video/BV1Py4y1j7x5/) | 16:24 | 因子模型 |
| 11 | [知识图谱在金融风控场景的应用](https://www.bilibili.com/video/BV1Zm4y1w7xF/) | 42:50 | 风控图谱 |
| 12 | [SHAP可解释性分析](https://www.bilibili.com/video/BV1Xk4ZeCEX6/) | 16:49 | XAI |
| 13 | [生成式AI管理办法解读](https://www.bilibili.com/video/BV15u411J7Aj/) | 06:37 | 合规 |
| 14 | [多Agent研报摘要生成平台](https://www.bilibili.com/video/BV1ucVn6gEhk/) | 11:16 | AI研报助手 |
| 15 | [Flink实时风控系统落地](https://www.bilibili.com/video/BV1ga411p7GW/) | 24:18 | 系统架构 |

## 搜索注意事项

- 搜索页比 API 更稳定。继续使用：`https://search.bilibili.com/all?keyword=<关键词>`。
- 不建议短时间批量调用 B站 API，容易触发 412。
- 产品经理、大模型面试、AI产品设计关键词标题党很多，优先选短视频或行业案例。
- 长课可以收藏，但 7 天冲刺不要完整追课。
- 每天必须先产出当天卡片/口述稿，再补视频。