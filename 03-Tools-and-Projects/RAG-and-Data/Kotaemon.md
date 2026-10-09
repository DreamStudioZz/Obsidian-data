---
tags:
  - ai-tool
  - open-source
  - rag
  - document-qa
  - multimodal
  - status/adopted
category: RAG-and-Data
github: "https://github.com/Cinnamon/kotaemon"
stars: "18k+"
license: "Apache-2.0"
date_added: 2026-10-09
---

# 📦 Kotaemon

> **一句话简介**：生产级开源可定制 RAG 文档智能问答底座与 UI 框架，具备混合多路召回、重排精滤、多模态复杂图表解析及高保真溯源引用的企业级全流程架构。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - Cinnamon/kotaemon](https://github.com/Cinnamon/kotaemon) |
| **技术栈** | Python, Gradio, LangChain/LlamaIndex, SQLite/PostgreSQL |
| **许可证** | Apache-2.0 |
| **相关关联** | 关联专题: [[RAGFlow]], [[MinerU]], [[Docling]], [[Milvus]], [[Qdrant]] |

---

## 🚀 核心特性与亮点

1. **工业级混合多路检索与重排（Hybrid Retrieval & Re-ranking Pipeline）**：
   - 原生支持稠密语义向量（Dense Embedding）与稀疏关键词索引（BM25/全文检索）协同召回；
   - 深度内置 Cross-Encoder 重排器（Reranker，如 BAAI/bge-reranker），大幅压制长文本检索噪点并提升上下文精准度。
2. **多模态图表解析与复杂 PDF 支持**：
   - 支持高精度表格、图像与复杂版面结构识别，配合多模态大模型实现对图表、财务报表与技术规范的深度问答。
3. **原生精准引用与可视化划线定位（In-situ Verifiable Citations）**：
   - 生成回答中包含细粒度上标引用标记，点击可直接呼出内置 PDF 查看器并高亮召回片段所在原始物理页码与包围盒（Bounding Box），彻底解决回答溯源信任难题。
4. **灵活可定制的代码架构与开箱即用 UI**：
   - 前端基于 Gradio 打造，开箱即用支持多用户认证、权限隔离、公有/私有知识库切换；
   - 后端基于高度模块化的 `libs/kotaemon` 与 `libs/ktem`，可灵活插拔向量数据库（Chroma/Milvus/Qdrant）与推理底座（Ollama/vLLM/OpenAI）。
5. **与同类工具对比**：
   - **相比于 AnythingLLM**：Kotaemon 在企业级长文档解析、重排器集成与可视化 PDF 划线溯源机制上更为硬核、工程透明度更高；
   - **相比于 Dify 知识库模块**：Kotaemon 代码结构高度专注于端到端文档 RAG 管道优化，易于被二次开发为独立问答微服务。

---

## 🛠️ 快速上手与配置

```bash
# 克隆仓库并安装依赖
git clone https://github.com/Cinnamon/kotaemon.git
cd kotaemon
pip install -e "libs/kotaemon[all]"
pip install -e "libs/ktem"

# 启动 Gradio 交互式 Web 服务
python app.py
```

```python
# 基于 Kotaemon 核心库构建轻量混合检索流水线示例
from kotaemon.pipelines.retrieval import BaseRetrievalPipeline
from kotaemon.indices.vector import InMemoryVectorIndex
from kotaemon.indices.text import BM25Index
from kotaemon.rerankers import CohereRerank, FastEmbedRerank

# 1. 初始化混合检索流水线
class EnterpriseHybridPipeline(BaseRetrievalPipeline):
    def __init__(self):
        super().__init__()
        self.vector_index = InMemoryVectorIndex()
        self.bm25_index = BM25Index()
        self.reranker = FastEmbedRerank(model_name="BAAI/bge-reranker-base")

    def run(self, query: str, top_k: int = 5):
        # 多路召回
        vec_results = self.vector_index.search(query, top_k=20)
        bm25_results = self.bm25_index.search(query, top_k=20)
        
        # 候选集去重与合并
        merged_candidates = list({doc.id: doc for doc in vec_results + bm25_results}.values())
        
        # 交叉重排
        reranked = self.reranker.rerank(query, merged_candidates, top_k=top_k)
        return reranked
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 法律合规审查、医疗诊断参考、金融研报深度检索等对“事实正确性”与“证据溯源链条”有刚性要求的严苛业务；
  - 包含大量表格、多栏排版与技术蓝图的复杂 PDF 知识库；
  - 需要快速交付内网私有化部署、支持本地模型（Ollama）的开箱即用知识库问答门户。
- **潜在不足 / 局限性**：
  - Gradio 前端在面对超大规模高并发企业门户场景时扩展性略逊于 React/Vue 微前端体系，建议重度场景将后端 API 独立解耦。
- **个人评估结论**：**建议引入**。是当前开箱即用度最高、文档高保真溯源体验最好的开源 RAG 框架之一。
