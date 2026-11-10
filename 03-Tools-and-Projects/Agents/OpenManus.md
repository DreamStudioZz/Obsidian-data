---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/FoundationAgents/OpenManus
stars: "58.9k+"
license: Apache-2.0
date_added: 2026-11-10
---

# 📦 OpenManus

> **一句话简介**：全球最具影响力的开源通用自主智能体（General-Purpose Autonomous Agent）框架，由 FoundationAgents 团队（MetaGPT 核心创作者）重磅打造。旨在打破商业 Agent 产品的封闭垄断与邀请码壁垒，首创模块化分层智能体架构与原生模型上下文协议（Model Context Protocol, MCP）深度生态集成。具备强大的多步自主任务规划、Browser-Use 原生网页端到端交互、终端执行沙箱与反思自纠错机制，为全球开发者提供开箱即用且高度可扩展的开源“全能自治助理”基座。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: FoundationAgents/OpenManus](https://github.com/FoundationAgents/OpenManus) |
| **研发机构** | FoundationAgents 团队（MetaGPT 核心团队主导发起）与全球开源社区 |
| **技术栈** | Python 3.12+, Pydantic, Model Context Protocol (MCP), Playwright, Asyncio, uv |
| **许可证** | Apache-2.0 License |
| **核心机制** | 模块化 Agent 抽象（BaseAgent -> Manus）、动态多步骤计划分解（PlanningAgent）、原生 MCP 工具服务网格接入（Browser-Use CLI MCP、OS Shell、File Tools）、执行链反思自纠错（Step-wise Thought & Reflection） |
| **模型支持** | 支持 OpenAI (GPT-4o/o3-mini)、Anthropic (Claude 3.5/3.7 Sonnet)、DeepSeek (DeepSeek-V3/R1)、Qwen (Qwen2.5/Qwen-VL) 等主流闭源与开源旗舰模型 |
| **强化学习演进** | 衍生开源强化学习对齐项目 `OpenManus-RL`，支持基于 GRPO 算法对智能体决策长链进行端到端微调 |
| **相关关联** | 关联实践: [[自主通用智能体工程实践：基于 OpenManus 的模块化 Agent 架构、MCP 工具编排与长链推理自愈实战]], [[2026-11-10-AI-Digest]], [[MetaGPT]], [[Browser-Use]], [[FastMCP]], [[Cline]], [[SWE-agent]], [[OpenHands]], [[E2B]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph UserLayer [用户交互层]
        UserInput["用户模糊复杂意图<br/>(例如: 调研本周 AI 前沿项目并生成带图表的 Markdown 研报)"]
    end

    subgraph ManusCore [OpenManus 核心运行时架构]
        Controller["OpenManus 核心调度器 (Manus Agent)"]
        
        subgraph PlanningModule [认知与规划中枢 (Planning Engine)]
            Thought["思维链生成 (Chain of Thought)"]
            PlanGen["多步任务拆解 (Dynamic Steps Breakdown)"]
            StateTracker["执行进度追踪 (Progress & Status Tracker)"]
        end

        subgraph ExecutionEngine [执行与反思中枢 (Execution & Reflection)]
            ActionPicker["工具自适应分发 (Tool Selection)"]
            ResultVerifier["环境反馈验证 (Environment Feedback Verifier)"]
            SelfCorrection["异常自愈重试 (Self-Healing Loop)"]
        end
    end

    subgraph ToolMesh [原生 MCP 协议工具网格 (Model Context Protocol)]
        BrowserMCP["Browser-Use MCP Server<br/>(Playwright 真实浏览器交互、视觉快照、DOM 寻址)"]
        ShellMCP["Terminal / OS Shell MCP<br/>(Bash/PowerShell 命令执行、代码运行)"]
        FileMCP["FileSystem MCP<br/>(文件智能读写、研报持久化导出)"]
        CustomMCP["企业自定义 MCP Servers<br/>(ERP、SQL 数据库、飞书/钉钉 Webhook)"]
    end

    UserInput ==> Controller
    Controller --> Thought
    Thought --> PlanGen
    PlanGen --> StateTracker
    StateTracker ==> ActionPicker

    ActionPicker -->|JSON-RPC 协议标准调用| BrowserMCP
    ActionPicker -->|JSON-RPC 协议标准调用| ShellMCP
    ActionPicker -->|JSON-RPC 协议标准调用| FileMCP
    ActionPicker -->|JSON-RPC 协议标准调用| CustomMCP

    BrowserMCP -.->|执行结果 / 截屏视觉 / 报错| ResultVerifier
    ShellMCP -.->|标准输出 / Exit Code| ResultVerifier
    FileMCP -.->|文件句柄 / 变更 diff| ResultVerifier
    CustomMCP -.->|API 响应数据| ResultVerifier

    ResultVerifier -->|校验通过| StateTracker
    ResultVerifier -.->|检测到失败/死循环| SelfCorrection
    SelfCorrection ==>|动态修正后续步骤| PlanGen
```

1. **彻底打破封闭壁垒的工业级开源通用自治架构**：
   - 商业通用智能体（如 Manus）的横空出世展示了端到端自主解决真实世界复杂任务的巨大潜力，但其封闭代码与高昂的订阅门槛限制了企业的深度定制。
   - OpenManus 在数日内迅速集聚开源力量，通过高度模块化的设计将通用 Agent 的核心骨架透明化。它不依赖专有黑盒协议，而是构建在标准化的 Python 异步体系之上，使得任何企业和个人开发者都能快速拥有属于自己的本地化、私有化自治助理。
2. **原生整合 Model Context Protocol (MCP) 标准工具生态**：
   - 传统智能体框架（如早期 AutoGPT 或简易 LangChain Agent）大多采用定制私有 Function Call 格式，工具扩展脆弱且碎片化。
   - OpenManus 全面拥抱 Anthropic 倡导的 **Model Context Protocol (MCP)** 标准，通过客户端直接集成诸如 `uvx browser-use --cli-mcp` 等标准化服务端。这使得智能体不仅能随时调用本地文件系统与终端 Shell，还能跨进程、无缝挂接全球社区数以万计的现成 MCP 扩展，彻底解决了工具调用协议碎片化的历史顽疾。
3. **分层任务规划与端到端反思自愈机制（Plan-Execute-Reflect Loop）**：
   - 面对跨度极大、需要数十步交互的宏观指令，OpenManus 摒弃了单步即兴生成的盲目决策机制，内置专职的规划引擎（Planning Module）。
   - 规划引擎首先将总任务拆解为具有依赖关系的步骤流（Steps Queue）；在执行每个步骤后，结合环境真实返回（DOM 变化、终端退出码、网络响应），触发反思校验。一旦发现环境状态与预期不符（如网页弹出验证码、依赖安装失败），能够动态重整后续执行拓扑，展现出极强的任务韧性。
4. **轻量极速与现代 Python 工具链（uv 原生驱动）**：
   - 全面支持新一代 Rust 极速包管理器 `uv`，摆脱了传统 Anaconda 或臃肿虚拟环境构建慢、体积大、依赖冲突频繁的痛点，秒级完成依赖解析与环境冷启动。
   - 框架核心代码精炼清晰，核心基类 `BaseAgent`、`Manus`、`Tool` 设计严密，极大降低了二次开发与企业定制的技术门槛。

---

## 🛠️ 快速上手与配置

### 1. 环境准备与极速初始化

推荐使用现代 Python 3.12+ 与 `uv` 工具链进行初始化：

```bash
# 1. 克隆官方代码库
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus

# 2. 通过 uv 快速创建并激活虚拟环境 (推荐)
uv venv --python 3.12
source .venv/bin/activate # Windows: .venv\Scripts\activate

# 3. 极速安装核心依赖与 Playwright 驱动
uv pip install -r requirements.txt
playwright install chromium
```

### 2. 配置文件配置 (`config/config.toml`)

在项目根目录下配置大模型 API 与 MCP 服务参数：

```toml
# config/config.toml: OpenManus 基础配置
[llm]
# 支持: openai, anthropic, deepseek, ollama 等
provider = "deepseek"
model = "deepseek-reasoner" # 或 deepseek-chat, gpt-4o, claude-3-5-sonnet
api_key = "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
base_url = "https://api.deepseek.com/v1"
max_tokens = 8192
temperature = 0.2

# 智能体核心运行参数
[agent]
max_steps = 30                     # 最大自主执行步数限制
workspace_dir = "./workspace"      # 本地文件输出与隔离工作区
enable_browser = true              # 启用 Browser-Use 原生自动化
browser_headless = false           # 设为 false 可实时直观观看浏览器自主操作过程
```

### 3. 一键运行与任务下发

```bash
# 启动交互式终端自治运行
python main.py
```

在终端中输入你的宏观目标：
```text
OpenManus > 请帮我调研 GitHub 上 2026 年最新最火的 LLM 推理加速开源项目，抓取它们的核心亮点与 Star 数，并在 workspace 目录下生成一份排版优雅的 markdown 调研报告。
```

### 4. Python 代码级深度调用与自定义 Agent

```python
# run_custom_agent.py: 代码级编排 OpenManus
import asyncio
from app.agent.manus import Manus
from app.logger import logger

async def run_enterprise_workflow():
    # 初始化 Manus 自治智能体
    agent = Manus()
    
    task_prompt = (
        "分析竞争对手官网的最新定价模型："
        "1. 使用浏览器访问 target-site.com/pricing；"
        "2. 提取不同 Tier 的价格、限制与特色功能；"
        "3. 在本地 workspace/pricing_summary.json 保存结构化 JSON 数据；"
        "4. 输出最终的分析结论。"
    )
    
    logger.info("🚀 启动 OpenManus 自动化自治任务流...")
    try:
        # 执行完整的规划-执行-自愈闭环
        final_result = await agent.run(task_prompt)
        print("\n✅ 任务成功交付：\n", final_result)
    except Exception as e:
        logger.error(f"❌ 任务执行异常: {str(e)}")
    finally:
        # 确保浏览器与沙箱环境优雅销毁
        await agent.close()

if __name__ == "__main__":
    asyncio.run(run_enterprise_workflow())
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **自动化市场调研与长链信息搜集**：需要跨越数个网页进行登录、检索、翻页、过滤多格式内容并最终聚合为结构化报告的复杂场景；
  2. **端到端测试与真实系统操作自动化**：替代传统编写繁复的 UI 自动化测试用例，交由 Agent 理解自然语言需求并自主驱动真实系统验证；
  3. **研发辅助与跨系统自治集成**：借助 MCP 协议，连接私有 Git 仓库、数据库与告警平台，实现故障排查、日志提取与代码补丁自动化。
- **潜在不足与局限性**：
  - **长链执行的 Token 消耗与成本**：在面对极长任务时，随着多轮思考和页面 DOM 信息的不断累积，Prompt 长度迅速膨胀，需合理配置 Context Window 截断或选用高性价比推理模型（如 DeepSeek）；
  - **复杂反爬与人机验证（CAPTCHA）阻断**：虽然集成了 Browser-Use，但遇到强风控阻断（Cloudflare 5秒盾、极验等）时依然可能需要人工接管机制。
- **个人评估结论**：**当前全球开源生态中最成熟、演进最迅速的通用自主 Agent 典范，是构建私有企业级自治 Agent 系统的首选参考底座**。
