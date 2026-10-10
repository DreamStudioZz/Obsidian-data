---
tags:
  - ai-tool
  - open-source
  - agent-tools
  - oauth
  - mcp
  - status/adopted
category: Agents
github: "https://github.com/ComposioHQ/composio"
stars: "15k+"
license: "Apache-2.0"
date_added: 2026-10-10
---

# 📦 Composio

> **一句话简介**：生产级 Agent 高可靠外部工具与 Action 执行基础设施，提供 1000+ 企业 SaaS 工具链集成、全托管 OAuth/JWT 认证、原生 MCP 桥接与沙箱化执行能力。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - ComposioHQ/composio](https://github.com/ComposioHQ/composio) |
| **技术栈** | Python, TypeScript, REST API, MCP (Model Context Protocol) |
| **许可证** | Apache-2.0 |
| **相关关联** | 关联专题: [[FastMCP]], [[PydanticAI]], [[LangGraph]], [[CrewAI]], [[E2B]] |

---

## 🚀 核心特性与亮点

1. **企业级托管认证中枢（Managed Authentication & OAuth Proxy）**：
   - 彻底解决 Agent 调用外部服务（如 GitHub、Slack、Jira、Salesforce、Google Workspace、Linear）时的鉴权噩梦；
   - 自动处理多租户用户授权重定向、OAuth 2.0 握手、Token 加密存储、无感 Refresh Token 自动刷新与权限撤销。
2. **海量生产就绪工具库（1,000+ Pre-built Toolkits）**：
   - 覆盖代码托管、项目管理、办公协同、CRM、数据分析等主流 SaaS；
   - 每个 Action 都经过严格的 JSON Schema 与 Pydantic 类型定义，带字段校验与输入自动纠错，大幅降低大模型参数幻觉。
3. **原生 Model Context Protocol (MCP) 与框架泛化兼容**：
   - 支持将所有 Composio Actions 一键转换为 MCP Server，无缝供给 Claude Desktop、Cline、Cursor 等 MCP Client；
   - 深度集成主流编排框架：OpenAI Assistants、LangGraph、CrewAI、PydanticAI、Autogen、LlamaIndex 等。
4. **沙箱化隔离执行与重试容错（Sandboxed & Resilient Execution）**：
   - 支持本地环境、Docker 以及云端 E2B 沙箱执行，隔绝高危命令对宿主系统的威胁；
   - 内置智能工具检索（Tool RAG）、API 速率限制（Rate Limiting）拦截、幂等重试与结构化异常捕获。
5. **与同类工具对比**：
   - **相比于 LangChain Community Tools**：LangChain 原生工具多为 Demo 级封装，缺乏真实多租户 OAuth 状态流转和 Token 刷新机制，生产落地维护成本高；
   - **相比于原生自定义 Function Calling**：Composio 提供了完整的鉴权鉴权代理、鉴权回调 Webhook 与审计日志，大幅减少企业重构胶水代码的投入。

---

## 🛠️ 快速上手与配置

```bash
# 1. 安装核心 SDK（以 Python 为例）
pip install composio-core composio-openai

# 2. 命令行快速登录与配置
composio login

# 3. 为当前环境接入外部应用（例如 GitHub）
composio add github
```

```python
from openai import OpenAI
from composio_openai import ComposioToolSet, Action, App

# 1. 初始化客户端与 Composio 工具集
client = OpenAI()
toolset = ComposioToolSet(entity_id="user_enterprise_1024")

# 2. 获取预授权的 GitHub 工具动作（带托管 OAuth）
# 例如：获取 Star 仓库、创建 Issue 的受管工具定义
tools = toolset.get_tools(apps=[App.GITHUB])

# 3. 发送给大模型进行自主决策执行
prompt = "请帮我检查仓库 ComposioHQ/composio 的最新 Star 数量，并在该仓库提一个测试 Feedback Issue。"

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": prompt}],
    tools=tools,
    tool_choice="auto",
)

# 4. 执行大模型返回的 Tool Call（Composio 自动注入用户 OAuth Token 并执行 API）
result = toolset.handle_tool_calls(response)
print("Action 执行结果：", result)
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **企业级 Copilot 与自动化助理**：需要帮员工真实操作飞书、钉钉、Slack、GitHub、Jira、Gmail 等内外部系统的业务 Agent；
  - **多租户 SaaS 平台**：每个终端用户需要绑定各自私有云账号，必须在多租户维度实现 OAuth Token 安全隔离与生命周期管理；
  - **MCP 协议统一网关**：希望将几十种复杂 SaaS API 快速聚合为标准化 MCP 服务的工程架构。
- **潜在不足 / 局限性**：
  - 深度依赖云端托管平台（Composio Cloud）的 OAuth 会话中继，如果企业要求 100% 内网离线物理隔离，需要部署其开源自托管组件并自行配置各 SaaS 的 App Client ID/Secret；
  - 部分小众垂直 SaaS 工具更新滞后时，仍需开发者通过 Composio Custom Action 进行补丁定义。
- **个人评估结论**：**建议引入**。是当前将 AI Agent 从“只会聊天的玩具 Demo”推向“具备真实业务系统执行力”的生产级核心工具基础设施。
