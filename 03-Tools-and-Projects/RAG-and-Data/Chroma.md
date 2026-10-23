---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/chroma-core/chroma
stars: "16k+"
license: Apache-2.0
date_added: 2026-10-23
---

# 📦 Chroma

> **一句话简介**：被誉为“向量数据库领域的 SQLite”。开源、AI 原生、轻量级嵌入式向量存储引擎，专为快速构建 LLM 应用与 RAG 检索管线设计。支持无服务器内存模式（In-Memory）、单机持久化存储（PersistentClient）与轻量化微服务集群（Client-Server），具备开箱即用的自动 Embedding 处理与多维结构化元数据过滤能力。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: chroma-core/chroma](https://github.com/chroma-core/chroma) |
| **技术栈** | Python / Rust (自 0.5+ 起逐步采用 Rust 重写核心索引层) / SQLite / ClickHouse / HNSW |
| **许可证** | Apache-2.0 License |
| **核心特性** | 零配置本地嵌入运行、多模态与文本开箱即用自动 Embedding、HNSW 高性能近似近邻索引、强大易用的元数据复合条件过滤（`$where` / `$where_document`）、原生事务原子性与去重 Upsert、全主流 LLM 框架无缝集成 |
| **相关关联** | 关联实践: [[轻量级嵌入式向量检索架构：基于 Chroma 的无服务器本地持久化与多维元数据过滤实战]], [[2026-10-23-AI-Digest]], [[Open-WebUI]], [[Qdrant]], [[Milvus]], [[Weaviate]], [[LanceDB]], [[FlagEmbedding]] |

---

## 🚀 核心特性与架构亮点

### 1. “向量领域的 SQLite”：双模极简架构设计
在海量 AI 应用与中轻量级工程场景中，部署一套重型分布式向量集群（如 Milvus 或 Kubernetes 托管的 Weaviate）不仅硬件开销沉重，还会带来巨大的配置与维护心智负担。Chroma 提供了绝妙的工程折中：
- **无感嵌入式持久化（Embedded Persistent Mode）**：直接在应用进程内部以 Python 原生库的形式引入（`chromadb.PersistentClient(path="./chroma_data")`），数据直接以 SQLite（元数据）与本地文件（HNSW 向量索引）沉淀在指定目录，无需安装任何前置数据库服务或 Docker 容器；
- **微服务客户端模式（Client-Server Mode）**：当需要跨进程、跨微服务多客户端共享同一向量库时，仅需通过一行命令或官方 Docker 镜像启动独立的 Chroma 服务端，客户端代码几乎零成本切换（`chromadb.HttpClient(host="...", port=8000)`）。

### 2. 开发者友好的开箱即用自动 Embedding 管线
传统向量数据库要求用户在存入数据前必须自行调用模型生成高维浮点数列表（`embeddings=[[0.1, ...]]`），这在快速原型与常规应用中增加了大量样板代码。Chroma 彻底简化了这一流程：
- **文本直入直出**：调用 `collection.add(documents=["...", "..."])`，Chroma 会自动调用内置的轻量嵌入函数（默认使用 `all-MiniLM-L6-v2`）完成文本向量化；
- **可插拔自定义 Embedding Function**：支持一行代码替换为 OpenAI、Hugging Face Transformers、Cohere 或国内开源的 [[FlagEmbedding]] (BGE-M3) 等高性能嵌入模型。

### 3. 结构化元数据复合过滤（Metadata Filtering）
在生产级 RAG 检索中，纯向量相似度往往无法完全满足复杂的业务规则（如“仅检索特定租户、特定部门且在 2026 年创建的文档”）。Chroma 提供了表达力极强的元数据过滤引擎：
- **多条件布尔逻辑查询**：原生支持 `$and`、`$or` 复合操作符，支持 `$eq`、`$ne`、`$gt`、`$gte`、`$lt`、`$lte`、`$in`、`$nin` 等多维度字段过滤；
- **文档全文关键词匹配（`$where_document`）**：支持在向量相似度检索的同时，硬性约束文档文本中必须包含（`$contains`）或不包含特定的业务敏感关键字；
- **先过滤后检索（Pre-Filtering）与 HNSW 联合优化**：利用 SQLite 高速元数据初筛缩小候选 ID 范围，随后在 HNSW 索引中快速定位最相似向量，避免全库暴力扫描。

---

## 🛠️ 快速上手与典型代码示例

### 1. 本地持久化与自定义 BGE-M3 嵌入模型实战

```python
import chromadb
from chromadb.utils import embedding_functions

# 1. 初始化本地持久化客户端（数据落地至 ./chroma_db 目录）
client = chromadb.PersistentClient(path="./chroma_db")

# 2. 定义自定义 Embedding 函数（使用 Hugging Face 优质开源模型，如 BAAI/bge-small-zh-v1.5）
sentence_transformer_ef = embedding_functions.SentenceTransformerEmbeddingFunction(
    model_name="BAAI/bge-small-zh-v1.5",
    device="cpu" # 生产环境若有 GPU 可指定为 "cuda"
)

# 3. 创建或获取集合（Collection），并指定距离度量算法（cosine 余弦距离）
collection = client.get_or_create_collection(
    name="enterprise_knowledge_base",
    embedding_function=sentence_transformer_ef,
    metadata={"hnsw:space": "cosine"} # 支持 l2, ip, cosine
)

# 4. 幂等增量写入数据（Upsert）包含文档文本、结构化元数据与唯一 ID
collection.upsert(
    documents=[
        "企业考勤管理规定：员工每月享有1天带薪事假，需提前在 OA 提交申请并由直接主管审批。",
        "网络安全防护规程：研发环境禁止连接外部公网未经审核的代理服务器，违者记安全违规一次。",
        "报销审批流程：日常差旅费用必须在出差结束后 5 个工作日内提交财务系统凭证。"
    ],
    metadatas=[
        {"department": "HR", "level": "public", "year": 2026},
        {"department": "Security", "level": "confidential", "year": 2026},
        {"department": "Finance", "level": "public", "year": 2026}
    ],
    ids=["doc_hr_001", "doc_sec_002", "doc_fin_003"]
)

# 5. 执行联合多维条件过滤与向量近似检索
results = collection.query(
    query_texts=["请问事假需要提前几天申请？"],
    n_results=2,
    where={
        "$and": [
            {"department": {"$eq": "HR"}},
            {"year": {"$gte": 2026}}
        ]
    },
    where_document={"$contains": "OA"}
)

print("--- 检索结果 ---")
for doc, meta, dist in zip(results['documents'][0], results['metadatas'][0], results['distances'][0]):
    print(f"[匹配距离: {dist:.4f}] 来源部门: {meta['department']} | 内容: {doc}")
```

### 2. 轻量微服务独立部署（Docker 启动）

```bash
# 启动独立的 Chroma 服务端容器（监听 8000 端口）
docker run -d -p 8000:8000 \
  -v ./chroma_data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  -e PERSISTENCE_DIRECTORY=/chroma/chroma \
  --name chromadb \
  chromadb/chroma:latest
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **桌面客户端 / IDE 智能插件 / 移动端离线知识库**：对进程隔离与资源消耗极其敏感，需要零额外进程启动；
  2. **中轻量级 RAG 原型与中小企业私有知识库**：文档规模在 50 万条向量以内，无需承担 Milvus/Elasticsearch 复杂的集群运维成本；
  3. **单元测试与 CI/CD 自动化流水线**：单测时使用内存临时实例，测试完毕自动销毁，避免测试污染环境。
- **潜在不足 / 局限性**：
  1. **百亿级超大规模分布式瓶颈**：Chroma 核心定位于中轻量级与单机/小集群场景。对于千万级以上海量高频并发向量检索，建议选用 [[Milvus]] 或 [[Qdrant]]；
  2. **本地 SQLite 并发写锁**：在极端高并发写入场景下，SQLite 底层的写锁可能触发短暂等待，建议在应用层做好批量分批（Batching）与并发限流。
- **个人评估结论**：**轻量场景首选（status/adopted）**。开发体验极致优雅，是中小型 RAG 系统与本地智能体项目的标准向量底座。
