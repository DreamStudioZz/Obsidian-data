---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/All-Hands-AI/OpenHands
stars: "45k+"
license: MIT
date_added: 2026-09-23
---

# 📦 OpenHands

> **一句话简介**：开源软件工程智能体（AI Software Engineer）标杆平台（前身为 OpenDevin），通过 Docker 隔离沙箱、终端/浏览器/代码编辑器多模态工具链与微智能体（Micro-Agents）架构，实现从需求分析、代码编写、单测运行到 PR 提交的全自动闭环。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands) |
| **项目曾用名** | OpenDevin |
| **技术栈** | Python, FastAPI, Docker, React / TypeScript |
| **评测基准** | SWE-bench Verified 领先级开源表现 |
| **关联实践** | [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[Pi-Agent]], [[smolagents]] |

---

## 🚀 核心特性与架构亮点

1. **安全隔离沙箱（Secure Execution Sandbox）**：
   - 彻底解决 Coding Agent 执行危险命令（如 `rm -rf`、端口占用、依赖污染）的安全隐患。
   - 所有终端命令、文件读写与环境安装均在独立的 Docker 容器内执行，支持挂载用户本地宿主机代码工作区。
2. **多模态环境交互能力（Multi-Modal Tooling）**：
   - **Terminal Tool**：直接交互 bash 命令行，支持实时进程捕获、长时间运行命令监控与自动退出码检测；
   - **File Editor Tool**：提供精准行定位、局部字符串替换、语法检查与撤销（Rollback）机制；
   - **Browser Tool**：集成无头浏览器，可自主浏览官方文档、查看本地 Dev Server 网页渲染与调试控制台报错。
3. **微智能体协同生态（Micro-Agents & AgentHub）**：
   - 将复杂软件工程任务拆解为针对特定任务域优化的专门 Agent（如 CodeWritingAgent、RepoAnalysisAgent、GitWorkflowAgent），通过统一协议交换状态。
4. **人机协作控制台（Human-in-the-Loop Web GUI）**：
   - 提供可视化 Web 界面，实时流式展示 Agent 的思考过程、终端输出、当前编辑文件 Diff 以及交互式断点干预。

---

## 🛠️ 快速上手与部署

### 1. 基于 Docker 一键启动 OpenHands

```bash
# 启动 OpenHands 容器并在宿主机挂载代码目录
WORKSPACE_BASE=$(pwd)/workspace
docker run -it --pull=always \
    -e SANDBOX_USER_ID=$(id -u) \
    -e LLM_API_KEY="your-llm-api-key" \
    -e LLM_MODEL="claude-3-5-sonnet-20241022" \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $WORKSPACE_BASE:/opt/workspace_base \
    -p 3000:3000 \
    --name openhands-app \
    ghcr.io/all-hands-ai/openhands:latest
```

### 2. 通过 Python SDK 编程式调用

```python
from openhands.core.main import run_controller
from openhands.controller.agent import Agent
from openhands.events.action import MessageAction

# 配置 Agent 与执行任务
task_prompt = "请克隆仓库，定位 issue #102 的空指针异常，编写复现单测，修复 Bug 并确保所有 pytest 通过。"
# 驱动 Agent 在沙箱中自主排查并提交代码
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **GitHub Issue 自动修复与 PR 生成**：接入 CI/CD 流水线，对有明确复现步骤的 bug 自动定位并提 PR；
  - **遗留代码库重构与单测补全**：自动扫描测试覆盖率低的核心模块并生成高覆盖率单元测试；
  - **端到端自动化原型搭建**：从零初始化项目骨架、配置脚手架并安装对应依赖。
- **与同类工具对比**：
  - 相比于仅做代码补全的 Cursor / Copilot：OpenHands 是**全自主执行型智能体**，具备自建环境、自跑测试、自我纠错的完整生命周期；
  - 相比于轻量终端脚本 Pi-Agent / Aider：OpenHands 提供了**工业级 Docker 沙箱强隔离**与**完整 Web 协同工作台**，更适合复杂大型项目与团队集成。
- **综合评估结论**：生产级开源自主编程智能体首选，强烈推荐采纳（Adopted）。
