---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "Agent Memory & Context Engineering 记忆与上下文工程"
postSlug: agent-02-memory-context
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: Agent Memory & Context Engineering 记忆与上下文工程"
---

---
tags: [LLM, Agent, Memory, RAG, ContextEngineering]
created: 2026-05-22
---

# Agent Memory & Context Engineering 记忆与上下文工程

> 核心问题：**如何在有限的 Context Window 内塞入最有价值的信息？**  
> LLM 的上下文窗口是有限资源，如何"花好每一个 token"是工程的核心挑战。

---

## 一、短期记忆（Short-term Memory）

### 1.1 Sliding Window（滑动窗口）

最简单的上下文管理策略：只保留最近 N 轮对话，丢弃最早的历史。

**工程代价**：早期的重要信息会丢失（如用户的系统级偏好设置）

```python
from dataclasses import dataclass, field
from openai import OpenAI

client = OpenAI()


@dataclass
class Message:
    role: str    # "user" | "assistant" | "system"
    content: str


class SlidingWindowMemory:
    """
    滑动窗口记忆：保留最近 max_turns 轮对话
    system prompt 永久保留，不计入窗口
    """
    def __init__(self, max_turns: int = 10, system_prompt: str = ""):
        self.max_turns = max_turns
        self.system_prompt = system_prompt
        self._history: list[Message] = []  # 不含 system

    def add(self, role: str, content: str):
        self._history.append(Message(role=role, content=content))
        # 超出窗口则截断最早的消息（成对截断，保证 user/assistant 对齐）
        while len(self._history) > self.max_turns * 2:
            self._history.pop(0)

    def to_api_messages(self) -> list[dict]:
        """转换为 OpenAI API 格式"""
        messages = []
        if self.system_prompt:
            messages.append({"role": "system", "content": self.system_prompt})
        messages.extend({"role": m.role, "content": m.content} for m in self._history)
        return messages

    def chat(self, user_input: str) -> str:
        self.add("user", user_input)
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=self.to_api_messages(),
        )
        reply = response.choices[0].message.content
        self.add("assistant", reply)
        return reply

    @property
    def current_turns(self) -> int:
        return len(self._history) // 2


# 测试
memory = SlidingWindowMemory(max_turns=5, system_prompt="你是一个 AI 工程助手。")
memory.chat("我在用 vLLM 部署模型")
memory.chat("遇到了显存不足的问题")
print(f"当前保留 {memory.current_turns} 轮对话")
print(memory.chat("该怎么解决？"))
```

### 1.2 Summary Memory（摘要压缩）

当 token 数超过阈值时，调用 LLM 将历史对话压缩为摘要，替换原始历史。

**工程优势**：保留语义，而非丢弃
**工程代价**：压缩本身消耗一次 API 调用；压缩可能丢失细节

```python
import tiktoken
from openai import OpenAI

client = OpenAI()
# 使用 tiktoken 精确计算 token 数
tokenizer = tiktoken.encoding_for_model("gpt-4o-mini")


def count_tokens(text: str) -> int:
    return len(tokenizer.encode(text))


class SummaryMemory:
    """
    摘要记忆：超出 token 阈值时，将旧历史压缩为摘要
    """
    def __init__(self, max_tokens: int = 2000, system_prompt: str = ""):
        self.max_tokens = max_tokens
        self.system_prompt = system_prompt
        self._history: list[dict] = []
        self._summary: str = ""  # 历史的压缩摘要

    def _total_tokens(self) -> int:
        all_text = self._summary + " ".join(m["content"] for m in self._history)
        return count_tokens(all_text)

    def _compress(self):
        """将最早的一半历史压缩为摘要"""
        split_point = len(self._history) // 2
        to_compress = self._history[:split_point]
        self._history = self._history[split_point:]

        history_text = "\n".join(f"{m['role']}: {m['content']}" for m in to_compress)
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {
                    "role": "user",
                    "content": f"请将以下对话历史压缩为简洁的摘要（保留关键信息和决策）：\n\n"
                               f"旧摘要：{self._summary}\n\n新对话：\n{history_text}",
                }
            ],
            temperature=0,
        )
        self._summary = response.choices[0].message.content
        print(f"[SummaryMemory] 已压缩，摘要: {self._summary[:80]}...")

    def add(self, role: str, content: str):
        self._history.append({"role": role, "content": content})
        if self._total_tokens() > self.max_tokens:
            self._compress()

    def to_api_messages(self) -> list[dict]:
        messages = []
        if self.system_prompt:
            messages.append({"role": "system", "content": self.system_prompt})
        # 将摘要作为 system 级上下文注入
        if self._summary:
            messages.append({
                "role": "system",
                "content": f"[对话历史摘要]\n{self._summary}"
            })
        messages.extend(self._history)
        return messages

    def chat(self, user_input: str) -> str:
        self.add("user", user_input)
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=self.to_api_messages(),
        )
        reply = response.choices[0].message.content
        self.add("assistant", reply)
        return reply
```

---

## 二、长期记忆：RAG 全流程

### 2.1 RAG 架构总览

```
文档集合
  ↓  [Chunking]        按语义切割文档
  ↓  [Embedding]       转换为向量
  ↓  [Index]           存入向量数据库
                              ↓
用户查询 → [Query Embed] → [检索] → [Rerank] → [注入 Prompt] → LLM → 回答
```

### 2.2 Hybrid Search（混合检索 + RRF 融合）

**BM25**（关键词匹配）+ **向量检索**（语义匹配）互补：
- BM25 擅长精确匹配（如代码函数名、专有名词）
- 向量检索擅长语义匹配（同义词、改写）

**RRF（Reciprocal Rank Fusion）**：将两路结果融合，避免人工调权重

```python
from rank_bm25 import BM25Okapi  # pip install rank_bm25
import numpy as np
from sentence_transformers import SentenceTransformer  # pip install sentence-transformers
import faiss  # pip install faiss-cpu


class HybridSearchIndex:
    """
    混合检索：BM25（关键词） + FAISS（向量），RRF 融合排序
    """
    def __init__(self, model_name: str = "BAAI/bge-small-zh-v1.5"):
        self.embed_model = SentenceTransformer(model_name)
        self.documents: list[str] = []
        self.bm25: BM25Okapi | None = None
        self.faiss_index: faiss.IndexFlatIP | None = None

    def build(self, documents: list[str]):
        """建立索引"""
        self.documents = documents

        # BM25 索引（基于分词后的词频）
        tokenized = [doc.split() for doc in documents]
        self.bm25 = BM25Okapi(tokenized)

        # 向量索引（内积 = cosine，因为 BGE 输出是归一化向量）
        embeddings = self.embed_model.encode(documents, normalize_embeddings=True)
        dim = embeddings.shape[1]
        self.faiss_index = faiss.IndexFlatIP(dim)
        self.faiss_index.add(embeddings.astype("float32"))

        print(f"[Index] 已建立索引，文档数：{len(documents)}")

    def search(self, query: str, top_k: int = 5, rrf_k: int = 60) -> list[tuple[str, float]]:
        """
        混合检索 + RRF 融合
        rrf_k: RRF 平滑参数，通常取 60
        """
        # ── BM25 检索 ──────────────────────────────────────
        bm25_scores = self.bm25.get_scores(query.split())
        bm25_ranking = np.argsort(bm25_scores)[::-1][:top_k * 2]

        # ── 向量检索 ──────────────────────────────────────
        query_embed = self.embed_model.encode([query], normalize_embeddings=True)
        _, vector_ids = self.faiss_index.search(query_embed.astype("float32"), top_k * 2)
        vector_ranking = vector_ids[0]

        # ── RRF 融合 ──────────────────────────────────────
        # RRF score = 1 / (rank + k)，rank 从 1 开始
        rrf_scores: dict[int, float] = {}
        for rank, doc_id in enumerate(bm25_ranking, start=1):
            rrf_scores[doc_id] = rrf_scores.get(doc_id, 0) + 1 / (rank + rrf_k)
        for rank, doc_id in enumerate(vector_ranking, start=1):
            rrf_scores[doc_id] = rrf_scores.get(doc_id, 0) + 1 / (rank + rrf_k)

        # 按 RRF 分数降序排列
        sorted_ids = sorted(rrf_scores, key=lambda x: rrf_scores[x], reverse=True)[:top_k]
        return [(self.documents[i], rrf_scores[i]) for i in sorted_ids]


# 测试
docs = [
    "vLLM 使用 PagedAttention 优化 KV Cache 的显存管理",
    "BM25 是一种基于词频和文档频率的检索算法",
    "向量检索通过计算语义相似度找到相关文档",
    "混合检索结合了精确匹配和语义匹配的优势",
    "RAG 系统的核心是检索质量，决定了生成质量的上限",
    "Reranker 使用 Cross-Encoder 对候选文档重新打分",
]

index = HybridSearchIndex()
index.build(docs)
results = index.search("如何提高检索准确率？", top_k=3)
for doc, score in results:
    print(f"  [{score:.4f}] {doc}")
```

### 2.3 Cross-Encoder Reranker（重排序）

Bi-Encoder（FAISS）检索速度快但精度低；Cross-Encoder 精度高但速度慢。  
**最佳实践**：Bi-Encoder 召回 Top-50 → Cross-Encoder 精排 Top-5。

```python
from sentence_transformers import CrossEncoder


class CrossEncoderReranker:
    """
    Cross-Encoder 重排序
    模型同时看 query 和 document，比 Bi-Encoder 精度高 10-20%
    """
    def __init__(self, model_name: str = "cross-encoder/ms-marco-MiniLM-L-6-v2"):
        # 中文可用：BAAI/bge-reranker-base
        self.model = CrossEncoder(model_name)

    def rerank(
        self, query: str, candidates: list[str], top_k: int = 5
    ) -> list[tuple[str, float]]:
        """
        对候选文档重排序
        返回 (document, score) 列表，按分数降序
        """
        # 构建 (query, doc) 对
        pairs = [[query, doc] for doc in candidates]
        scores = self.model.predict(pairs)

        # 按分数降序排列
        ranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
        return ranked[:top_k]


# 完整 RAG Pipeline（混合检索 + 重排序）
def rag_pipeline(query: str, index: HybridSearchIndex, reranker: CrossEncoderReranker) -> str:
    from openai import OpenAI
    client = OpenAI()

    # Step 1: 粗召回（Top-20）
    candidates_with_scores = index.search(query, top_k=20)
    candidates = [doc for doc, _ in candidates_with_scores]

    # Step 2: 精排（Top-5）
    reranked = reranker.rerank(query, candidates, top_k=5)

    # Step 3: 构建 Context 注入 Prompt
    context = "\n\n".join([f"[{i+1}] {doc}" for i, (doc, _) in enumerate(reranked)])

    # Step 4: 生成
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": (
                    "你是一个问答助手。请严格基于以下参考资料回答问题，"
                    "不要添加参考资料中没有的信息。"
                    "如果参考资料不足以回答问题，请明确说明。\n\n"
                    f"参考资料：\n{context}"
                ),
            },
            {"role": "user", "content": query},
        ],
        temperature=0,
    )
    return response.choices[0].message.content
```

---

## 三、AST-based Code Chunking（代码专用切割）

**痛点**：按固定字符数切割代码会破坏函数/类的完整性，导致语义丢失。  
**方案**：用 Python `ast` 模块解析代码，按函数/类边界切割。

```python
import ast
import textwrap
from dataclasses import dataclass


@dataclass
class CodeChunk:
    type: str          # "function" | "class" | "module_level"
    name: str
    code: str          # 完整代码（含签名）
    signature: str     # 仅签名 + docstring（用于摘要索引）
    start_line: int
    end_line: int


class ASTCodeChunker:
    """
    基于 AST 的 Python 代码切割器
    切割粒度：顶层函数 + 类（含类方法）
    """

    def chunk(self, source_code: str, file_path: str = "") -> list[CodeChunk]:
        tree = ast.parse(source_code)
        lines = source_code.splitlines()
        chunks: list[CodeChunk] = []

        for node in ast.walk(tree):
            # 只处理顶层函数和类（不递归进入方法）
            if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef, ast.ClassDef)):
                # 跳过嵌套定义（parent 不是 Module 的话跳过）
                # 简单判断：通过行号范围是否在已有 chunk 内
                start = node.lineno - 1
                end = node.end_lineno
                code = "\n".join(lines[start:end])

                # 提取签名 + docstring
                signature = self._extract_signature(node, lines)

                chunk_type = "class" if isinstance(node, ast.ClassDef) else "function"
                chunks.append(CodeChunk(
                    type=chunk_type,
                    name=node.name,
                    code=code,
                    signature=signature,
                    start_line=node.lineno,
                    end_line=node.end_lineno,
                ))

        return chunks

    def _extract_signature(self, node: ast.AST, lines: list[str]) -> str:
        """提取函数/类的签名行 + docstring（用于摘要向量索引）"""
        start = node.lineno - 1
        sig_lines = [lines[start]]

        # 如果有 docstring，提取前 3 行
        if (
            isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef, ast.ClassDef))
            and node.body
            and isinstance(node.body[0], ast.Expr)
            and isinstance(node.body[0].value, ast.Constant)
        ):
            docstring = node.body[0].value.value
            doc_preview = "\n".join(docstring.splitlines()[:3])
            sig_lines.append(f'    """{doc_preview}"""')

        return "\n".join(sig_lines)


# 测试：切割当前文件
sample_code = '''
def calculate_rrf_score(rankings: list[list[int]], k: int = 60) -> dict[int, float]:
    """
    计算 Reciprocal Rank Fusion 分数
    参数：rankings - 多路检索结果的排名列表
    返回：每个文档 ID 的 RRF 分数
    """
    scores = {}
    for ranking in rankings:
        for rank, doc_id in enumerate(ranking, start=1):
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (rank + k)
    return scores


class VectorIndex:
    """向量索引类，封装 FAISS 操作"""

    def __init__(self, dim: int):
        self.dim = dim
        self.index = None

    def build(self, vectors):
        """建立索引"""
        import faiss
        self.index = faiss.IndexFlatIP(self.dim)
        self.index.add(vectors)
'''

chunker = ASTCodeChunker()
chunks = chunker.chunk(sample_code)
for chunk in chunks:
    print(f"[{chunk.type}] {chunk.name} (L{chunk.start_line}-{chunk.end_line})")
    print(f"  签名: {chunk.signature}")
    print()
```

---

## 四、Lost in the Middle 问题

### 问题描述

研究表明，LLM 对长上下文中**开头和结尾**的内容关注度高，**中间部分**容易被忽略。  
当 RAG 检索到 10 个文档时，真正有用的文档如果排在第 5-7 位，模型可能忽略它。

### 工程应对策略

```python
def reorder_for_attention(chunks: list[str]) -> list[str]:
    """
    对检索结果重排序，将最重要的内容放在首尾
    研究：Lost in the Middle (Liu et al., 2023)
    """
    if len(chunks) <= 2:
        return chunks
    
    # 将奇数位置放首部，偶数位置放尾部
    # 最相关的 chunk（index 0）放最前，第二相关放最后
    n = len(chunks)
    reordered = []
    
    # 最重要的放首部
    reordered.append(chunks[0])
    
    # 次重要的放尾部（反序）
    tail = []
    for i in range(1, n):
        if i % 2 == 1:
            tail.append(chunks[i])
        else:
            reordered.append(chunks[i])
    
    return reordered + tail[::-1]


# 更简单的工程策略：只取 Top-3，不贪心取 Top-10
# 更少的 context = 更少的干扰 = 更高的 Faithfulness
```

---

## 五、RAG 系统诊断速查表

| 症状 | 可能原因 | 解决方案 |
|------|---------|---------|
| 回答质量差但检索结果看起来相关 | Lost in the Middle | 减少 Top-K 数量；重排相关文档到首尾 |
| 检索不到明显相关的文档 | Chunking 粒度太大/太小 | 调整 chunk size；尝试 AST/sentence 级切割 |
| 专有名词检索失败 | 纯向量检索对 OOV 词不敏感 | 加入 BM25 混合检索 |
| 语义相似但不相关的干扰文档被召回 | Bi-Encoder 精度不足 | 加入 Cross-Encoder Reranker |
| 模型回答超出 Context 范围（幻觉） | Faithfulness 不足 | 强化 system prompt 约束；Eval 监控 Faithfulness |
| 检索速度慢（>500ms） | FAISS 全量扫描 | 使用 HNSW 近似检索；分片索引 |
| 多跳推理失败 | 单次检索无法覆盖多个知识点 | 迭代检索（Query Decomposition）；GraphRAG |
| 长文档内容丢失 | Chunk 切割破坏了段落完整性 | 使用句子边界切割；增加 chunk overlap |

---

*← [[agent-01-planning-reasoning]] | 下一篇 → [[agent-03-tool-use-action]]*
