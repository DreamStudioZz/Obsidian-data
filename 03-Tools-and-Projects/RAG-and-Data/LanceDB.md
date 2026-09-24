---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/lancedb/lancedb
stars: "16k+"
license: Apache-2.0
date_added: 2026-09-24
---

# 📦 LanceDB

> **一句话简介**：基于全新 Lance 列式存储格式构建的开源无服务器（Serverless）嵌入式向量数据库，支持零内存爆炸的磁盘直查、亚毫秒级向量与全文混合检索，为本地化与边缘轻量级 RAG 提供“SQLite 级”极简体验。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: lancedb/lancedb](https://github.com/lancedb/lancedb) |
| **核心特点** | 嵌入式进程内运行、Lance 磁盘列式存储、原生混合检索（BM25 + 向量）、DuckDB 零拷贝生态 |
| **技术栈** | Rust / Python / TypeScript / PyArrow |
| **关联实践** | [[轻量无服务器检索架构：基于 LanceDB 的列式存储与端侧混合检索实战]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]], [[AnythingLLM]] |

---

## 🚀 核心特性与技术亮点

1. **磁盘优先（Disk-First）与零内存爆炸架构**：
   - 传统向量数据库（如 Milvus、Qdrant、Pinecone）往往要求将百万级高维向量全部载入内存（RAM）以维持 HNSW 索引，造成高昂的基础设施成本。
   - LanceDB 基于专为 AI 设计的 **Lance 列式格式**，采用 IVF-PQ（倒排文件乘积量化）索引，数据和索引均持久化于磁盘/对象存储，检索时按需流式读取，内存占用减少 80%~90%。
2. **极简嵌入式运行（Embedded like SQLite）**：
   - 像 SQLite / DuckDB 一样直接在应用程序进程内加载运行，无需拉起独立的 Docker 守护进程、无需维护网络端口连接池，支持直接部署在 AWS Lambda、边缘设备或客户端桌面应用中。
3. **原生混合检索（Native Hybrid Search）**：
   - 内置 Tantivy 全文检索引擎，单表原生支持基于 BM25 的关键词全文检索与稠密向量余弦检索，并通过倒数排名融合（RRF）算法一键混合输出。
4. **与现代数据湖零拷贝集成**：
   - 深度兼容 Apache Arrow 生态，与 Polars、Pandas、DuckDB 零拷贝（Zero-Copy）互通，支持直接执行 SQL 复杂条件过滤（如 `price < 100 AND category = 'AI'`）。

---

## 🛠️ 快速上手与集成

### 1. 安装与初始化

```bash
pip install lancedb tantivy
```

### 2. 混合检索与过滤代码示例

```python
import lancedb
from lancedb.pydantic import LanceModel, Vector
from lancedb.embeddings import get_registry

# 1. 配置自动 Embedding 模型
func = get_registry().get("sentence-transformers").create(name="BAAI/bge-small-en-v1.5")

class DocumentSchema(LanceModel):
    id: int
    text: str = func.SourceField()
    vector: Vector(func.ndims()) = func.VectorField()
    category: str

# 2. 连接本地目录（像 SQLite 一样以文件持久化）
db = lancedb.connect("./data/lancedb_vault")
table = db.create_table("ai_docs", schema=DocumentSchema, mode="overwrite")

# 3. 写入样本数据（自动计算向量）
table.add([
    {"id": 1, "text": "LanceDB provides serverless vector search based on Lance format.", "category": "database"},
    {"id": 2, "text": "DeepEval is a unit testing framework for LLM hallucination and faithfulness.", "category": "eval"},
    {"id": 3, "text": "Firecrawl crawls dynamic websites and turns them into markdown for RAG.", "category": "crawler"}
])

# 4. 创建全文索引以支持混合检索
table.create_fts_index("text")

# 5. 执行混合检索（关键词 + 语义向量）与元数据过滤
query = "How to evaluate RAG hallucination?"
results = (
    table.search(query, query_type="hybrid")
    .where("category = 'eval'")
    .limit(2)
    .to_pandas()
)

print(results[["id", "text", "_score"]])
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **单机应用 / 桌面端应用（如 Obsidian 插件、客户端知识库）**：无需额外运行臃肿的数据库容器；
  - **中轻量级企业级微服务**：数据量在 100 万 ~ 1000 万条之间，追求极低维护成本与极快冷启动响应；
  - **Serverless / 边缘计算（AWS Lambda / Cloudflare Workers）**：直接以 S3 / OSS 为后端存储。
- **与传统集群式向量库对比**：
  - 相比 Milvus / ElasticSearch，开发测试极其轻便，无运维负担；
  - 在数十万级向量规模下，IVF-PQ 磁盘直查延迟仅需 1~5ms，吞吐与成本表现优异。
- **综合评估结论**：轻量级、端侧与 Serverless RAG 架构的首选存储引擎，强烈推荐采用（Adopted）。
