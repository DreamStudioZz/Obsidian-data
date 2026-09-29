---
tags:
  - moc
  - index
  - home
---

# 🧠 个人 AI 知识库仪表盘 (AI Knowledge Dashboard)

> [!TIP] 💡 Dataview 动态渲染说明
> 本页面已内置 **Dataview 动态查询组件**。
> 只要在 Obsidian 处于 **实时预览（Live Preview）** 或 **阅读视图（Reading View）**，下方表格就会自动从各个文件夹中扫描提取最新卡片并实时渲染成表格，无需手动维护！

---

## 🛠️ 开源工具与框架动态库 (Tools & Frameworks)

```dataview
TABLE category AS "分类", github AS "GitHub 仓库", date_added AS "收录日期"
FROM "03-Tools-and-Projects"
WHERE file.name != "Template-Project-Card"
SORT date_added DESC, file.name ASC
```

---

## 💡 工程实践与架构设计库 (Engineering Practices)

```dataview
TABLE domain AS "技术领域", difficulty AS "难度", date_added AS "收录日期"
FROM "04-Engineering-Practices"
WHERE file.name != "Template-Practice-Card"
SORT date_added DESC, file.name ASC
```

---

## 📰 最近 AI 日报与前沿雷达 (Recent Digests)

```dataview
TABLE date AS "发布日期", summary AS "核心导读"
FROM "01-Daily-Digest"
WHERE file.name != "Template-Daily-Digest"
SORT file.name DESC
LIMIT 7
```

---

## ⚡ 快捷专题核心脉络

### 💻 本地部署与高性能推理 (Inference & Deployment)
- 现代推理与微调基座：[[llama.cpp]]（纯 C/C++ 零依赖、GGUF 量化与投机解码基石）, [[Ollama]]（本地私有化引擎）, [[vLLM]]（PagedAttention 高并发）, [[SGLang]]（RadixAttention 前缀缓存）, [[Unsloth]]（Triton 手写内核单卡极限微调与 GRPO 对齐）
- 降本加速实战：[[端侧高吞吐低延迟推理：基于 llama.cpp 的 GGUF 极限混合量化与投机解码工程实战]], [[私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构]], [[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]], [[大模型极致后训练调优：基于 Unsloth 的显存节省与 LoRA 极速微调工程实战]]

### 🤖 智能体与应用编排平台 (Agents & Workflow Platforms)
- 应用中台与工作流平台：[[Dify]]（可视化工作流与 Agent 中台）
- 自主工程师与多智能体：[[Aider]]（终端 Git 原生架构师结对）, [[OpenHands]]（SWE 级自主编程）, [[CrewAI]]（角色协作与事件流）, [[smolagents]]（代码驱动）, [[LangGraph]]（状态图）, [[PydanticAI]]（类型安全）
- 协议与终端 Harness：[[FastMCP]], [[Pi-Agent]], [[Browser-Use]]
- 实战沉淀：[[终端代码智能体实践：基于 Aider 的 Repo Map 压缩与 Architect-Editor 双模型分工实战]], [[基于 Dify 的企业级 Agent 工作流与 RAG 落地架构实践]], [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[多智能体协同工程：基于 CrewAI 的角色编排与层级流（Flows）实践]], [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]], [[多模态网页自动化：基于 Browser-Use 的视觉驱动 Agent 设计与工程避坑]]

### 🔍 深度文档解析与混合检索 (Advanced RAG & Knowledge Layer)
- 版面解析与图谱引擎：[[MinerU]]（学术论文与复杂研报高保真 LaTeX/表格抽取）, [[AnythingLLM]]（开箱即用桌面/单容器知识库）, [[RAGFlow]]（深度版面理解与零幻觉溯源）, [[Docling]]（复杂版面解析）, [[LightRAG]]（双层知识图谱）, [[Graphify]]（本地代码图谱）, [[Firecrawl]]（网页智能清洗与反爬提取）, [[LanceDB]]（无服务器嵌入式列式向量检索）
- 实战沉淀：[[复杂文档高保真结构化：基于 MinerU 的论文研报公式还原与跨页版面解析实践]], [[企业级 RAG 深度文档解析：基于 RAGFlow 的视觉模板分块与防幻觉实战]], [[双层图谱增强检索：基于 LightRAG 的轻量化 GraphRAG 架构实践]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]], [[企业级 RAG 外部数据源清洗：基于 Firecrawl 的反爬对抗与智能 Markdown 结构化抽取实践]], [[轻量无服务器检索架构：基于 LanceDB 的列式存储与端侧混合检索实战]]

### 🧪 大模型评测、合规与质量安全 (Evaluation, Quality & Safety Guardrails)
- 质量评估与安全合规底座：[[NeMo-Guardrails]]（Colang 2.0 声明式安全护栏与运行时对齐）, [[DeepEval]]（大模型单元测试与 CI-CD 门禁）, [[Langfuse]]（全链路可观测与追溯）, [[DSPy]]（算法化提示词编译）
- 实战沉淀：[[企业级大模型可编程安全底座：基于 NeMo Guardrails 的对话护栏与防越狱拦截实战]], [[大模型应用质量工程：基于 DeepEval 的端到端指标评测与 CI-CD 自动化门禁实战]], [[Agent 生产级可观测性与评估：基于 Langfuse 的全链路追踪实践]], [[告别手工调优：基于 DSPy 的提示词与 Few-Shot 自动编译优化实践]]

---

## 🗺️ 知识库常规目录导航

| 模块 | 目录路径 | 说明 |
| :--- | :--- | :--- |
| 📰 **每日雷达 (Daily Digest)** | `[[01-Daily-Digest]]` | 每日由 AI 自动收集的精选工程实战技巧、新开源工具与行业动向 |
| 🏗️ **工程实践 (Practices)** | `[[04-Engineering-Practices]]` | Agent设计模式、RAG深度调优、Prompt最佳工程、本地部署加速 |
| 📦 **开源工具 (Tools & Projects)** | `[[03-Tools-and-Projects]]` | GitHub / HuggingFace 上经过验证的高质量框架与实用工具库 |
| 🧩 **核心概念 (Core Concepts)** | `[[02-Core-Concepts]]` | LLM原理、注意力机制、量化原理、上下文压缩等核心技术内功 |
| 📚 **精读与资料 (Resources)** | `[[05-Papers-and-Resources]]` | 经典论文笔记、优质博客、官方技术报告整理 |
| 📑 **卡片模板 (Templates)** | `[[99-Templates]]` | 日报模板、开源工具卡片、实践卡片规范 |
| 📘 **插件教程 (Tutorial)** | `[[Dataview插件使用说明与实战]]` | Dataview 语法、核心用法与拓展指令速查 |
