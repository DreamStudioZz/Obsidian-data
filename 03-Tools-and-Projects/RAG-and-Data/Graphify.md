---
tags:
  - ai-tool
  - open-source
  - code-intelligence
  - knowledge-graph
  - status/adopted
category: RAG-and-Data
github: https://github.com/Graphify-Labs/graphify
pypi: graphifyy
license: MIT / Apache-2.0
date_added: 2026-09-21
---

# 📦 Graphify

> **一句话简介**：基于 AST 静态解析的本地代码知识图谱生成与检索工具，为 AI 编程助手提供结构化的代码依赖与调用关系，替代低效的扁平文本 grep。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库与包名** | GitHub: `Graphify-Labs/graphify` \| PyPI: `graphifyy` |
| **底层核心** | Tree-sitter AST 解析 + NetworkX 图遍历 + MCP 协议 |
| **适用环境** | Claude Code, Cursor, Antigravity, Gemini CLI, VS Code |
| **关联概念** | [[FastMCP]], [[000-AI-Index]], [[2026-09-21-AI-Digest]] |

---

## 🚀 解决了什么痛点？

在大型代码库（成千上万个文件）中，传统的 AI 编程助手通常面临两大瓶颈：
1. **扁平全文检索（Grep / Ripgrep）局限**：只能基于关键词文本匹配，无法理解“函数 A 调用了函数 B，函数 B 又继承自类 C”的语义调用链；
2. **上下文窗口与成本浪费**：盲目塞入全部文件会导致 Token 爆炸并引发大模型“中间迷失”，而切片检索又容易切断跨文件的依赖逻辑。

**Graphify 的核心突破**：
- **本地零成本解析**：使用 `tree-sitter` 在本地秒级解析完整语法树，无需调用 LLM，绝对保护代码隐私；
- **全链路图谱索引**：将整个代码库的关系结构化为有向图，让 AI 可以通过图遍历（BFS/DFS）顺藤摸瓜找到改动点影响的所有上下游。

---

## 📂 生成的 `graphify-out/` 产物结构

运行 Graphify 构建后，会在项目根目录生成 `graphify-out/` 文件夹：

```text
graphify-out/
├── graph.json        # 核心知识图谱数据（节点与边关系），供 AI / MCP 检索
├── GRAPH_REPORT.md   # 人类可读的架构分析报告（标注 God Nodes 与远距耦合）
└── graph.html        # 力导向图（Force-Directed）交互式可视化网页
```

- **`graph.html`**：直接双击浏览器打开，可以可视化拖拽、点击、搜索代码结构；
- **`GRAPH_REPORT.md`**：自动识别系统中的 **“上帝节点（God Nodes）”**（即出入度最高、依赖最密集的核心枢纽）。

---

## 🛠️ MCP 核心工具能力集

Graphify 提供了标准 MCP Server 接口，暴露给 AI 助手的典型工具包括：

| 工具名称 | 功能说明 | 典型使用场景 |
| :--- | :--- | :--- |
| `query_graph` | 基于自然语言或 BFS/DFS 搜索图谱 | “查找用户认证模块在哪些地方被调用” |
| `get_node` | 获取具体节点详情（类、函数、文件） | 快速读取某个方法签名及其元数据 |
| `get_neighbors` | 获取节点的上游调用方与下游依赖 | 评估修改一个函数前需要知晓的上下文 |
| `shortest_path` | 计算两个实体之间的最短调用路径 | “Controller 是如何一步步调用到 Repository 的” |
| `god_nodes` | 探测系统中依赖最集中的中枢节点 | 新人快速理清代码库架构主干 |
| `get_pr_impact` | 评估 PR 或代码改动的波及影响面 | 自动化精准回归测试、代码审查（Code Review） |

---

## 💻 快速安装与使用

```bash
# 1. 安装 (注意 PyPI 包名有两个 y)
pip install graphifyy

# 2. 为当前项目扫描并生成知识图谱
graphify init

# 3. 注册并安装到你的 AI 助手
graphify install
```

---

## 💡 工程实践建议
- **提交到 Git 共享**：建议将 `graphify-out/` 提交到 Git 仓库，这样团队所有成员或 CI/CD 流水线都可以直接共享代码图谱，免去重复解析。
- **与 PR Impact 审查结合**：在 CI 流程中挂载 `get_pr_impact`，改动代码时自动生成一份“改动波及分析清单”，极大降低重构风险。
