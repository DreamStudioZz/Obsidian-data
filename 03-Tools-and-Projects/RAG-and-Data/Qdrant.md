---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/qdrant/qdrant
stars: "22k+"
license: Apache-2.0
date_added: 2026-09-30
---

# 📦 Qdrant

> **一句话简介**：采用纯 Rust 编写的生产级云原生向量搜索引擎与数据库，独创 HNSW 紧密耦合 Payload 标量过滤机制，支持标量/乘积极致量化与稀疏-稠密多路混合检索，为严肃企业级 RAG 提供超低内存占用与毫秒级吞吐保障。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: qdrant/qdrant](https://github.com/qdrant/qdrant) |
| **核心特点** | Rust 极致内存性能、HNSW 原生 Payload 条件过滤、标量量化（SQ8）压缩 75% 内存、稠密+稀疏（SPLADE/BM25）混合多路召回 |
| **技术栈** | Rust / gRPC / REST / HNSW / SIMD Acceleration |
| **关联实践** | [[生产级高性能向量检索：基于 Qdrant 的 Payload 过滤与标量量化混合检索实战]], [[轻量无服务器检索架构：基于 LanceDB 的列式存储与端侧混合检索实战]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]] |

---

## 🚀 核心特性与技术亮点

1. **Rust 零运行时开销与高并发稳定性**：
   - 彻底摆脱 Java JVM GC 停顿与 Python 解释器 GIL 锁的困扰，在高吞吐并发读写和密集向量距离计算下保持纳秒级响应抖动极小；
   - 深度利用现代 CPU 的 AVX-512 与 ARM Neon SIMD 指令集，单核距离计算吞吐极其优异。
2. **独创的 HNSW 与 Payload 标量过滤无缝融合**：
   - 传统向量数据库采用“先搜后滤”（Post-filtering 导致结果数量骤减甚至为空）或“先滤后搜”（Pre-filtering 导致全表扫描无法利用图索引）；
   - Qdrant 在构建 HNSW 图索引时，将结构化元数据（Payload）条件无缝整合至遍历跳转阶段，即使在大规模租户隔离、权限过滤和时间范围限定下，依然保持完整的 Top-K 召回率。
3. **内存友好型标量量化（Scalar Quantization - int8）与磁盘加载**：
   - 支持一键将 float32 原始高维向量压缩为 int8 整数向量，内存占用直接缩减至原来的 25%，且检索准确率保留 99% 以上；
   - 配合原始向量落盘（Mmap 存储）并在最终重排序阶段进行精准距离回算，让单台 32GB 内存机器能承载千万级稠密向量在线检索。
4. **原生多模态混合检索（Dense + Sparse Vectors）**：
   - 单个集合可同时存储语义稠密向量（Dense Vector，如 BGE/OpenAI Embedding）与关键词稀疏向量（Sparse Vector，如 BM25 / SPLADE）；
   - 服务端原生支持 Reciprocal Rank Fusion (RRF) 或加权融合打分，一条 API 请求即可完成语义与精确专有名词的双重召回。

---

## 🛠️ 快速上手与配置

### 1. Docker 快速部署

```bash
docker run -d -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage:z \
    qdrant/qdrant:latest
```

### 2. Python 客户端配置集合与量化

```python
from qdrant_client import QdrantClient
from qdrant_client.models import (
    VectorParams, Distance, ScalarQuantization,
    ScalarQuantizationConfig, ScalarType, PointStruct, Filter, FieldCondition, MatchValue
)

client = QdrantClient(url="http://localhost:6333")

# 1. 创建具备标量量化与 HNSW 优化的集合
client.create_collection(
    collection_name="enterprise_knowledge",
    vectors_config=VectorParams(size=1536, distance=Distance.COSINE),
    quantization_config=ScalarQuantization(
        scalar=ScalarQuantizationConfig(
            type=ScalarType.INT8,
            quantile=0.99,
            always_ram=True  # 将压缩后的量化索引常驻内存，原向量落盘
        )
    )
)

# 2. 插入带 Payload 元数据的知识块
client.upsert(
    collection_name="enterprise_knowledge",
    points=[
        PointStruct(
            id=1,
            vector=[0.05] * 1536,
            payload={
                "doc_id": "KB-2026-09",
                "department": "fintech",
                "security_level": "confidential",
                "content": "2026财年企业合规与跨境资金清算实施细则..."
            }
        )
    ]
)

# 3. 毫秒级带过滤条件混合检索
results = client.search(
    collection_name="enterprise_knowledge",
    query_vector=[0.05] * 1536,
    query_filter=Filter(
        must=[
            FieldCondition(key="department", match=MatchValue(value="fintech"))
        ]
    ),
    limit=5
)

for hit in results:
    print(f"得分: {hit.score:.4f}, 文档: {hit.payload['content'][:30]}...")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 严肃企业级生产环境、金融或电商行业的多租户私有化 RAG 检索中台；
  - 依赖复杂元数据过滤（权限系统、部门隔离、时间衰减、状态筛选）的高并发实时问答；
  - 资源预算受限但数据量级在百万至千万以上，需要通过标量量化压低服务器内存成本的团队。
- **潜在不足 / 局限性**：
  - 相比 SQLite/LanceDB 等纯内嵌单文件数据库，Qdrant 作为分布式服务需要单独运维容器或集群；
  - 超大规模十亿（Billion）级别以上的极端场景，在超大节点扩缩容复杂性上需要配合专用的 Kubernetes Operator 管理。
- **评估结论**：**极力推荐引入（Adopted）**。综合性能、内存占用率、过滤灵活性与 API 人性化设计，Qdrant 是目前独立向量数据库领域的绝对标杆之一。
