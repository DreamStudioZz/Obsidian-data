---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/milvus-io/milvus
stars: "32k+"
license: Apache-2.0
date_added: 2026-10-04
---

# 📦 Milvus

> **一句话简介**：全球领先的云原生分布式海量向量数据库，采用真正存算分离微服务架构，支持百亿级高维向量低延迟检索、分区键（Partition Key）多租户隔离与稠密/稀疏向量多路混合召回。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: milvus-io/milvus](https://github.com/milvus-io/milvus) |
| **核心特点** | 存算分离分布式架构、百亿级向量水平扩展、Partition Key 物理路由、Dense+Sparse 混合检索、支持 GPU 加速检索（Knowhere） |
| **技术栈** | Go / C++ / Python / MinIO / etcd / Pulsar / Kafka |
| **关联实践** | [[企业级百亿向量分布式检索：基于 Milvus 的分区键切分与稠密稀疏混合检索实战]], [[生产级高性能向量检索：基于 Qdrant 的 Payload 过滤与标量量化混合检索实战]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]] |

---

## 🚀 核心特性与技术亮点

1. **彻底的存算分离与无状态微服务体系**：
   - 架构清晰划分为四个平面：**协调层（Access Layer / Coordinator）**、**工作计算层（Worker Nodes: QueryNode / DataNode / IndexNode）**、**消息流总线（Log Broker: Pulsar / Kafka）**与**持久存储层（Storage: MinIO / S3 / etcd）**；
   - 核心检索计算节点（QueryNode）具备完全的无状态特性，可根据吞吐峰谷实现秒级弹性伸缩，故障节点可瞬间被漂移接管。
2. **百亿级向量的横向水平扩展与分区键（Partition Key）机制**：
   - 原生支持基于指定字段（如 `tenant_id`、`org_code`）自动将物理数据和检索请求定向路由至独立的物理 Partition，单集群可支持千万级租户；
   - 检索时直接利用 Partition Key 进行前置剪枝，查询开销从全表扫描降为指定分区遍历，避免多租户数据混布导致的缓存失效与性能雪崩。
3. **原生稠密（Dense）与稀疏（Sparse）混合检索**：
   - 支持在同一个 Collection 中定义多个向量字段，同时支持稠密语义向量（BERT / OpenAI / BGE）与稀疏权重向量（SPLADE / BM25）；
   - 内置强大的混合检索 API 与倒数排名融合（Reciprocal Rank Fusion, RRF）机制，一次请求完成多路召回与排名融合，解决语义漂移与专有名词精确匹配难题。
4. **多样化的高性能索引引擎（Knowhere）**：
   - 底层由高性能 C++ 检索库 Knowhere 提供算力支撑，支持 HNSW、IVF-FLAT、IVF-PQ、SCaNN 及 GPU 索引（GPU-IVF-FLAT / GPU-IVF-PQ），可根据内存预算灵活在检索精度与 QPS 之间实现最优权衡。

---

## 🛠️ 快速上手与配置

### 1. 安装 Python 客户端 SDK
```bash
pip install pymilvus
```

### 2. 构建混合检索 Collection 与插入数据
```python
from pymilvus import MilvusClient, DataType

client = MilvusClient(uri="http://localhost:19530")

# 创建包含稠密与稀疏向量的 Schema
schema = client.create_schema(auto_id=True, enable_dynamic_field=True)
schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="user_id", datatype=DataType.VARCHAR, max_length=64, is_partition_key=True)
schema.add_field(field_name="dense_vector", datatype=DataType.FLOAT_VECTOR, dim=768)
schema.add_field(field_name="sparse_vector", datatype=DataType.SPARSE_FLOAT_VECTOR)
schema.add_field(field_name="content", datatype=DataType.VARCHAR, max_length=2048)

# 配置索引参数
index_params = client.prepare_index_params()
index_params.add_index(
    field_name="dense_vector",
    index_type="HNSW",
    metric_type="COSINE",
    params={"M": 16, "efConstruction": 200}
)
index_params.add_index(
    field_name="sparse_vector",
    index_type="SPARSE_INVERTED_INDEX",
    metric_type="IP"
)

# 创建集合
client.create_collection(
    collection_name="enterprise_knowledge",
    schema=schema,
    index_params=index_params
)
print("Milvus 混合检索集合创建成功！")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **超大规模向量数据库（十亿~百亿级）**：单机向量库（如 LanceDB、轻量版 Qdrant、Chroma）无法容纳的超大容量生产集群；
  - **大型企业多租户知识中台**：需要通过 Partition Key 严格物理隔离各部门/租户数据的企业级 RAG 平台；
  - **严苛的高可用与实时写入场景**：Log Broker 确保分布式数据零丢失，计算节点崩溃秒级自愈。
- **潜在不足 / 局限性**：
  - 依赖组件较多（etcd, MinIO, Pulsar/Kafka），单机本地尝鲜调试或极简边缘设备部署复杂度显著高于轻量级向量库（如 Qdrant 单二进制包或 SQLite/LanceDB）。
- **个人评估结论**：**大型分布式架构首选基石（status/adopted）**。生产环境百亿规模混合召回的工业界标杆。
