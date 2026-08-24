---
title: RAG 檢索增強生成實戰
type: concept
tags: [ai, rag, embedding, vector-db, llm, retrieval]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# RAG 檢索增強生成實戰

## 什麼是 RAG

RAG（Retrieval-Augmented Generation）解決 LLM 的兩個核心問題：
1. **知識截止日期**：LLM 不知道最新資訊
2. **幻覺（Hallucination）**：LLM 可能編造不存在的事實

RAG 的核心邏輯：
```
用戶問題 → 搜索相關文件 → 把文件塞進 Prompt → LLM 根據文件回答
```

---

## RAG 系統架構

```
Indexing Pipeline（離線）：
文件 → 切塊（Chunking）→ Embedding → 存入 Vector DB

Query Pipeline（線上）：
用戶問題 → Embedding → 向量搜索 → Top-K 文件 → LLM → 回答
```

---

## 核心組件

### 1. 文件切塊（Chunking）

切塊策略對 RAG 品質影響最大：

```python
# 固定大小切塊（簡單但效果差）
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,      # 每塊 token 數
    chunk_overlap=50,    # 重疊，防止上下文斷裂
    separators=["\n\n", "\n", "。", ".", " "],  # 優先在段落邊界切
)
chunks = splitter.split_text(document_text)
```

**切塊策略比較**：

| 策略 | 適用場景 | 優點 | 缺點 |
|------|---------|------|------|
| Fixed Size | 通用 | 簡單 | 可能截斷語意 |
| Semantic Chunking | 長文件 | 語意完整 | 計算成本高 |
| 段落/章節 | 結構化文件 | 自然邊界 | 大小不均 |
| Sliding Window | 密集資訊 | 不遺漏 | 重複度高 |
| Proposition | 問答型 RAG | 精準匹配 | 需額外處理 |

**Parent-Child Chunking**（推薦）：
```python
# 小 chunk 用於精準搜索，返回時附上父級大 chunk 給 LLM
# 子 chunk：256 token（高精準）
# 父 chunk：1024 token（豐富上下文）
```

---

### 2. Embedding 模型

```python
from openai import OpenAI
import anthropic
import numpy as np

# OpenAI Embedding（業界標準）
client = OpenAI()

def embed(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",  # 1536 維，便宜
        # model="text-embedding-3-large"  # 3072 維，更準
        input=text,
    )
    return response.data[0].embedding

# 本地 Embedding（免費，適合私有資料）
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("BAAI/bge-m3")  # 支援多語言（含繁中）
vector = model.encode("你好世界")
```

**相似度計算**：

```python
def cosine_similarity(a: list[float], b: list[float]) -> float:
    a, b = np.array(a), np.array(b)
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
```

---

### 3. Vector Database

| DB | 適用場景 | 特點 |
|----|---------|------|
| **pgvector** | 已有 PostgreSQL | 免費，整合簡單 |
| **Chroma** | 本地 / 小型專案 | 零配置，Python 原生 |
| **Qdrant** | 生產環境 | 高效能，豐富過濾 |
| **Pinecone** | 雲端全託管 | 最簡單，但有成本 |
| Weaviate | 混合搜索 | GraphQL API |

**pgvector（推薦：若已用 PostgreSQL）**：

```sql
-- 開啟擴展
CREATE EXTENSION vector;

-- 建表
CREATE TABLE documents (
    id        SERIAL PRIMARY KEY,
    content   TEXT,
    metadata  JSONB,
    embedding vector(1536)  -- OpenAI text-embedding-3-small 維度
);

-- 建立 HNSW 索引（推薦，速度快）
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- 向量搜索
SELECT id, content, 1 - (embedding <=> $1::vector) AS similarity
FROM documents
ORDER BY embedding <=> $1::vector
LIMIT 5;
```

**Chroma（快速原型）**：

```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("my_docs")

# 加入文件
collection.add(
    documents=["Python 是高階語言", "Go 適合高並發"],
    embeddings=[[0.1, 0.2, ...], [0.3, 0.4, ...]],  # 或讓 Chroma 自動 embed
    ids=["doc1", "doc2"],
    metadatas=[{"source": "wiki"}, {"source": "blog"}],
)

# 搜索
results = collection.query(
    query_texts=["什麼語言適合後端開發"],
    n_results=3,
    where={"source": "wiki"},  # 元資料過濾
)
```

---

### 4. 完整 RAG Pipeline

```python
import anthropic
from openai import OpenAI
import psycopg2

openai_client = OpenAI()
anthropic_client = anthropic.Anthropic()

def index_document(text: str, metadata: dict, conn):
    """將文件切塊、embedding、存入 pgvector"""
    chunks = splitter.split_text(text)
    for chunk in chunks:
        embedding = embed(chunk)
        conn.execute(
            "INSERT INTO documents (content, metadata, embedding) VALUES (%s, %s, %s)",
            (chunk, json.dumps(metadata), embedding),
        )
    conn.commit()

def retrieve(query: str, conn, top_k: int = 5) -> list[dict]:
    """語義搜索"""
    query_embedding = embed(query)
    rows = conn.execute(
        """
        SELECT content, metadata, 1 - (embedding <=> %s::vector) AS score
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT %s
        """,
        (query_embedding, query_embedding, top_k),
    ).fetchall()
    return [{"content": r[0], "metadata": r[1], "score": r[2]} for r in rows]

def generate(query: str, context_docs: list[dict]) -> str:
    """將檢索結果塞入 Prompt 讓 LLM 回答"""
    context = "\n\n".join(
        f"[來源 {i+1}] {doc['content']}"
        for i, doc in enumerate(context_docs)
    )
    message = anthropic_client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""根據以下資料回答問題。若資料不足，請說明。

<context>
{context}
</context>

問題：{query}""",
        }],
    )
    return message.content[0].text

def rag(query: str, conn) -> str:
    docs = retrieve(query, conn)
    return generate(query, docs)
```

---

## 進階技巧

### Hybrid Search（混合搜索）

結合語義搜索和關鍵字搜索，效果通常更好：

```python
# BM25（關鍵字） + 向量搜索，用 RRF 合併排名
def hybrid_search(query: str, top_k: int = 5):
    vector_results = vector_search(query, top_k=top_k * 2)
    bm25_results = bm25_search(query, top_k=top_k * 2)

    # Reciprocal Rank Fusion
    scores = {}
    k = 60
    for rank, doc in enumerate(vector_results):
        scores[doc.id] = scores.get(doc.id, 0) + 1 / (k + rank + 1)
    for rank, doc in enumerate(bm25_results):
        scores[doc.id] = scores.get(doc.id, 0) + 1 / (k + rank + 1)

    return sorted(scores.items(), key=lambda x: -x[1])[:top_k]
```

### Reranking（重排序）

搜索後用 Cross-Encoder 重新排序，精準度更高：

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("BAAI/bge-reranker-v2-m3")

def rerank(query: str, docs: list[str], top_k: int = 3) -> list[str]:
    pairs = [(query, doc) for doc in docs]
    scores = reranker.predict(pairs)
    ranked = sorted(zip(scores, docs), reverse=True)
    return [doc for _, doc in ranked[:top_k]]
```

### Query Rewriting（查詢改寫）

```python
def rewrite_query(original_query: str) -> list[str]:
    """生成多個搜索查詢，提高 recall"""
    response = anthropic_client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[{
            "role": "user",
            "content": f"""生成 3 個不同角度的搜索查詢，用來找尋回答以下問題的資料。
每行一個查詢，不要編號。

問題：{original_query}""",
        }],
    )
    queries = response.content[0].text.strip().split("\n")
    return [original_query] + queries  # 原始查詢 + 改寫版本
```

---

## RAG 評估指標

```python
# RAGAS 框架（最常用的 RAG 評估工具）
# pip install ragas

from ragas.metrics import faithfulness, answer_relevancy, context_recall
```

| 指標 | 評估什麼 | 理想值 |
|------|---------|-------|
| Faithfulness | 回答是否忠實於檢索到的文件 | > 0.8 |
| Answer Relevancy | 回答是否切題 | > 0.8 |
| Context Recall | 相關文件是否被檢索到 | > 0.7 |
| Context Precision | 檢索到的文件是否都相關 | > 0.7 |

---

## 常見問題與解法

| 問題 | 症狀 | 解法 |
|------|------|------|
| Chunking 太小 | 上下文不夠，回答片面 | 加大 chunk，或用 Parent-Child |
| Chunking 太大 | 干擾資訊多，精準度下降 | 縮小 chunk，加 reranker |
| 語義搜索搜不到關鍵字 | 專有名詞、代號搜不到 | Hybrid Search（加 BM25）|
| LLM 不遵循文件回答 | 幻覺仍然存在 | 加強 Prompt 約束，用 Citation |
| 索引更新困難 | 新文件未被搜索到 | 建立增量索引管道 |

---

## 相關頁面

- [[LangGraph Agent工作流設計]] — RAG 作為 Agent 的工具
- [[Prompt Engineering進階]] — 如何設計 RAG Prompt
- [[Claude Agent 設計模式]] — Claude 工具使用與 RAG 整合
- [[Claude API基礎與最佳實踐]] — Anthropic API 使用
