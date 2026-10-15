---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/weaviate/weaviate
stars: "17k+"
license: BSD-3-Clause
date_added: 2026-10-15
---

# 📦 Weaviate

> **一句话简介**：开源云原生分布式向量数据库与搜索引擎，基于 Go 底层高性能架构，原生支持**动态多租户分级隔离（Dynamic Multi-Tenancy）**与热/冷分层存储，集成**产品量化（PQ）与二值量化（BQ）**极致内存压缩，结合 BM25 与密集向量混合搜索（Hybrid Search with RRF），专为大规模企业级 SaaS 与海量知识库量身定制。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: weaviate/weaviate](https://github.com/weaviate/weaviate) |
| **技术栈** | Go / C++ / Python Client |
| **许可证** | BSD-3-Clause |
| **核心特性** | 动态多租户与冷热离线存储, BQ/PQ 向量压缩（节省 90%+ 内存）, BM25 + 密集向量混合检索与 RRF 融合, gRPC 强类型 Collections API |
| **相关关联** | 关联实践: [[企业级多租户海量向量检索：基于 Weaviate 的动态租户隔离与 PQ-BQ 标量压缩实战]], [[2026-10-15-AI-Digest]], [[Milvus]], [[Qdrant]], [[LanceDB]] |

---

## 🚀 核心特性与架构亮点

### 1. 原生动态多租户分级隔离（Dynamic Multi-Tenancy）
- **真正的租户级物理隔离**：摒弃传统“在共享索引中添加 `tenant_id` 过滤”的脆弱方案（该方案会导致 HNSW 图检索质量退化与数据越权风险）；Weaviate 在 Collection 下为每个 Tenant 分配独立的向量与倒排分片；
- **热/冷分层生命周期（HOT / COLD / OFFLOADED）**：
  - **HOT**：高频活跃租户，索引全内存加载，毫秒级响应；
  - **COLD**：低频租户，将 HNSW 图与索引持久化至本地 SSD 或对象存储（S3/GCS），释放宝贵的 RAM 内存；
  - **秒级激活**：租户发起请求时按需瞬时加载（Lazy Loading），支撑单集群承载数万乃至数十万企业租户。

### 2. 极致向量量化与内存压缩（Product & Binary Quantization）
- **产品量化（Product Quantization, PQ）**：将高维浮点向量分解为低维子空间码本，节省 75%~85% 内存，支持带残差重打分（Rescoring）；
- **二值量化（Binary Quantization, BQ）**：将 32 位浮点数直接按符号投影为 1-bit 二进制数（0 或 1），内存开销降低 **96.8%**（例如 1000 万条 1536 维向量内存从 60GB 骤降至不到 2GB），配合现代 CPU 的 AVX-512 指令集执行汉明距离计算，吞吐性能提升数十倍。

### 3. 混合检索与倒数排序融合（Hybrid Search with RRF）
- **双模引擎融合**：单次查询同时触发 BM25 词法全文检索与密集向量语义检索；
- **倒数排序融合（Reciprocal Rank Fusion, RRF）**：结合关键词的精确匹配（如专业术语、料号、人名）与语义向量的模糊泛化能力，通过纯排名打分消除分数尺度的差异，召回率显著优于单一通道。

### 4. 新一代 Python Client v4 与 gRPC 二进制通信
- **全面转向 gRPC 原生通信**：批量导入与查询走 HTTP/2 高性能二进制通道，相比传统 REST/JSON 提升 60% 以上的网络吞吐；
- **面向对象的 Collections 强类型 API**：完全重构了旧版无类型的字典接口，提供完善的 IDE 代码提示与模式校验。

---

## 🛠️ 快速上手与典型代码

```bash
# 安装新版 Python Client v4
pip install -U weaviate-client
```

### 带有多租户、二值量化（BQ）与混合检索的完整 Python 示例：

```python
import weaviate
from weaviate.classes.config import (
    Configure, Property, DataType, MultiTenancyConfig, VectorIndexConfig
)
from weaviate.classes.tenants import Tenant, TenantActivityStatus
from weaviate.classes.query import MetadataQuery

# 1. 连接本地或云端 Weaviate 实例 (自动启用 gRPC 端口)
client = weaviate.connect_to_local(
    port=8080,
    grpc_port=50051
)

try:
    # 2. 定义带多租户与 BQ 压缩配置的 Collection
    collection_name = "EnterpriseKnowledge"
    if client.collections.exists(collection_name):
        client.collections.delete(collection_name)

    knowledge_base = client.collections.create(
        name=collection_name,
        # 启用动态多租户
        multi_tenancy_config=Configure.multi_tenancy(enabled=True),
        # 启用二值量化 (BQ) 极大压缩内存
        vector_index_config=Configure.VectorIndex.hnsw(
            quantizer=Configure.VectorIndex.Quantizer.bq(
                rescore_limit=100  # 初筛 100 条后用原始向量精排
            )
        ),
        properties=[
            Property(name="title", data_type=DataType.TEXT),
            Property(name="content", data_type=DataType.TEXT),
            Property(name="category", data_type=DataType.TEXT),
        ]
    )

    # 3. 动态注册多租户并分配状态
    knowledge_base.tenants.create([
        Tenant(name="tenant_corp_alpha", activity_status=TenantActivityStatus.HOT),
        Tenant(name="tenant_corp_beta", activity_status=TenantActivityStatus.HOT)
    ])

    # 4. 针对特定租户进行数据隔离写入
    tenant_alpha_col = knowledge_base.with_tenant("tenant_corp_alpha")
    
    tenant_alpha_col.data.insert(
        properties={
            "title": "Alpha 集团财务风控规范",
            "content": "单笔超过 50 万元的外部转账必须由 CFO 及法务总监双签审批。",
            "category": "财务制度"
        },
        # 传入浮点向量 (底层自动由 BQ 压缩为二进制码)
        vector=[0.025, -0.013, 0.088, 0.041] * 384
    )

    # 5. 执行针对特定租户的混合检索 (Hybrid Search)
    query_results = tenant_alpha_col.query.hybrid(
        query="50万转账财务审批流程",
        vector=[0.024, -0.010, 0.085, 0.040] * 384,
        alpha=0.5,  # 0.5 代表词法检索与语义检索各占一半权重
        limit=3,
        return_metadata=MetadataQuery(score=True, explain_score=True)
    )

    for item in query_results.objects:
        print(f"📄 命中文档: {item.properties['title']} (相关性得分: {item.metadata.score:.4f})")

finally:
    client.close()
```

---

## 💡 方案对比与选型评估

| 选型维度 | Weaviate | Qdrant | Milvus | Chroma |
| :--- | :--- | :--- | :--- | :--- |
| **多租户隔离方案** | ⭐⭐⭐⭐⭐（原生租户分片与冷热离线机制） | ⭐⭐⭐（依赖 Payload 过滤或创建不同 Collection） | ⭐⭐⭐⭐（支持 Partition Key 分区，但冷离线较复杂） | ⭐⭐（单机简单分离） |
| **内存极致压缩** | ⭐⭐⭐⭐⭐（原生支持 BQ + PQ，支持压缩后精排） | ⭐⭐⭐⭐（标量量化 SQ 与 PQ） | ⭐⭐⭐⭐（SQ8、FP16、PQ） | ⭐⭐（默认全精度常驻） |
| **混合检索集成** | 原生集成 BM25 + Dense + RRF | 原生支持 Dense + Sparse 向量 | 原生支持 Dense + BM25 Sparse | 基础过滤 |
| **通信与客户端协议**| HTTP/2 gRPC 二进制直连，强类型 API | gRPC / REST | gRPC 原生 | HTTP REST |
| **运维与交付难度** | 单二进制部署或多节点 Docker，极度轻量 | 单二进制 Rust 部署，极轻量 | 依赖 etcd、MinIO、Pulsar，架构较重 | 最轻量（嵌入式） |

### 生产评估结论：
- **SaaS 平台不可替代的利器**：在需要服务成千上万个企业客户、每个客户数据量有限但必须严密物理隔离、且大部分客户处于非活跃状态的场景下，Weaviate 的动态多租户与冷分层技术是行业内综合 TCO 最低的解决方案；
- **海量向量低成本常驻首选**：借助二值量化（BQ），可将千万级向量检索的内存需求压缩至数个 GB，结合 CPU AVX-512 指令集，兼顾极低成本与毫秒级延迟。
