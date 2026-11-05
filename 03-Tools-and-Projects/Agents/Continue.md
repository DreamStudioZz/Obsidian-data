---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/continuedev/continue
stars: "23.5k+"
license: Apache-2.0
date_added: 2026-11-05
---

# 📦 Continue

> **一句话简介**：全球最领先的开源本地化 AI 结对编程与智能代码助理插件架构（The Leading Open-Source AI Code Assistant for VS Code & JetBrains）。深度支持任意主流本地与云端模型无缝接入，原生内置基于 AST 语法树与向量混合检索的 `@Codebase` 上下文工程索引，全面集成 Model Context Protocol (MCP) 开放工具生态，支持高响应度 FIM (Fill-In-The-Middle) 代码补全与企业私有化安全合规落地。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: continuedev/continue](https://github.com/continuedev/continue) |
| **研发机构** | Continue Dev 团队与全球开源社区 |
| **技术栈** | TypeScript, React, Python, SQLite, LanceDB, Tree-Sitter, Model Context Protocol (MCP) |
| **许可证** | Apache-2.0 License |
| **支持平台** | Visual Studio Code, JetBrains 全家桶 (IntelliJ IDEA, PyCharm, WebStorm 等), Headless CLI |
| **核心协议与机制** | Context Provider 协议、Model Context Protocol (MCP) 客户端、FIM (Fill-in-the-Middle) 自动补全、AST 代码分块与 LanceDB 嵌入索引 |
| **兼容模型后端** | 本地推理（Ollama, vLLM, LMDeploy, llama.cpp）、私有云网关（LiteLLM, Xinference）以及主流商用 API（DeepSeek, Anthropic Claude, OpenAI, Qwen） |
| **相关关联** | 关联实践: [[企业级私有化 AI 辅助编程架构：基于 Continue 的本地上下文编排、MCP 扩展与 FIM 补全实战]], [[2026-11-05-AI-Digest]], [[Cline]], [[Aider]], [[SWE-agent]], [[FastMCP]], [[LiteLLM]], [[LanceDB]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph IDE_Surface [IDE 宿主环境 (VS Code / JetBrains)]
        ChatPanel[侧边栏对话面板: Continue GUI]
        InlineEdit[行内编辑与重构: Ctrl+I / Cmd+I]
        AutocompleteEngine[Tab 自动补全引擎: FIM 极速预测]
    end

    subgraph ContinueCore [Continue 核心中间层 (Continue Core Process)]
        ContextEngine[上下文装配引擎: Context Providers]
        MCP_Client[Model Context Protocol 客户端]
        ConfigEngine[配置解析器: config.yaml / config.json]
    end

    subgraph ContextSources [智能上下文源 (Context System)]
        CodebaseProvider["@Codebase: Tree-Sitter AST + LanceDB 混合检索"]
        DocsProvider["@Docs: 在线/离线技术文档摄取爬取"]
        FolderProvider["@Folder / @File: 显式多文件引用"]
        TerminalProvider["@Terminal: 终端输出与报错捕获"]
    end

    subgraph ExtTools [外部扩展与协议 (MCP Ecosystem)]
        GitLabMCP[GitLab/GitHub MCP: Issue 与 PR 联动]
        DB_MCP[Database MCP: 元数据与 Schema 洞察]
        CustomMCP[企业内部私有服务 MCP]
    end

    subgraph LLM_Backends [模型推理后端 (Model Providers)]
        LocalOllama["本地 Ollama / vLLM (Qwen2.5-Coder / DeepSeek-Coder)"]
        PrivateGateway["企业私有网关 (LiteLLM / 自建反向代理)"]
        CommercialCloud["公有云端 (Claude 3.5 Sonnet / GPT-4o)"]
    end

    IDE_Surface <--> ContinueCore
    ContinueCore --> ContextSources
    ContinueCore <--> ExtTools
    ContinueCore --> LLM_Backends
```

### 1. 彻底打破厂商锁定的模型自由度与数据隐私安全
- 商业闭源方案（如 GitHub Copilot、Cursor）存在严重的数据安全合规隐患，金融、政企与大型工业制造企业严禁将核心业务代码上传至外部公有云；
- Continue 采用完全解耦的模型适配层，允许开发者在 `config.yaml` 中将不同任务指派给不同模型：
  - **聊天与长架构规划（Chat）**：使用云端 Claude 3.5 Sonnet 或企业内网部署的 DeepSeek-V2.5 / DeepSeek-Coder-33B；
  - **高频 Tab 自动补全（Autocomplete）**：直连本地单卡 Ollama 运行的 `Qwen2.5-Coder-1.5B/7B-Instruct`，毫秒级响应且零外发网络请求；
  - **嵌入检索（Embeddings）**：本地轻量级 `nomic-embed-text` 或 `bge-small-en-v1.5`，索引过程 100% 运行在本地内存与磁盘。

### 2. 强大的多维上下文感知系统与 `@Codebase` 混合索引
- 单文件代码编辑往往导致模型因缺乏全局上下文而产生“幻觉引用”或不兼容重构；
- Continue 引入标准 **Context Provider** 规范，原生支持：
  - **`@Codebase`**：利用 Tree-Sitter 对整个代码仓库进行语法抽象解析（AST 分块），生成函数与类级别的代码语义块，结合 LanceDB 本地列式向量检索与 BM25 关键词匹配，自动检索相关依赖注入当前 Prompt；
  - **`@Docs`**：一键预爬取第三方库或内部框架官方文档（如 FastAPI, PyTorch, Internal SDK），使模型具备最新 API 上下文；
  - **`@Terminal` / `@Problems`**：自动抓取终端运行报错堆栈与 IDE 编译错误，实现“一键定位修复”。

### 3. 原生支持 Model Context Protocol (MCP) 开放生态
- Continue 作为标准 **MCP 客户端**，能够直接连接任何符合 MCP 协议的本地或远程工具服务；
- 开发者可以通过简单的 JSON/YAML 配置挂载各种 MCP Server（如读取企业私有 PostgreSQL 数据库表结构、检索内部 GitLab Merge Request、操作 Docker 容器等），使得 IDE 内的 AI 助手直接蜕变为具备全系统操作能力的本地 Agent。

### 4. 极致调优的 FIM (Fill-in-the-Middle) 自动补全架构
- 补全不仅要求低延迟（< 250ms），更要求模型理解光标前后的代码断点；
- Continue 内置工业级 FIM 预处理器，精确计算 Prefix（前文）与 Suffix（后文）的 Token 预算，自适应适配 DeepSeek、Qwen、StarCoder 等主流模型的特定特殊字符（如 `<|fim_prefix|>`, `<|fim_suffix|>`, `<|fim_middle|>`），辅以前缀缓存与流式反抖动（Debouncing），提供媲美原生 Copilot 的丝滑输入体验。

---

## 🛠️ 快速上手与配置

### 1. 插件安装
- **VS Code**：在扩展商店搜索 `Continue` 点击安装（插件 ID: `Continue.continue`）；
- **JetBrains**：在 Settings -> Plugins -> Marketplace 搜索 `Continue` 安装重启。

### 2. 企业级混合部署核心配置文件 (`~/.continue/config.yaml`)

```yaml
name: Enterprise AI Assistant
version: 0.1.0

# 1. 交互式聊天与复杂任务规划模型 (连接企业内网私有网关或高性能本地服务)
models:
  - name: DeepSeek-Coder-V2.5 (Intranet)
    provider: openai
    model: deepseek-ai/DeepSeek-Coder-V2.5
    apiBase: http://ai-gateway.internal.corp/v1
    apiKey: corp-sec-token-xxx
    roles:
      - chat
      - edit
    contextLength: 65536

  - name: Local Qwen2.5-Coder 7B (Ollama)
    provider: ollama
    model: qwen2.5-coder:7b
    roles:
      - chat
      - edit

# 2. 极速 Tab 自动补全模型 (专为 50-150ms 超低延迟设计)
tabAutocompleteModel:
  title: Local Qwen2.5-Coder 1.5B (FIM)
  provider: ollama
  model: qwen2.5-coder:1.5b-base
  roles:
    - autocomplete

tabAutocompleteOptions:
  debounceDelay: 150 # 防抖延迟 (毫秒)
  maxPromptTokens: 1024 # 上下文 Token 限制
  multilineCompletions: always # 开启多行语义智能续写

# 3. 本地嵌入与代码仓库向量库配置
embeddingsProvider:
  provider: ollama
  model: nomic-embed-text

# 4. 上下文扩展与企业 MCP 服务挂载
context:
  - name: codebase
    params: {}
  - name: docs
    params: {}
  - name: terminal
    params: {}

# 5. 集成 Model Context Protocol (MCP) 扩展服务器
mcpServers:
  - name: internal-database-meta
    command: python
    args:
      - -m
      - enterprise_mcp.db_inspector
    env:
      DB_HOST: "db-replica.internal.corp"
      DB_PORT: "5432"

# 6. 自定义快捷 Slash Commands
slashCommands:
  - name: review
    description: 针对当前选中代码执行企业代码规范审查与性能分析
    prompt: |
      请作为资深架构师，严格审查以下代码：
      1. 是否存在内存泄露或未释放连接？
      2. 异常捕获是否过于泛化？
      3. 命名与类型契约是否严谨？
      {{{ input }}}
```

---

## 💡 工程实战点评与落地选型建议

- **生产选型定位**：
  - 若团队对**知识产权与代码保密**有严格红线，且期望打造**全功能统一 IDE 交互体验**，Continue 是目前全球生态最繁荣、更新最积极的绝对首选；
  - 相较于 **Cline** 强调“自主修改多文件、端到端跑测试”的重度自治执行模式，Continue 更偏向于“伴随式编程、实时 Tab 补全、交互式重构与即时咨询”，二者在开发生命周期中形成极佳的互补。
- **硬件与部署门槛**：
  - **开发者工作站（端侧）**：建议配备 Apple Silicon (M1/M2/M3 Pro 16GB+) 或配备 RTX 4060 (8GB VRAM) 以上的 PC，即可在本地流畅运行 `Qwen2.5-Coder-1.5B` 进行毫秒级 FIM 补全；
  - **企业内网集中服务（服务端）**：建议在 GPU 计算节点集群使用 `vLLM` 部署 32B/70B 级别的 Coder 模型，并搭配 `LiteLLM` 进行鉴权、限流与审计日志留存，客户端统一通过 Continue 连接。
- **最佳实践建议**：
  - 在大型代码工程中，务必在仓库根目录维护 `.continueignore`，过滤大型构建中间产物（如 `dist/`, `target/`, `node_modules/`），避免 LanceDB 索引时耗尽本地 CPU 与内存资源。

---

## 🔗 关联卡片与生态导航
- 关联实战：[[企业级私有化 AI 辅助编程架构：基于 Continue 的本地上下文编排、MCP 扩展与 FIM 补全实战]]
- 关联工具：[[Cline]]、[[Aider]]、[[SWE-agent]]、[[FastMCP]]、[[LiteLLM]]、[[LanceDB]]
- 今日日报：[[2026-11-05-AI-Digest]]
