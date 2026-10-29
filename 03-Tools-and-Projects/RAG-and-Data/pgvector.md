---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/pgvector/pgvector
stars: "17k+"
license: PostgreSQL
date_added: 2026-10-29
---

# 📦 pgvector

> **一句话简介**：开源、工业级 PostgreSQL 原生高维向量相似度检索扩展。彻底打破“业务关系型数据库”与“独立专用向量数据库”之间的数据孤岛，让企业能够在单一 PostgreSQL 实例内同时享有强 ACID 事务保障、复杂多表 JOIN 关联、BM25 全文检索（tsvector）以及亚毫秒级的 HNSW/IVFFlat 向量近似近邻检索能力。原生支持 halfvec 半精度与二进制量化（bit），彻底终结企业数据跨库双写一致性脆弱与架构复杂度膨胀的顽疾。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: pgvector/pgvector](https://github.com/pgvector/pgvector) |
| **研发社区** | PostgreSQL 开源社区与核心作者 Andrew Kane 等 |
| **技术栈** | C, PostgreSQL Extension, SIMD (AVX-512, AVX2, ARM NEON) |
| **许可证** | PostgreSQL License |
| **核心特性** | 原生 SQL/ACID 事务支持、HNSW 图索引与 IVFFlat 聚类索引、L2/Cosine/Inner Product 统一距离度量、半精度 halfvec (FP16)、二进制位图向量 (bit)、迭代式索引扫描 (Iterative Index Scans)、单语句多路混合检索 (Hybrid Search with RRF) |
| **相关关联** | 关联实践: [[企业级关系型与向量融合检索：基于 PostgreSQL 与 pgvector 的 HNSW 索引调优与混合检索实战]], [[2026-10-29-AI-Digest]], [[Chroma]], [[Qdrant]], [[Milvus]], [[Weaviate]], [[LanceDB]], [[Dify]], [[Letta]], [[Agno]] |

---

## 🚀 核心特性与架构亮点

### 1. 终结双写一致性灾难：统一数据底座与原生 ACID 事务
在构建企业级 RAG 或智能体记忆系统时，传统做法通常采用“主关系库（MySQL/PG 存业务）+ 独立向量库（Milvus/Qdrant 存 Embedding）”的双轨架构：
- **致命痛点**：由于缺乏分布式事务，系统经常在主表写入成功后因网络抖动或崩溃导致向量库写入失败，引发“有单据无向量”或“有向量无单据”的脏数据；更新或软删除时更是需要通过 Kafka/Debezium 维系复杂的异步 CDC 管道，运维成本极其高昂；
- **pgvector 破局**：向量作为 PostgreSQL 的原生一等公民数据类型（`vector(dim)`），向量的增删改查与业务实体的更新完全包裹在同一个数据库事务内部（`BEGIN ... COMMIT`），享受 100% 的 WAL 日志持久化、读写分离复制与快照隔离保护。

### 2. 工业级 HNSW 图索引与毫秒级高召回近似检索
pgvector 深入内核实现了高度优化的 **HNSW (Hierarchical Navigable Small World)** 图索引：
- **极速近邻检索**：通过多层跳表式的拓扑结构组织高维向量，在数百万级向量规模下，单次 Top-K 余弦相似度查询耗时通常压降在 2ms ~ 8ms 之间，召回率高达 98%+；
- **原生 SIMD 硬件加速**：内核代码底层深度适配 AVX-512、AVX2 及 ARM NEON 指令集，向量点积与欧氏距离计算直接由 CPU 矢量化单元极速执行；
- **平滑并发写入**：支持在持续高频数据插入的同时构建与更新 HNSW 索引，彻底避免早期内存向量库必须“停服构建索引”的弊端。

### 3. Halfvec (FP16) 与二进制量化 (Binary Quantization, bit) 显存/内存极致压缩
高维向量（如 OpenAI text-embedding-3-large 的 3072 维，或 BGE-M3 的 1024 维）在大规模数据下对宿主机内存提出了严苛挑战：
- **`halfvec` 半精度浮点**：将标准的 32 位浮点数（FP32）压缩为 16 位浮点数（FP16），索引占用空间与内存常驻直接**腰斩 50%**，而语义精度损失几乎为零（召回率衰减 < 0.1%）；
- **`bit` 二进制位图向量**：支持通过符号位量化（Sign Quantization）将向量转化为二进制位串，通过 CPU 原生的汉明距离（Hamming Distance / POPCOUNT 指令）执行超高速比对，内存占用**狂降至 1/32**，极其适合海量初筛（Coarse Search）管道。

### 4. 迭代式索引扫描（Iterative Index Scans）：彻底攻克元数据硬过滤痛点
传统向量数据库在应对 `WHERE tenant_id = 'A' AND is_deleted = false ORDER BY embedding <=> query LIMIT 10` 这种带复杂元数据过滤的查询时，经常遭遇**“后过滤死穴”**：
- **传统缺陷**：HNSW 索引按纯几何距离搜索 Top-50，随后在关系层过滤租户，结果发现这 50 个全部属于租户 B，导致最终向用户返回空结果（Over-filtering 问题）；
- **pgvector 0.7+ 架构突破**：引入原生迭代式索引扫描机制，如果前置搜索出的近邻不满足 `WHERE` 谓词，HNSW 扫描器会自动在图拓扑中动态下探搜索更深层级的节点，直到满足 `LIMIT` 数量为止，彻底保证复杂过滤下的精准召回！

### 5. 单一 SQL 语句原生实现 BM25 + 向量多路混合检索（RRF 融合）
无需在应用层编写繁重的双路请求与 Python 排序逻辑，直接利用 PostgreSQL 的 `tsvector`（全文关键词检索，BM25 变体）与 `pgvector`（语义稠密向量检索），通过一条优雅的 SQL 原生执行**互惠排名融合（Reciprocal Rank Fusion, RRF）**，一站式交付工业级混合检索。

---

## 🛠️ 快速上手与典型代码示例

### 1. 扩展启用与表结构定义

```sql
-- 1. 启用 pgvector 扩展
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. 创建具备业务字段、全文检索向量与语义稠密向量的企业文档表
CREATE TABLE enterprise_documents (
    id BIGSERIAL PRIMARY KEY,
    tenant_id VARCHAR(64) NOT NULL,
    department VARCHAR(64) NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    -- PostgreSQL 原生全文检索向量
    content_tsv TSVECTOR GENERATED ALWAYS AS (to_tsvector('simple', title || ' ' || content)) STORED,
    -- 1024 维稠密语义向量 (以 BGE-M3 嵌入为例)
    embedding VECTOR(1024) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- 3. 构建 HNSW 高性能近似近邻索引 (使用余弦距离 vector_cosine_ops)
CREATE INDEX idx_docs_embedding_hnsw ON enterprise_documents 
USING hnsw (embedding vector_cosine_ops) 
WITH (m = 16, ef_construction = 64);

-- 4. 构建业务元数据与全文检索 GIN 索引
CREATE INDEX idx_docs_tenant_dept ON enterprise_documents(tenant_id, department, is_deleted);
CREATE INDEX idx_docs_tsv_gin ON enterprise_documents USING GIN(content_tsv);
```

### 2. 单语句原生 SQL 实现“元数据硬隔离 + 全文 + 向量”RRF 混合检索

```sql
WITH 
-- 1. 稠密语义向量检索路 (Top-30)
dense_search AS (
    SELECT 
        id,
        ROW_NUMBER() OVER (ORDER BY embedding <=> '[0.012, -0.045, ...]'::vector) AS dense_rank
    FROM enterprise_documents
    WHERE tenant_id = 'finance_dept_01' AND is_deleted = FALSE
    ORDER BY embedding <=> '[0.012, -0.045, ...]'::vector
    LIMIT 30
),
-- 2. 全文关键词检索路 (Top-30)
sparse_search AS (
    SELECT 
        id,
        ROW_NUMBER() OVER (ORDER BY ts_rank_cd(content_tsv, plainto_tsquery('simple', '2026 财务审计 资产负债')) DESC) AS sparse_rank
    FROM enterprise_documents
    WHERE tenant_id = 'finance_dept_01' AND is_deleted = FALSE
      AND content_tsv @@ plainto_tsquery('simple', '2026 财务审计 资产负债')
    ORDER BY sparse_rank ASC
    LIMIT 30
)
-- 3. 在 SQL 内核级通过 RRF (k=60) 算法融合双路结果并关联业务实体
SELECT 
    d.id,
    d.title,
    d.department,
    d.created_at,
    COALESCE(1.0 / (60 + ds.dense_rank), 0.0) + 
    COALESCE(1.0 / (60 + ss.sparse_rank), 0.0) AS rrf_score
FROM enterprise_documents d
LEFT JOIN dense_search ds ON d.id = ds.id
LEFT JOIN sparse_search ss ON d.id = ss.id
WHERE ds.id IS NOT NULL OR ss.id IS NOT NULL
ORDER BY rrf_score DESC
LIMIT 10;
```

---

## 📊 架构方案横向对比

| 评测维度 | pgvector (PostgreSQL) | 独立向量库 (如 Milvus / Qdrant) | 嵌入式向量库 (Chroma / LanceDB) |
| :--- | :--- | :--- | :--- |
| **架构拓扑** | **单一数据库单体闭环，零额外组件** | 独立分布式集群，需引入 CDC 同步 | 进程内运行或独立小型服务 |
| **事务支持 (ACID)** | **原生强一致性，支持行级锁与多表事务** | 仅最终一致性，跨系统无 ACID 保障 | 单机文件锁或弱一致性 |
| **关系多表 JOIN** | **无缝支持千亿级关系表任意 JOIN 关联** | 不支持或仅支持弱元数据标量过滤 | 仅支持简单的字段字典过滤 |
| **混合检索能力** | **单 SQL 原生支持 BM25 全文 + 向量 RRF** | 依赖自带的有限全文或外部 ES 插件 | 需应用层自行编排融合 |
| **运维成本** | **极低，复用企业现有 PG 备份/监控基础设施** | 较高，需独立规划节点、内存与容灾 | 极低（但仅限中小单机场景） |
| **极限扩展能力** | 百万至千万级向量性能卓越；亿级需分库分表 | 原生支持百亿级分布式水平分片扩展 | 适合十万至百万级轻量嵌入场景 |

---

## 💡 生产环境避坑与实战建议

1. **`maintenance_work_mem` 显存配置陷阱**：
   - 构建 HNSW 索引时，PostgreSQL 默认分配的 `maintenance_work_mem`（通常只有 64MB）会导致索引构建极其缓慢甚至退化；
   - **生产规范**：在执行 `CREATE INDEX ... USING hnsw` 前，在当前会话中显式提升内存：`SET maintenance_work_mem = '4GB';`，构建完成后恢复，索引构建速度可提升 10 倍以上。
2. **`hnsw.ef_search` 查询时动态参数调优**：
   - `ef_search` 决定了查询时在 HNSW 图中遍历的最大近邻候选池大小（默认值为 40）；
   - 对精度要求极高的场景（如法律法条、精准审计），建议在查询事务内执行 `SET LOCAL hnsw.ef_search = 100;`，可在增加不到 2ms 延迟的代价下将召回率从 95% 提升至 99.5%。
3. **500 万数据规模下的分级选型法则**：
   - 如果你的业务向量规模在 **1000 万以内**，且伴随复杂的租户隔离、权限管控与高频业务关系更新，**坚决首选 PostgreSQL + pgvector**，它将为你省去一整套 CDC 管道和分布式向量运维团队；
   - 只有当向量规模确定突破 **5000 万以上**且纯以离线高并发批量检索为主时，才考虑引入分布式向量数据库。
