---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/openclaw/openclaw
stars: "395k+"
license: MIT
date_added: 2026-11-19
---

# 📦 OpenClaw

> **一句话简介**：Peter Steinberger 团队开源的现象级自托管个人自主智能体网关（Autonomous Agent Gateway & Runtime）。彻底颠覆传统聊天网页套壳与封闭式云端 Agent 服务，以长驻守护进程（Daemon Gateway）统一调度接入 50+ 主流即时通讯协议（WhatsApp、Telegram、Slack、Discord、Signal、iMessage 等）；创新采用强类型 WebSocket 事件总线、跨设备本地外设调用协议（Node Protocol）、完全基于文件的透明工作区人格定义（File-Backed Workspace），以及后台异步梦境整合记忆引擎（Dreaming & Consolidation Memory），打造兼具绝对数据主权与物理世界执行能力的个人超级 AI 助手底座。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: openclaw/openclaw](https://github.com/openclaw/openclaw) |
| **研发核心** | Peter Steinberger (@steipete) 及全球开源贡献者社区 |
| **技术栈** | TypeScript / Node.js, Swift, Go, WebSockets, SQLite (FTS5 + Vector), JSON Schema, Baileys, grammY, discord.js, Model Context Protocol (MCP) |
| **许可证** | MIT License |
| **核心机制** | 统一守护网关（Central Long-lived Gateway）、强类型双向 WebSocket 消息总线（`agent`, `chat`, `presence`, `health`, `cron`）、设备节点执行协议（Node Protocol 穿透 macOS/Linux/Android 系统外设）、文件驱动工作区人格（File-backed Persona & Workspace）、分层持久化异步记忆（Tiered Memory with Dreaming Loop）、多模型与 MCP 插件编排 |
| **部署形态** | **100% 本地自托管**：单台家庭 NAS、Mac mini 或廉价 VPS 上单进程常驻运行，拒绝任何中心化云服务数据窃取；**跨设备节点编排**：桌面电脑、手机、边缘网关作为受控 Node 安全挂载接入 |
| **相关关联** | 关联实践: [[个人自主智能体网关架构：基于 OpenClaw 的多通道 IM 接入、WebSocket 事件网关与分层文件记忆实战]], [[2026-11-19-AI-Digest]], [[Aider]], [[Cline]], [[Open-Interpreter]], [[Browser-Use]], [[FastMCP]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph MessagingChannels [多源即时通讯渠道 (Inbound Channels)]
        WA["WhatsApp (Baileys)"]
        TG["Telegram (grammY)"]
        DC["Discord (discord.js)"]
        SL["Slack (Bolt)"]
        SG["Signal / iMessage"]
        WebUI["Web / Desktop Dashboard"]
    end

    subgraph CentralGateway [OpenClaw 常驻网关内核 (ws://127.0.0.1:18789)]
        ChannelAdapter["多渠道协议适配器 (Channel Adapters)"]
        SessionRouter["会话路由与上下文预算器 (Session & Context Budgeter)"]
        EventBus["强类型 JSON Schema WebSocket 调度总线"]
        
        subgraph MemoryCore [分层记忆与反思引擎]
            WorkspaceFiles["工作区持久文件<br/>(AGENTS.md / RULES.md / IDENTITY.md)"]
            SQLiteIndex["SQLite 语义与 FTS5 倒排索引"]
            DreamingLoop["后台异步梦境整合 (Async Dreaming Loop)"]
        end
    end

    subgraph ExecutionPlane [分布式设备执行节点 (Nodes Protocol)]
        MacNode["macOS 桌面 Node<br/>(camera, screen, shell)"]
        LinuxServer["Linux 生产服务器 Node<br/>(docker, git, k8s)"]
        MobileNode["Android/iOS 移动端 Node<br/>(location, notify)"]
    end

    subgraph IntelligencePlane [多模型推理与工具总线]
        MCPClient["Model Context Protocol (MCP) 客户端"]
        LLMProviders["Claude 3.7 / DeepSeek-V3 / 本地 Ollama"]
    end

    MessagingChannels <--> ChannelAdapter
    ChannelAdapter --> SessionRouter
    SessionRouter <--> EventBus
    EventBus <--> MemoryCore
    EventBus <--> ExecutionPlane
    EventBus <--> IntelligencePlane
```

### 1. 集中式常驻守护网关（The Long-Lived Daemon Gateway）
传统智能体工具要么是临时执行的命令行脚本（跑完即退），要么是脆弱的纯前端应用（标签页关闭即中断会话）。OpenClaw 设计为单进程长期驻留网关：
- **全局连接持久化**：统一维护与 WhatsApp（基于 Baileys）、Telegram（基于 grammY）、Discord、Slack、Signal 的双向长连接，网络瞬断具备断线重连与消息排队机制；
- **标准化事件分发**：所有渠道的进出消息均被解构为标准的强类型事件（`agent`, `chat`, `presence`, `health`, `cron`），统一经由内部 WebSocket 转发；
- **定时与主动触发器（Cron Trigger）**：智能体不再单纯被动等待用户输入，可依托内置 Cron 引擎主动拉取日程、监控告警、定期复盘待办事项并向用户主流聊天窗口发起主动对话。

### 2. 节点协议与物理硬件能力穿透（Node Protocol）
OpenClaw 创新性地引入了客户端与执行节点的松耦合架构：
- 任何安装了轻量客户端的设备（如开发者的 M4 Mac mini、工作站 Linux、乃至随身携带的智能手机）均可通过带鉴权的 WebSocket 握手注册为 `role: node`；
- Gateway 将本地能力封装为安全的 RPC 指令：`screen.record`（截屏分析）、`camera.snap`（外接摄像头视觉巡检）、`shell.exec`（沙箱终端命令运行）、`location.get`（基于物理位置的任务触发）；
- **细粒度权限安全门禁**：每个 Node 均可本地声明可暴露的方法白名单与敏感指令交互式人工确认机制，确保 AI 即使受到 Prompt 注入攻击也无法突破本地隔离边界。

### 3. 基于纯文本工作区的人格与规则驱动（File-Backed Workspace）
摒弃了传统框架将 Agent 系统提示词（System Prompt）和用户偏好硬编码在数据库或内存变量中的“黑盒”模式：
- **可版本化的配置体系**：Agent 的角色定位、对话语气、核心守则、长期偏好全部保存在工作区的标准 Markdown 文件中（`IDENTITY.md`、`AGENTS.md`、`RULES.md`、`PREFERENCES.md`）；
- **像改代码一样调教 Agent**：用户可以直接使用任何代码编辑器（VS Code、Obsidian）打开工作区随时微调规则，每次 Git 提交即自动完成智能体演进回溯；
- **动态分发与子 Agent 矩阵**：支持在工作区声明多角色协作树（如 Researcher 深度调研、Architect 架构方案、Coder 代码实现、Validator 质量审查），任务自动路由。

### 4. 分层记忆与异步“梦境整合”引擎（Tiered Memory with Dreaming Loop）
长链个人助手最怕长期对话导致的上下文膨胀、Token 爆表与记忆混乱。OpenClaw 创造了仿生分层记忆机制：
- **短期工作窗口（Working Memory）**：严密计算当前对话 Token 预算，仅挂载最近活跃回合及强制规则；
- **中期语义索引（Episodic & FTS5 Index）**：会话沉淀进本地 SQLite，集成 SQLite-Vector 与全文搜索，按语义余弦相似度与关键词混合召回；
- **后台异步梦境整合（Dreaming Loop）**：在系统空闲或夜间定时调度期间，启动后台模型将零碎日常日志精简、提炼、去重，并生成结构化的长效实体知识更新回工作区核心 Markdown 文件中，彻底摆脱前台请求时的巨大计算开销。

---

## 🛠️ 快速上手与配置

### 1. 全局安装与初始化常驻网关

```bash
# 全局安装 OpenClaw CLI 与网关内核
npm install -g openclaw@latest

# 初始化工作区（生成工作区规则模板与 SQLite 存储目录）
openclaw init ~/openclaw-workspace

# 启动常驻网关后台服务
openclaw gateway start --port 18789 --daemon
```

### 2. 核心工作区规则配置示例 (`~/openclaw-workspace/AGENTS.md`)

```markdown
# 🤖 Agent Operational Rules & Roles

## Core Identity
- Name: Jarvis
- Primary Persona: 极其干练、严谨的高级系统架构师兼个人参谋，回答简明直接，崇尚工程确定性。

## Channel Behaviors
- **Telegram / WhatsApp**: 移动端响应，回复精简在 3 句话以内，重点呈现结论与行动项。
- **Slack / Desktop**: 工作场景，详细提供架构分析、代码示例与终端操作日志。

## Safety Policies
- 严禁未经二次确认执行 `rm -rf`、`git push --force` 或修改系统根权限文件。
- 涉及外部金融转账、邮件群发等操作，必须生成确认链接挂起等待。
```

### 3. 多渠道接入配置 (`config.yaml`)

```yaml
gateway:
  host: 127.0.0.1
  port: 18789
  auth_token: "sk-claw-sec-prod-token-98437"

channels:
  telegram:
    enabled: true
    bot_token: "BOT_TOKEN_FROM_BOTFATHER"
    allowed_chat_ids: [123456789]
  whatsapp:
    enabled: true
    session_path: "~/.openclaw/sessions/whatsapp"
  slack:
    enabled: true
    app_token: "xapp-..."
    bot_token: "xoxb-..."

models:
  primary:
    provider: deepseek
    model: deepseek-chat
    base_url: https://api.deepseek.com/v1
    api_key: env(DEEPSEEK_API_KEY)
  reasoning:
    provider: anthropic
    model: claude-3-7-sonnet-20250219
    api_key: env(ANTHROPIC_API_KEY)

memory:
  sqlite_path: "~/openclaw-workspace/memory.sqlite"
  dreaming_interval: "0 3 * * *" # 每天凌晨 3 点执行梦境记忆提炼
```

### 4. 挂载本地执行 Node (以 macOS 为例)

```bash
# 在宿主开发机上启动受控执行节点并连接到本地网关
openclaw node connect \
  --gateway ws://127.0.0.1:18789 \
  --token sk-claw-sec-prod-token-98437 \
  --capabilities "camera,screen,shell" \
  --whitelist-commands "git,pnpm,docker,pytest,tree"
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **跨设备终极个人 AI 副驾**：无论是出门在外通过手机 WhatsApp/Telegram 查阅资料，还是在工位让 AI 操控桌面终端跑编译排障，会话记忆与工作区状态全局无缝同步；
  2. **企业研发/运维告警联动智能体**：长期监听内部 Slack 频道，配合 Cron 轮询 Kubernetes 或云监控事件，一旦出现异常自动调度诊断工具排查，并给出修复 PR；
  3. **数据敏感型本地工作流自动化**：所有会话数据、知识索引与核心规则完全沉淀在本地磁盘，不依赖第三方闭源 Agent 云平台。
- **潜在不足 / 局限性**：
  1. **多 IM 协议反爬与风控挑战**：WhatsApp（基于逆向 Web 的 Baileys 协议）长时间高频自动化操作有被 Meta 风控标记的风险，推荐搭配企业官方 WhatsApp Business API 或 Telegram/Slack 官方 Bot 接口；
  2. **常驻进程的资源开销**：当挂载海量渠道并持续运行本地向量检索时，网关长时间驻留需合理配置 Node.js 内存上限；
  3. **Node 节点端到端鉴权必须严密配置**：若暴露到公网，必须配置严格的 TLS 与 Token 双向绑定，避免终端执行能力被未授权入侵。
- **个人评估结论**：**极度推荐采用（Adopt）**。OpenClaw 代表了个人 AI 智能体从“网页对话框玩具”迈向“全功能个人操作系统（Personal Operating System）”的转折点，其解耦的架构设计、基于文件的透明状态管理与多端执行总线是目前开源界最高工程完成度的范例。