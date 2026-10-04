---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/cline/cline
stars: "45k+"
license: Apache-2.0
date_added: 2026-09-30
---

# 📦 Cline

> **一句话简介**：深度集成在 VS Code / Cursor 等 IDE 中的开源自主编程智能体，原生支持 MCP（Model Context Protocol）协议扩展生态，具备文件差异安全审批、终端自主执行与上下文精准分阶管理能力。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: cline/cline](https://github.com/cline/cline) |
| **核心特点** | IDE 原生集成、MCP 协议首选客户端、Plan/Act 双模式分步执行、逐行 Diff 视觉审批、上下文动态压缩 |
| **技术栈** | TypeScript / VS Code Extension API / Model Context Protocol (MCP) |
| **关联实践** | [[IDE 智能编程范式：基于 Cline 与 MCP 协议的上下文工程与自主开发闭环实战]], [[终端代码智能体实践：基于 Aider 的 Repo Map 压缩与 Architect-Editor 双模型分工实战]], [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]] |

---

## 🚀 核心特性与技术亮点

1. **原生 MCP（Model Context Protocol）扩展生态**：
   - 业内首个全功能拥抱 Anthropic MCP 协议的 IDE 智能体，支持通过简单的 JSON 配置热插拔任意标准 MCP 服务器（如 GitHub、PostgreSQL、Puppeteer、Sentry、Brave Search 等）；
   - 智能体可动态发现、加载与调用外部专业工具与资源，实现从代码编写到数据库查询、线上报错排查的全链路自治。
2. **Plan & Act 分阶双模式与人在回路（Human-in-the-Loop）**：
   - **Plan 模式**：只读分析代码库结构，输出详尽的架构重构方案、改动清单与潜在风险，不擅自改动磁盘；
   - **Act 模式**：严格按照规划方案逐步调用工具、打补丁、执行命令，并在关键敏感动作（如终端高危命令、大面积文件写入）前向开发者请求显式确认。
3. **逐行差异审批与安全沙箱控制**：
   - 每次代码修改均在 VS Code 内置的多标签差异视图（Diff Editor）中清晰高亮展示，开发者可单行审查、接受或拒绝；
   - 支持配置指令白名单与只读目录沙箱，防止代码智能体执行具有破坏性的脚本或误删系统关键文件。
4. **长任务执行上下文弹性管理与动态窗口滑移**：
   - 内置智能 Token 追踪与上下文压缩机制，长任务执行时自动对历史工具调用输出进行关键信息摘录与归纳，避免注意力稀释与 API 费用急剧膨胀。

---

## 🛠️ 快速上手与配置

### 1. 安装与环境变量
在 VS Code 插件市场搜索并安装 `Cline`（前身为 Claude Dev）。

### 2. 配置 MCP 服务器（`cline_mcp_settings.json`）
通过 IDE 设置面板打开或编辑 `~/.vscode/cline_mcp_settings.json`：

```json
{
  "mcpServers": {
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "."]
    },
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

### 3. 典型交互闭环
1. 切换至 **Plan Mode**，提问：
   ```text
   我们需要在现有的 FastAPI 工程中引入 Redis 分布式限流，请先调研项目结构并给出实施方案。
   ```
2. 审阅架构方案后，切换至 **Act Mode**：
   ```text
   方案可行，请开始按步骤创建限流中间件，并完善单元测试与 docker-compose 配置。
   ```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 深度依赖 IDE 界面开发、重视代码修改过程可视化与逐行安全核验的研发团队；
  - 需要打通企业内部系统（如内部 Jira、数据库、私有 API）的定制化开发助手，通过自建 MCP 服务实现低成本赋能；
  - 复杂全栈功能的端到端开发：从建表、写接口、编写前端页面到浏览器端自动化冒烟测试。
- **潜在不足 / 局限性**：
  - 依赖 VS Code 宿主环境，在纯远程无图形界面 Headless Linux 服务器上不如纯终端工具（如 Aider）轻便；
  - 长周期运行多轮工具调用时，若模型未能适时终止，可能造成 Token 消耗较快，需合理设置上下文预算。
- **评估结论**：**极力推荐引入（Adopted）**。当前开源 IDE 智能体中架构设计最规范、MCP 扩展性最强、落地实用性极高的标杆级工具。
