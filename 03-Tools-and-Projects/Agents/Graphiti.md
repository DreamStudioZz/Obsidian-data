---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/getzep/graphiti
stars: "5.6k+"
license: Apache-2.0
date_added: 2026-10-17
---

# 📦 Graphiti

> **一句话简介**：由 Zep 团队开源的智能体**动态时序知识图谱（Dynamically Updating Temporal Knowledge Graphs）**与长效记忆架构。彻底解决传统向量检索无法感知“事实时序演变”与“状态覆盖失效”的致命缺陷，通过双时态建模、实时实体增量消歧与亚 10 毫秒混合检索，为自主 Agent 提供高保真、自进化的长期情境记忆。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: getzep/graphiti](https://github.com/getzep/graphiti) |
| **技术栈** | Python / Neo4j / FalkorDB / Kùzu / Memgraph |
| **许可证** | Apache-2.0 |
| **核心特性** | 动态时序图谱、双时态建模（Bi-temporal）、增量实体消歧融合、事实失效边失效打标、亚 10ms 混合检索（向量+图+全文） |
| **相关关联** | 关联实践: [[智能体时序记忆架构：基于 Graphiti 的动态时序知识图谱与状态演化检索实战]], [[2026-10-17-AI-Digest]], [[Mem0]], [[Letta]], [[LangGraph]], [[LlamaIndex]] |

---

## 🚀 核心特性与架构亮点

### 1. 突破静态向量检索：双时态建模（Bi-Temporal Modeling）
在长周期 Agent 与用户交互或业务运营中，事实是在不断随时间变化的（如：“用户以前住在西雅图，上周搬到了纽约”；“系统管理员将数据库密码临时修改为 X，并在两小时后恢复为 Y”）：
- **传统向量记忆的崩溃**：传统 Vector DB 依据语义相似度进行余弦检索，会将“住在西雅图”与“住在纽约”两个相互矛盾的 Chunk 同时召回，导致下游 Agent 产生逻辑幻觉；
- **有效时间（Valid Time）与事务时间（Transaction Time）解耦**：Graphiti 为图谱中的每条边显式标注 `valid_at`（事实发生时间）与 `invalid_at`（事实失效时间），以及系统感知记录的时间戳。当新事实到来时，旧事实边被自动标记为失效而不会被生硬物理删除，保留了完整的历史因果回溯链条。

### 2. 动态增量图谱构建（无需全量重构）
传统的知识图谱（Knowledge Graph）构建往往依赖极其沉重、昂贵的全量实体链接与图谱重构流水线：
- **流式提取与实体解析（Entity Resolution）**：Graphiti 采用分层小模型与确定性规则，在对话流或新文档进入时，实时抽取实体节点（Nodes）与关系边（Edges）；
- **跨会话实体融合消歧（Entity Deduplication）**：利用图嵌入与上下文相似度，自动判断“Alice”、“李经理”、“Tech Lead Alice”是否为同一实体并做拓扑合并，消除了图谱节点膨胀；
- **无须全局重索引**：新节点与新事实边以 $O(1)$ 或 $O(K)$（局部邻居）代价快速连接，实现毫秒级增量写入。

### 3. 三模合一的高效混合检索（Hybrid Search < 10ms）
Graphiti 深度结合了三种检索能力的互补优势：
1. **拓扑图遍历（Graph Traversal）**：沿实体关系多跳扩展，获取关联上下文与前置依赖（如：查询某项目的负责人及其下属的待办事项）；
2. **稠密语义检索（Dense Vector Search）**：通过节点与边属性的向量表征捕获模糊语义；
3. **BM25 关键词精确匹配（Full-Text Search）**：确保专有名词、流水号、代码符号等精准命中；
- **加权融合重排**：底层支持 FalkorDB、Neo4j、Kùzu（单机轻量级嵌入式图数据库）等多样化图存储引擎，保证复杂多跳时序查询在 10 毫秒内完成返回。

### 4. 框架无关与原生生态集成
- 原生兼容 **LangChain**、**LlamaIndex**、**CrewAI**、**AutoGen** 等主流智能体编排底座；
- 可直接作为工具函数封装为 **MCP（Model Context Protocol）** 服务，供 Claude Code、Cursor、Cline 等自主编程 Agent 直接调用长期项目历史上下文。

---

## 🛠️ 快速上手与典型代码示例

### 1. 安装与环境依赖

```bash
# 安装 Graphiti 核心库
pip install graphiti-core

# 启动 Neo4j 或 FalkorDB 实例 (以 FalkorDB Docker 容器为例)
docker run -p 6379:6379 -it --rm falkordb/falkordb:latest
```

### 2. 初始化 Graphiti 并增量写入时序交互数据

```python
import asyncio
from datetime import datetime, timezone
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType

async def main():
    # 1. 初始化 Graphiti 客户端，连接到本地 FalkorDB/Neo4j
    graphiti = Graphiti("bolt://localhost:7687", auth=("neo4j", "password"))
    
    # 2. 模拟第一轮对话输入 (建立初始实体状态)
    await graphiti.add_episode(
        name="对话记录_第1周",
        episode_body="张伟目前在研发一部负责高并发网关重构项目，项目预期截止日期是 2026年11月1日。",
        source=EpisodeType.message,
        source_description="团队周会记录",
        reference_time=datetime(2026, 10, 1, 10, 0, tzinfo=timezone.utc)
    )
    
    # 3. 模拟第二轮对话输入 (发生状态演变：负责人调岗与截止日顺延)
    await graphiti.add_episode(
        name="对话记录_第3周",
        episode_body="经管理层调整，张伟调往 AI 平台组，网关重构项目转由李雷负责，且因需求变更高并发网关截止日期顺延至 2026年12月15日。",
        source=EpisodeType.message,
        source_description="架构委员会纪要",
        reference_time=datetime(2026, 10, 17, 10, 0, tzinfo=timezone.utc)
    )
    
    # 4. 执行时序感知混合检索
    query = "高并发网关重构项目的当前负责人是谁？截止日期是什么时候？"
    results = await graphiti.search(query)
    
    print("=== 检索到的时序关联事实与当前状态 ===")
    for edge in results:
        print(f"关系: {edge.source_node.name} --[{edge.name}]--> {edge.target_node.name}")
        print(f"事实陈述: {edge.fact}")
        print(f"有效区间: {edge.valid_at} ~ {edge.invalid_at}")
        print("-" * 40)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **长生命周期客户成功与运维 Agent**：准确识别客户历史配置变更与当前真实环境状态，拒绝老配置引起的排障误导；
  - **个性化 AI 个人助理与副驾驶**：跨越数月对话持续更新用户的习惯、偏好、居住地与日程安排，旧习惯自动让位于新偏好；
  - **跨项目代码研发协同 Agent**：沉淀微服务架构迁移历程、接口废弃状态演变与模块所属人变更。
- **潜在不足 / 局限性**：
  - 依赖高质量的实体抽取小模型或 LLM，极度口语化或暗语隐喻的文本可能导致实体识别不准；
  - 相比纯向量数据库，图数据库维护成本略高（但在引入轻量嵌入式引擎如 Kùzu 后已大幅改善）。
- **个人评估结论**：**自主智能体长效记忆体系向时序图谱演进的杀手级框架（Adopted）**。
