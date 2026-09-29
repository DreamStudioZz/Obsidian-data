---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/paul-gauthier/aider
stars: "36k+"
license: Apache-2.0
date_added: 2026-09-29
---

# 📦 Aider

> **一句话简介**：基于终端的 Git 原生 AI 结对编程神器，结合基于 Tree-sitter 的代码库全景 AST 地图（Repo Map）与双模型架构（Architect-Editor），在本地终端实现高精度的多文件协同重构与原子级版本控制。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: paul-gauthier/aider](https://github.com/paul-gauthier/aider) |
| **核心特点** | 终端命令行交互、Git 自动原子提交与回滚、基于 PageRank 的 Repo Map、Architect/Editor 双模型分工 |
| **技术栈** | Python / Tree-sitter / Git / LiteLLM |
| **关联实践** | [[终端代码智能体实践：基于 Aider 的 Repo Map 压缩与 Architect-Editor 双模型分工实战]], [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]] |

---

## 🚀 核心特性与技术亮点

1. **基于 Tree-sitter 与 PageRank 的全库代码地图（Repo Map）**：
   - 面对几十万行的大型项目，传统方案无法将全量源码丢进上下文窗口。Aider 利用 Tree-sitter 高效提取整个代码库的类、方法、函数签名定义与调用拓扑，通过图论 PageRank 算法计算标识符重要度；
   - 仅用 1k~2k tokens 的紧凑开销，即可让大模型清晰掌握跨模块引用的全景上下文，精准定位修改点。
2. **Git 原生深度集成与自动化原子提交（Auto-Commit）**：
   - Aider 会在每次代码生成前检查 Git 脏工作区，并在变更生效且测试通过后自动生成符合 Conventional Commits 规范的 Git 提交日志；
   - 支持 `/undo` 一键撤销最后一次提交，让 AI 结对编程具备随时回滚的安全沙箱感。
3. **架构师与编辑者双模型协作范式（Architect Mode）**：
   - 解决单模型在复杂重构任务中“思考规划”与“生成精准补丁”不可兼得的问题；
   - 采用大参数顶级推理模型（如 Claude 3.5 Sonnet / o1 / DeepSeek-R1）担任 **Architect** 负责逻辑推理与修改方案设计；
   - 调度低成本高代码精度的快速模型（如 DeepSeek-V3 / Haiku）担任 **Editor**，严格按 Unified Diff 格式高效打补丁。
4. **多样化 Edit Format 容错引擎**：
   - 支持 `diff`、`udiff`、`whole`、`ask` 等多种编辑格式，并在模型输出格式偏离时具备自适应重试与提示词自动修复能力，确保长文件修改不会截断代码。

---

## 🛠️ 快速上手与集成

### 1. 安装 Aider

```bash
# 通过 pipx 或 uv 快速安装
pipx install aider-chat
# 或直接通过 pip
pip install aider-chat
```

### 2. 启动与常用工作流

```bash
# 在现有 Git 仓库根目录启动，配置大模型 API Key
export ANTHROPIC_API_KEY="sk-..."
export OPENAI_API_KEY="sk-..."

# 1. 以经典 Architect 双模型模式启动
aider --model claude-3-5-sonnet-20241022 --editor-model deepseek/deepseek-chat

# 2. 将相关关注文件加入上下文中
# 在交互命令行中：
/add src/services/user_service.py src/models/user.py

# 3. 输入自然语言需求提示
> 请给 UserService 增加根据邮箱模糊查询的方法，并在 models 中增加对应索引定义，同时编写 pytest 单元测试
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 日常终端开发者的生产力辅助：解决“在 IDE 各种 Copilot 窗口间反复复制粘贴、手工核对 git diff”的繁琐流程；
  - 复杂代码库跨文件功能迁移、依赖版本升级、批量编写单元测试与文档补充；
  - 配合 CI 脚本或 PR 机器人在命令行自动化执行确定性的重构任务。
- **潜在不足 / 局限性**：
  - 对 Git 强依赖，若在未初始化的非 Git 目录下无法启用版本跟踪与回滚优势；
  - Repo Map 依赖 Tree-sitter 语法解析器，冷门编程语言或非标准宏语法的提取质量会有所折损。
- **评估结论**：**极力推荐引入（Adopted）**。目前业界纯命令行场景下代码质量与可控性最高的 AI 结对编程框架之一。
