# 📘 个人 AI 知识库日常使用与运作指南

本文档介绍本知识库的结构组织方式以及后续如何让我（AI 助手）每天为你收集沉淀前沿 AI 知识。

---

## 🔄 日常运作机制：如何让 AI 每日收集？

### 方式 1：随时直接在对话中触发（最轻量推荐）
每天你想获取最新动态或整理知识时，直接在对话中发送：
- **`今日AI日报`** 或 **`收集今日AI知识`**
- 我会自动：
  1. 联网追踪当天前沿的 GitHub Trending、顶会与业界开源的 AI 工具与最佳工程实践；
  2. 提炼具有实际生产落地价值的工具与方案；
  3. 按照标准模板自动生成 `01-Daily-Digest/YYYY/YYYY-MM/YYYY-MM-DD-AI-Digest.md`；
  4. 自动把高星工具/突破性实践提取为 `03-Tools-and-Projects/` 和 `04-Engineering-Practices/` 中的独立卡片，并建立双向链接；
  5. 自动完成 `git add`、规范化 `git commit` 并直接推送到远程仓库（`git push origin main`），实现云端全自动同步与备份。

### 方式 2：按需定制专项沉淀
当你遇到特定技术点或研究方向时，可以直接告诉我：
- *“帮我调研一下当前最火的 Coding Agent 开源项目，沉淀到工具卡片”*
- *“帮我写一份企业知识库构建中 Parent-Child Chunking 的最佳实践笔记”*
- 我会直接生成标准格式的 Markdown 卡片归档到对应目录。

---

## 🗂️ 目录与标签规范

- **`00-Home/`**：
  - `000-AI-Index.md`：知识库导航地图（MOC），整合各核心专题。
- **`01-Daily-Digest/`**：
  - 按 `年/年月/年月日-AI-Digest.md` 存放每日资讯流。
- **`03-Tools-and-Projects/`**：
  - 核心开源框架与工具库，按核心领域划分子目录归档：
    - `Agents/`：智能体框架、工具协议、记忆系统（如 PydanticAI, FastMCP, Mem0）
    - `Inference/`：推理加速引擎、本地部署底座、统一路由网关（如 vLLM, SGLang, LiteLLM）
    - `RAG-and-Data/`：向量检索、代码/知识图谱、数据清洗增强（如 Graphify）
    - `Evaluation-and-Ops/`：可观测性、LLM 评测打分、Prompt 管理（如 Langfuse）
- **`04-Engineering-Practices/`**：
  - 解决实际生产问题的工程架构方案、避坑指南与示例代码。
- **`99-Templates/`**：
  - 提供 Obsidian 标准模板，保证后续卡片格式一致与元数据（Frontmatter）规范。

---

## 💡 推荐 Obsidian 插件搭配

1. **Dataview**（必装）：
   - 可以通过类似 SQL 的语法动态检索知识库。例如在首页自动列出所有包含 `#ai-tool` 的项目列表。
2. **Omnisearch**：
   - 带来超强的本地语义混合搜索体验。
