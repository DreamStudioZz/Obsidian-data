---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/open-interpreter/open-interpreter
stars: "56.5k+"
license: AGPL-3.0
date_added: 2026-11-06
---

# 📦 Open-Interpreter

> **一句话简介**：全球最具代表性的开源本地系统级代码解释与全自主操作系统控制智能体（Open-Source Local Code Interpreter & Computer-Use Agent）。打破传统云端沙箱隔离限制，允许大语言模型在本地终端安全受控地编写并执行 Python、Bash、PowerShell、AppleScript 等多语言代码，深度集成纯视觉 GUI 屏幕感知与键鼠自动化，并原生支持本地模型（Ollama / vLLM）与 Docker 容器级安全沙箱隔离。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: open-interpreter/open-interpreter](https://github.com/open-interpreter/open-interpreter) |
| **研发团队** | Killian Lucas 与 Open Interpreter 全球开源社区 |
| **技术栈** | Python, LiteLLM, Docker, PyAutoGUI, Tree-Sitter, Jupyter Kernel Protocol |
| **许可证** | AGPL-3.0 License |
| **支持平台** | macOS, Linux, Windows, Docker 容器环境, 终端 CLI, 嵌入式 Python 库 |
| **核心执行模式** | Local CLI (本地直接执行)、Docker Sandbox (容器隔离模式)、Computer-Use (屏幕视觉驱动与键鼠接管)、Server/Client (分布式远程宿主调度) |
| **兼容模型后端** | 本地推理（Ollama, vLLM, LMDeploy, llama.cpp）、企业私有反向代理网关（LiteLLM）以及公有云主流商用模型（Claude 3.5 Sonnet, GPT-4o, DeepSeek-V2.5） |
| **相关关联** | 关联实践: [[本地系统级自主执行智能体架构：基于 Open-Interpreter 的沙箱安全隔离、跨平台 OS 控制与本地大模型闭环实战]], [[2026-11-06-AI-Digest]], [[Aider]], [[Cline]], [[SWE-agent]], [[OpenHands]], [[E2B]], [[OpenShell]], [[Ollama]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph UserInterface [交互入口 (Interaction Layer)]
        CLI_Terminal[交互式终端: interpreter CLI]
        Python_API[嵌入式 Python SDK: interpreter.chat]
        Vision_GUI[Computer-Use: 纯视觉屏幕与键鼠控制器]
    end

    subgraph CoreEngine [Open Interpreter 核心调度引擎]
        PlanningLoop[ReAct 自主循环与多轮状态追踪]
        CodeExtractor[代码块解析与语法流式提取器]
        SafetyGuard[安全门禁与确认机制: safe_mode & auto_run]
    end

    subgraph ExecutionSubsystems [语言执行子系统 (Execution Runtimes)]
        PythonRuntime[Python Subprocess / Jupyter 状态内核]
        BashRuntime[Bash / Zsh 交互式持久子进程]
        PowerShellRuntime[Windows PowerShell 执行环境]
        AppleScriptRuntime[macOS AppleScript / JXA 脚本引擎]
        BrowserRuntime[浏览器自动化与网络请求通道]
    end

    subgraph IsolationLayer [安全沙箱与隔离策略 (Security Sandboxing)]
        HostDirect[直接宿主执行: 用户按步交互确认]
        DockerSandbox[Docker 容器隔离: 虚拟挂载与资源熔断]
        NetworkFilter[出站流量代理与敏感文件只读挂载]
    end

    subgraph ModelLayer [模型路由与推理 (Model Inference Layer)]
        LocalOllama[本地轻量模型: Ollama / vLLM / llama.cpp]
        RemoteAPI[云端推理引擎: Claude 3.5 Sonnet / DeepSeek]
    end

    UserInterface <--> CoreEngine
    CoreEngine --> SafetyGuard
    SafetyGuard --> IsolationLayer
    IsolationLayer --> ExecutionSubsystems
    CoreEngine <--> ModelLayer
```

### 1. 突破云端围墙：本地原生全能力系统级执行
- 商业闭源方案（如 OpenAI Code Interpreter / Advanced Data Analysis）被困在云端短暂、隔离、无外网访问且无法操作本地文件的受限沙箱中；
- Open Interpreter 直接在用户本地环境或私有服务器运行，赋予大模型操作本地海量文件、调用本级 GPU/CPU 计算、处理数百 GB 本地数据集、编辑视频媒体以及控制系统级软件的完整物理能力。

### 2. 多语言多进程交互式状态持久化（Stateful Execution）
- 模型在单次会话中执行的代码并非一次性脚本丢弃，而是依托底层的持久化子进程机制保持上下文：
  - **Python 运行时**：维护全局变量字典、已导入包与中间计算结果，模型可在第一轮加载 10GB 数据集，在第十轮继续对该 DataFrame 进行绘图或统计推断；
  - **Shell 运行时**：持久保持环境变量（`export PATH=...`）与当前工作目录（`cd /target/dir`），实现无缝连贯的系统运维与自动化编译。

### 3. 多层次纵深安全防御机制（Safety & Sandboxing）
- 针对模型自主执行 Shell/Python 可能带来的恶意越权或灾难性误删（如 `rm -rf /` 或覆写注册表），构建了三级渐进防御机制：
  - **交互式确认（Interactive Safe Mode）**：默认情况下，模型生成的每一段代码与 Shell 指令在终端以语法高亮呈现，必须经由开发者键盘回车显式授权方可执行；
  - **Docker 容器沙箱（`--docker`）**：一键将整个执行环境封装入独立的 Docker 镜像，限制 CPU/内存配额，隔离宿主机真实磁盘与敏感根分区；
  - **细粒度路径挂载与安全白名单**：支持只读挂载特定数据目录，阻断私有密钥目录（如 `~/.ssh`, `~/.aws`）的访问权限。

### 4. 计算机使用与纯视觉操作系统接管（Computer-Use & Vision Mode）
- 引入类似人类使用电脑的 **OS World 控制范式**：结合视觉大模型（如 GPT-4o / Claude 3.5 Sonnet / Qwen2-VL），实时截取屏幕画面，解析桌面 GUI 窗口、按钮与输入框坐标；
- 通过 PyAutoGUI 驱动物理级鼠标点击、拖拽、键盘输入与快捷键触发，实现对无 API 遗留软件、桌面级专有系统（如 Photoshop、CAD、财务客户端）的端到端自主操控。

---

## 🛠️ 快速上手与配置

### 1. 基础安装与终端交互

```bash
# 通过 pip 极速安装
pip install open-interpreter

# 启动交互式命令行模式（默认提示配置模型 API）
interpreter

# 开启安全沙箱模式（自动拉取并运行在 Docker 隔离环境中）
interpreter --docker

# 开启自动执行模式（慎用，仅建议在隔离沙箱或只读测试场景使用）
interpreter -y
```

### 2. 结合本地私有化大模型（Ollama / vLLM）

```bash
# 零公网依赖：使用本地已启动的 Ollama 模型（如 deepseek-coder-v2 或 qwen2.5-coder）
interpreter --local

# 或者显式指定 Ollama 内部模型与自定义端点
interpreter --model ollama/qwen2.5-coder:14b --api_base http://localhost:11434
```

### 3. 嵌入式 Python SDK 工业级开发

```python
from interpreter import interpreter

# 配置模型路由底座
interpreter.llm.model = "claude-3-5-sonnet-20241022"
# interpreter.llm.model = "ollama/qwen2.5-coder:14b"
# interpreter.llm.api_base = "http://localhost:11434"

# 开启系统安全防护与执行沙箱
interpreter.auto_run = False  # 生产环境必须为 False，强制人工审查或安全拦截
interpreter.safe_mode = "ask" # 可选: "off", "ask", "auto"

# 自定义系统指令（注入企业安全基线与规范）
interpreter.system_message += """
【生产安全准则】：
1. 绝对禁止执行任何包含 rm -rf、drop database、mkfs 等不可逆破坏性指令；
2. 处理本地文件前必须先进行备份；
3. 输出的所有数据分析结果，优先持久化为 local_output/ 目录下的结构化 CSV 或 PNG。
"""

# 执行交互式多轮任务
response = interpreter.chat(
    "扫描当前工作目录下的 sales_data.csv，计算各季度的总营收和环比增速，"
    "并将趋势图保存为季度销售分析.png，最后用文字输出分析摘要。"
)
print("Agent 最终交付结果：", response)
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **本地复杂数据探索与自动化分析**：无缝调用 Pandas、Polars、Matplotlib 处理本地大文件，省去手工编写数据清洗脚本的繁琐操作；
  - **系统运维与复杂跨平台自动化**：自动化执行多步复杂的 Git 流水线操作、批量转换媒体格式（FFmpeg）、诊断系统端口占用与性能剖析；
  - **AI 赋能的端侧自治助手**：作为个人工作站上的超级数字员工，自动整理凌乱下载文件夹、批量重命名报表、抓取分析本地日志。
- **潜在不足 / 局限性**：
  - **安全边界挑战**：在启用 `-y`（自动执行）且无 Docker 隔离时，若模型遭遇 Prompt Injection 攻击，可能引发本地恶意代码执行风险；
  - **多轮会话上下文膨胀**：频繁执行输出超长终端日志（如大型项目编译日志或几十万行数据打印）会导致上下文窗口迅速被挤爆，需要配套输出截断机制。
- **个人评估结论**：**建议引入（status/adopted）**。作为个人与团队本地自动化研发的超级瑞士军刀，其与 Docker 沙箱及本地开源大模型的无缝结合，奠定了端侧系统级智能体的事实标准。