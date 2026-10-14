---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/run-llama/llama_index
stars: "38k+"
license: MIT
date_added: 2026-10-14
---

# 📦 LlamaIndex

> **一句话简介**：业界事实标准的 LLM 数据连接与上下文增强框架，全面进化至**事件驱动工作流（LlamaIndex Workflows）**与属性图索引（Property Graph），支持非阻塞异步流水线、多智能体协同与生产级复杂 RAG 架构。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: run-llama/llama_index](https://github.com/run-llama/llama_index) |
| **技术栈** | Python / TypeScript |
| **许可证** | MIT |
| **核心定位** | 数据摄入连接、多策略索引、事件驱动多 Agent 编排、企业级检索增强 |
| **相关关联** | 关联实践: [[基于 LlamaIndex Workflows 的事件驱动型多智能体编排与异步 RAG 流水线实战]], [[2026-10-14-AI-Digest]], [[RAGFlow]], [[LightRAG]] |

---

## 🚀 核心特性与架构演进

### 1. 事件驱动工作流架构（LlamaIndex Workflows）
- **传统困境**：早期的 `QueryEngine`、`ChatEngine` 与硬编码链式调用（Chains / DAG）结构死板，难以表达非线性循环、反思重查、动态并行分支（Fan-out/Fan-in）与人工在环（Human-in-the-Loop, HITL）审批。
- **Workflows 突破**：
  - 彻底摒弃传统硬编码有向无环图，采用基于**事件发布-订阅（Event Pub/Sub）**模型的非阻塞异步状态机；
  - 任何执行单元只需装饰 `@step`，指定监听的 `Event` 类型并返回下游 `Event`，天然实现任意拓扑流转；
  - 细粒度 `Context` 状态管理，提供线程安全的数据存储与 step 间传参，支持流式事件广播与无感断点恢复。

### 2. 属性图检索（Property Graph Index）
- **超越传统知识图谱**：突破单纯实体-关系三元组（Subject-Predicate-Object）无法承载复杂属性和上下文的局限；
- **混合知识表示**：将节点和边赋予键值对属性（Properties），无缝融合向量相似度检索、实体链接、Cypher 声明式图查询与全文检索（BM25），召回准确率显著超越纯向量 RAG。

### 3. 全面模块化生态（LlamaHub 300+ 扩展）
- `llama-index-core`：极简轻量核心内核；
- 插件化生态：无缝对接几乎所有商业/开源 LLM（OpenAI, Anthropic, Ollama, vLLM）、向量库（Qdrant, Milvus, LanceDB）与复杂文档解析器（Docling, Crawl4AI）。

---

## 🛠️ 快速上手与典型代码

```bash
# 安装核心与 OpenAI 驱动
pip install llama-index-core llama-index-llms-openai
```

### 事件驱动工作流（Workflows）最小实战：

```python
import asyncio
from llama_index.core.workflow import (
    Workflow,
    Event,
    StartEvent,
    StopEvent,
    step,
    Context,
)

# 1. 定义强类型自定义事件
class ProcessingEvent(Event):
    payload: str

class QualityCheckEvent(Event):
    score: float
    data: str

# 2. 编排事件驱动工作流
class SmartRAGWorkflow(Workflow):
    @step
    async def ingest_and_parse(self, ctx: Context, ev: StartEvent) -> ProcessingEvent:
        query = ev.get("query")
        # 异步预处理与检索...
        await ctx.set("raw_query", query)
        return ProcessingEvent(payload=f"检索上下文: [{query}] 的相关切片")

    @step
    async def evaluate_quality(self, ctx: Context, ev: ProcessingEvent) -> QualityCheckEvent:
        # 评估检索内容信噪比
        score = 0.92  # 模拟评估分数
        return QualityCheckEvent(score=score, data=ev.payload)

    @step
    async def generate_response(self, ctx: Context, ev: QualityCheckEvent) -> StopEvent:
        if ev.score >= 0.8:
            return StopEvent(result=f"【高置信度回答】基于数据: {ev.data}")
        else:
            return StopEvent(result="【低置信度警告】检索上下文不足，触发兜底回答")

# 3. 运行工作流
async def main():
    wf = SmartRAGWorkflow(timeout=30.0, verbose=True)
    result = await wf.run(query="企业私有化部署架构规范")
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 💡 框架对比与选型评估

| 选型维度 | LlamaIndex (Workflows) | LangChain (LangGraph) | Dify |
| :--- | :--- | :--- | :--- |
| **核心抽象** | **事件驱动（Event-Driven）**，极简优雅 | 显式状态图（State Graph），较重 | 可视化低代码工作流画布 |
| **数据与 RAG 纵深** | ⭐⭐⭐⭐⭐（属性图、分块策略、多路检索天花板） | ⭐⭐⭐⭐（依靠集成，自身相对分散） | ⭐⭐⭐⭐（开箱即用，定制深度受限） |
| **异步流式与 HITL** | 原生 `ctx.stream_events`，简单轻量 | 原生支持，但节点定义相对繁琐 | 需通过界面事件节点配置 |
| **多智能体协同** | 基于事件广播与过滤，无锁解耦 | 基于图循环路由与条件边 | 基于多节点拓扑连线 |

### 生产评估结论：
- **强烈推荐采用**：适合构建需要深度数据连接、多源知识融合、非线性复杂跳转与需要高代码控制力的企业级 RAG 与自主智能体业务。
