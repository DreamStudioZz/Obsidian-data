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
- 现代推理基座：[[Ollama]]（本地私有化引擎）, [[vLLM]]（PagedAttention 高并发）, [[SGLang]]（RadixAttention 前缀缓存）
- 降本加速实战：[[私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构]], [[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]]

### 🤖 智能体与自主软件工程 (Agents & Coding Harnesses)
- 自主工程师与多智能体：[[OpenHands]]（SWE 级自主编程）, [[CrewAI]]（角色协作与事件流）, [[smolagents]]（代码驱动）, [[LangGraph]]（状态图）, [[PydanticAI]]（类型安全）
- 协议与终端 Harness：[[FastMCP]], [[Pi-Agent]], [[Browser-Use]]
- 实战沉淀：[[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[多智能体协同工程：基于 CrewAI 的角色编排与层级流（Flows）实践]], [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]]

### 🔍 深度文档解析与混合检索 (Advanced RAG & Knowledge Layer)
- 版面解析与图谱引擎：[[RAGFlow]]（深度版面理解与零幻觉溯源）, [[Docling]]（复杂版面解析）, [[LightRAG]]（双层知识图谱）, [[Graphify]]（本地代码图谱）
- 实战沉淀：[[企业级 RAG 深度文档解析：基于 RAGFlow 的视觉模板分块与防幻觉实战]], [[双层图谱增强检索：基于 LightRAG 的轻量化 GraphRAG 架构实践]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]]

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
