<div align="right">
  <a href="README.md">English</a> | <strong>中文</strong>
</div>

# 📘 个人 AI 知识库日常使用与运作指南

> 基于 **Obsidian** + **Dataview** + **AI 结对助手** 构建的工业级个人 AI 前沿雷达与工程实践知识库。

本文档介绍本知识库的结构组织方式，以及日常如何通过 AI 助手自动化收集、沉淀前沿技术卡片并自动同步至云端。

---

## 🔄 日常运作机制：如何触发 AI 自动收集？

### 方式 1：随时在对话中一键触发（最轻量推荐）
每天你想获取最新动态或整理知识时，直接在对话中发送：
- **`今日AI日报`** 或 **`收集今日AI知识`**
- AI 助手会自动执行以下全自动闭环：
  1. **前沿追踪**：联网追踪当天前沿的 GitHub Trending、顶会与业界开源的 AI 工具与最佳工程实践；
  2. **价值提炼**：提炼具有实际生产落地价值的工具架构与避坑方案；
  3. **日报归档**：按照标准模板自动生成 `01-Daily-Digest/YYYY/YYYY-MM/YYYY-MM-DD-AI-Digest.md`；
  4. **卡片沉淀**：自动把高星工具/突破性实践提取为 `03-Tools-and-Projects/` 和 `04-Engineering-Practices/` 中的独立卡片，并建立双向链接；
  5. **自动推送**：自动完成 `git add`、规范化 `git commit` 并直接推送到远程仓库（`git push origin main`），实现云端全自动同步与备份。

### 方式 2：按需定制专项沉淀
当你遇到特定技术点或研究方向时，可以直接交代：
- *“帮我调研一下当前最火的 Coding Agent 开源项目，沉淀到工具卡片”*
- *“帮我写一份企业知识库构建中 Parent-Child Chunking 的最佳实践笔记”*
- AI 助手会直接生成标准格式的 Markdown 卡片归档到对应目录，并自动推送到远程。

---

## 🗂️ 目录与标签规范

知识库根目录下按关注维度拆分为清晰的模块目录：

- **`00-Home/`**：
  - `000-AI-Index.md`：知识库导航地图（MOC），内置 Dataview 动态查询表格，自动按分类展示全部工具与实践。
- **`01-Daily-Digest/`**：
  - 资讯流按 `年/年月/年月日-AI-Digest.md` 存放（例如 `2026/2026-09/2026-09-21-AI-Digest.md`）。
- **`03-Tools-and-Projects/`**：
  - 核心开源框架与实用工具库，按四大核心领域划分子目录归档：
    - **`Agents/`**：智能体框架、工具协议、记忆系统（如 PydanticAI, FastMCP, Mem0）
    - **`Inference/`**：推理加速引擎、本地部署底座、统一路由网关（如 vLLM, SGLang, LiteLLM）
    - **`RAG-and-Data/`**：向量检索、代码/知识图谱、数据清洗增强（如 Graphify）
    - **`Evaluation-and-Ops/`**：可观测性、LLM 评测打分、Prompt 管理（如 Langfuse）
- **`04-Engineering-Practices/`**：
  - 解决实际生产问题的工程架构方案、避坑指南与示例代码（如 Prompt Caching 设计、多路召回与 Rerank 实践）。
- **`99-Templates/`**：
  - 提供 Obsidian 标准模板（日报模板、工具卡片模板、工程实践模板），保证卡片格式与 YAML Frontmatter 规范统一。

---

## 💡 推荐 Obsidian 插件搭配

1. **Dataview**（必装）：
   - 支持通过类似 SQL 的语法动态检索知识库。在首页 `000-AI-Index.md` 中自动扫描全库并实时渲染表格，无需手动维护索引。
2. **Omnisearch**：
   - 为 Obsidian 提供超强的本地语义混合搜索体验，快速定位历史技术沉淀。
