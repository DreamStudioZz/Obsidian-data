---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/microsoft/graphrag
stars: "24.8k+"
license: MIT
date_added: 2026-10-27
---

# 📦 GraphRAG

> **一句话简介**：由微软研究院（Microsoft Research）开源的基于知识图谱与分层社区划分的新一代检索增强生成（Graph-based RAG）框架，彻底突破传统向量检索无法回答全文档集“宏观概括与主旨洞察（Sensemaking）”的致命痛点。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: microsoft/graphrag](https://github.com/microsoft/graphrag) |
| **技术栈** | Python 3.10+ / NetworkX / Leiden 算法 / Pydantic / AsyncIO / Azure OpenAI |
| **许可证** | MIT License |
| **核心能力** | 自动化知识图谱抽取（LLM-driven Entity/Relationship Extraction）、分层 Leiden 社区发现（Hierarchical Community Detection）、多层级社区摘要生成（Hierarchical Summarization）、全局检索（Global Search Map-Reduce）、局部检索（Local Search 实体邻域拓展）、DRIFT 混合检索 |
| **相关关联** | 关联实践: [[企业级全局宏观知识检索：基于 Microsoft GraphRAG 的分层社区发现与 Global-Local 混合检索实战]], [[2026-10-27-AI-Digest]], [[LightRAG]], [[Graphify]], [[LlamaIndex]], [[Haystack]], [[RAGFlow]], [[Kotaemon]] |

---

## 🚀 核心架构与杀手级特性

### 1. 破解传统 Naive RAG 的“盲人摸象”困境（Sensemaking 难题）
传统的基于嵌入向量相似度（Vector RAG）的检索策略仅适用于**“细粒度事实查找（Point Query）”**（如“某合同第三条约定的违约金是多少？”）。但在面对以下两类企业核心诉求时表现极差：
- **宏观总结型问题（Global Sensemaking）**：“请总结这 500 篇行业研报中出现的主要技术瓶颈有哪些？”；
- **隐式关联多跳问答（Multi-hop Reasoning）**：“实体 A 与实体 D 之间是否存在间接利益关联？”。

向量检索只能根据语义相似度召回局部的 Top-K 文本块，无法窥见跨文档的全局结构。GraphRAG 通过构建知识图谱与分层社区摘要，首次将 RAG 的能力边界从“局部点状检索”扩展到“全集宏观归纳”。

### 2. 基于 Leiden 算法的分层社区发现（Hierarchical Community Detection）
GraphRAG 从非结构化文本中抽取出实体、关系以及实体协变量（Claims/Covariates）并构建全景知识图谱，随后引入复杂网络图论中的 **Leiden 社区划分算法**：
- **自底向上的分层聚类**：将高度密集的实体与关系聚类为微观社区（Level 0 / Level 1），再递归聚类为宏观高阶社区（Level 2 / Level 3）；
- **模块度最大化保证**：相比传统 Louvain 算法，Leiden 算法保证了划分后的社区子图均具有内部连通性，避免了分离孤岛和畸形大团。

### 3. 多层级社区报告自动化生成（Hierarchical Summarization）
针对聚类得到的每一个图社区（Community），GraphRAG 调用大语言模型执行结构化总结，生成一份包含以下维度的标准 **社区报告（Community Report）**：
- 社区核心主题与重要性评分（Weight / Rank）；
- 核心实体、代表性关系以及关键发现（Key Findings）；
- 潜在矛盾点或未解问题。
通过预先离线将海量碎片化文本压缩为分层社区摘要，为运行时的低成本、全局性问答奠定了数据基础。

### 4. 双擎检索范式：Global Search 与 Local Search

```
[GraphRAG 检索矩阵]
├── Global Search (宏观全局问答)
│   ├── 适用场景: 全文主题、趋势归纳、高阶洞察、宏观对比
│   └── 核心流程: 扫描高层级 Community Reports -> Map 阶段逐个评分提取要点 -> Reduce 阶段合并生成全局报告
└── Local Search (微观实体关联)
    ├── 适用场景: 特定实体属性、局部子图多跳关系、精确事实溯源
    └── 核心流程: 实体提取 -> 命中图谱中心节点 -> 拓展 1~2 度邻居与关联边 -> 召回关联原始文本块 -> 综合回答
```

### 5. DRIFT 检索模式（动态分层递归检索）
除了标准的 Global/Local 搜索，新版 GraphRAG 推出了 **DRIFT (Dynamic Reasoning and Inference with Flexible Traversal)** 搜索范式：
- 结合 Local Search 的快速定位与 Global Search 的发散探索，在回答多跳问题时根据初步检索到的节点动态决定下一步图游走分支，大幅降低了无关社区的 Token 消耗。

---

## 🛠️ 快速上手与配置

### 1. 安装与初始化工作空间
```bash
pip install graphrag

# 在指定目录初始化知识库工程结构
mkdir -p ./my_graphrag && cd ./my_graphrag
python -m graphrag init --root .
```
初始化后会生成核心配置文件 `settings.yaml` 与输入目录 `input/`。

### 2. 生产级 `settings.yaml` 核心配置样例
```yaml
encoding_model: cl100k_base
skip_workflows: []

llm:
  api_key: ${OPENAI_API_KEY}
  type: openai_chat
  model: gpt-4o-mini # 推荐日常抽取使用高性价比模型
  model_supports_json: true
  max_tokens: 4000
  temperature: 0.0
  concurrent_requests: 16

embeddings:
  llm:
    api_key: ${OPENAI_API_KEY}
    type: openai_embedding
    model: text-embedding-3-small

chunks:
  size: 800
  overlap: 100
  group_by_columns: [id]

entity_extraction:
  prompt: "prompts/entity_extraction.txt"
  entity_types: [organization, person, geo, event, technology, risk]
  max_gleanings: 1

community_reports:
  prompt: "prompts/community_report.txt"
  max_length: 2000
  max_input_length: 8000
```

### 3. 一键构建图谱索引
```bash
# 将待分析的 txt/pdf/md 放入 input/ 目录后执行
python -m graphrag index --root .
```

### 4. Python API 检索调用示例
```python
import asyncio
from graphrag.query.indexer_adapters import read_indexer_reports, read_indexer_entities
from graphrag.query.llm.oai.chat_openai import ChatOpenAI
from graphrag.query.structured_search.global_search.community_context import GlobalCommunityContext
from graphrag.query.structured_search.global_search.search import GlobalSearch

async def run_global_query():
    # 1. 初始化模型与上下文
    llm = ChatOpenAI(api_key="sk-...", model_name="gpt-4o-mini")
    reports = read_indexer_reports("./output/create_final_community_reports.parquet")
    entities = read_indexer_entities("./output/create_final_nodes.parquet")
    
    context_builder = GlobalCommunityContext(
        community_reports=reports,
        entities=entities,
        token_limit=12000,
    )
    
    # 2. 构建全局检索器
    search_engine = GlobalSearch(
        llm=llm,
        context_builder=context_builder,
        token_limit=12000,
        max_data_tokens=8000,
        map_llm_params={"max_tokens": 1000, "temperature": 0.0},
        reduce_llm_params={"max_tokens": 2000, "temperature": 0.0},
    )
    
    # 3. 执行宏观全局洞察问答
    query = "请全面总结本知识库中涵盖的核心技术架构演进方向以及潜在的安全合规风险。"
    result = await search_engine.asearch(query)
    print("=== 全局洞察响应 ===")
    print(result.response)

if __name__ == "__main__":
    asyncio.run(run_global_query())
```

---

## 📊 与同类 RAG 架构横向对比

| 评估维度 | Naive Vector RAG | Microsoft GraphRAG | LightRAG | Neo4j + Text2Cypher |
| :--- | :--- | :--- | :--- | :--- |
| **检索核心单元** | 文本切块（Chunk） | 实体、关系、分层社区报告 | 实体/关系双层子图 | 图数据库预定义节点/边 |
| **宏观总结能力 (Sensemaking)** | 极差（几乎无法回答） | **业界顶尖（Map-Reduce 社区聚合）** | 良好（双层图摘要） | 中等（需手写聚合 Cypher） |
| **细粒度实体事实查找** | 优秀 | 优秀（Local Search） | 优秀 | 卓越（严格 schema 匹配） |
| **图谱维护与构建成本** | 极低（仅 Embedding） | 较高（多轮 LLM 抽取与总结） | 中等（轻量化提示词单步构建） | 极高（需定义图 Schema 与本体） |
| **是否依赖专用图数据库** | 否（仅向量库） | **否（基于 Parquet/NetworkX 纯文件）** | 否（本地文件或 KV 存储） | 是（必须部署 Neo4j/TuGraph 集群） |
| **解释性与溯源证据链** | 仅文本块片段 | **实体、关系、社区报告全链路可追溯** | 知识图谱节点可查 | Cypher 执行路径透明 |

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **金融与研报行业大模型知识库**：需要对成百上千篇行业研报进行宏观主题对比、竞品战略盘点与跨财报风险分析；
  - **复杂政企内部制度与法务合规库**：需要回答“涉及跨国数据跨境传输的所有法律规定与违规后果汇总”等全局排查类诉求；
  - **科研与医学文献全景调研**：挖掘未知实体间的潜在弱关联与跨学科多跳因果链条。
- **潜在不足 / 局限性**：
  - **离线构建期 Token 消耗显著**：全量抽取实体与生成分层社区摘要需要大量 LLM 推理，需采用本地大吞吐模型（如 vLLM + Qwen2.5-72B）削减成本；
  - **高频增量更新支持尚需完善**：当新增单个文件时，社区结构的重算与合并相较纯向量检索具有更高计算开销。
- **个人评估结论**：**建议全面引入（adopted）**。作为当前破解 RAG 领域“宏观总结与全局洞察”行业共性难题的事实标准，是构建真正具备分析师水准的企业知识大脑的核心底座。