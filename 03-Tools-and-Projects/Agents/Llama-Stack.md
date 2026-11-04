---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/meta-llama/llama-stack
stars: "7.2k+"
license: MIT
date_added: 2026-11-04
---

# 📦 Llama-Stack

> **一句话简介**：由 Meta 官方重磅推出的全生命周期标准化大模型应用与智能体协议规范架构（Standardized APIs & Building Blocks for Llama Ecosystem）。一举打破模型厂商私有 SDK 割裂局面，统一定义了推理（Inference）、安全拦截（Safety / Llama Guard）、外挂记忆（Memory）、有状态智能体（Agents Turn Loop）与工具执行（Tool Runtimes）的标准服务契约，支持在端侧（Ollama / ExecuTorch）、私有数据中心（vLLM / TGI）与公有云（AWS / Azure / Fireworks）之间零修改无缝分发。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: meta-llama/llama-stack](https://github.com/meta-llama/llama-stack) |
| **研发机构** | Meta AI (Llama Team) & 开源社区标准推进组织 |
| **技术栈** | Python, FastAPI, OpenAPI Spec, Docker, PyPI / Conda CLI, Pydantic |
| **许可证** | MIT License |
| **核心协议层** | Inference API, Safety API (Llama Guard), Memory API (Vector/Graph/RAG), Agents API (Turn Loop), Tool Runtime API (Code Interpreter/Search) |
| **支持发行版** | Meta Reference Distribution, Ollama Local, vLLM High-throughput, Together/Fireworks Cloud, AWS Bedrock, Dell On-Premises |
| **相关关联** | 关联实践: [[企业级统一智能体底座：基于 Meta Llama-Stack 的标准化 API 编排与跨端混合运行时实战]], [[2026-11-04-AI-Digest]], [[vLLM]], [[Ollama]], [[LiteLLM]], [[FastMCP]], [[LangGraph]], [[PydanticAI]] |

---

## 🚀 核心特性与架构亮点

### 1. 终结碎片化的统一标准化协议（Standardized APIs）
在过去，工程师在不同环境中构建 Agent 时面临极其严重的“胶水代码地狱”：本地调试用 Ollama API，上线生产用 vLLM 或云端 API；向量库换成 Milvus 需要重写一套检索逻辑；接入安全护栏又要嵌套额外的中间件。
Llama Stack 首次以行业标准的高度制定了五个核心 OpenAPI 接口契约：
- **Inference API**：统一纯文本与多模态流式补全、结构化 JSON 生成与 Logits 约束；
- **Safety API**：开箱即用支持 Llama Guard 3 与 Prompt Guard，支持用户输入与模型输出的双向安全合规前置/后置审查；
- **Memory API**：提供标准的向量检索、图谱关联与文档持久化接口，上层应用无需关心底层究竟是 Chroma、pgvector 还是 Milvus；
- **Tool Runtime API**：统一代码沙箱（Code Interpreter）、网页搜索（Brave / Tavily）与企业内部自定义 MCP 协议工具的执行生命周期；
- **Agents API**：规范化管理多轮对话状态机（Turn Loop）、工具自动选择与执行编排，向开发者屏蔽底层的轮询与调度细节。

### 2. 声明式发行版体系（Llama Stack Distributions）
- 借鉴现代操作系统 Linux 的“发行版（Distribution）”哲学，Llama Stack 提供了高度声明式的 Provider 装配机制；
- 开发者只需通过一份标准 `run.yaml` 声明所采用的后端引擎（如推理采用 vLLM，向量库采用 Qdrant，安全采用本地 Llama Guard），即可通过 `llama stack run` 一键拉起完全符合同一套 OpenAPI 标准的统一服务端点；
- 应用层客户端（无论是 Python SDK、Node.js 还是移动端）只与标准契约通信，实现**“一次编码，任意环境交付”**。

### 3. 原生全双工安全与合规拦截链（Built-in Safety Guardrails）
- 告别业务代码中硬编码的敏感词过滤与脆弱的正则表达式，Llama Stack 原生支持在请求入口与响应输出两个关键节点挂载 Safety Shield；
- 每次用户提问先经由 Safety Provider（如轻量级 Llama-Guard-3-1B）毫秒级研判，一旦触发有害内容评级直接拦截返回违规标签；模型输出时同样进行后验审查，构筑企业级生产合规闭环。

---

## 🛠️ 快速上手与配置

### 1. 安装与拉起本地发行版服务

```bash
# 安装 Llama Stack 核心 CLI 与客户端库
pip install llama-stack

# 查看并选择适合的发行版模板 (如 local-ollama, local-vllm, fireworks 等)
llama stack list-distributions

# 构建并启动一个基于本地 vLLM 与内置工具的发行版服务端
llama stack run local-vllm --port 8321
```

### 2. 基于标准客户端构建具备记忆与工具调用的安全智能体

```python
import os
from llama_stack_client import LlamaStackClient
from llama_stack_client.types import UserMessage, SystemMessage

# 1. 连接统一标准服务端点 (无论是本地运行还是私有云端)
client = LlamaStackClient(base_url="http://localhost:8321")

# 2. 注册可挂载的标准工具 (支持 Python 代码沙箱与内置搜索)
available_tools = client.tools.list()
print(f"🔧 已加载的标准工具组件: {[t.identifier for t in available_tools]}")

# 3. 创建具备安全护栏与记忆能力的标准化 Agent Session
agent_config = {
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "instructions": "你是一个严谨的资深财务分析专家，善于调用计算工具并保证输出准确客观。",
    "enable_session_persistence": True,
    "input_shields": ["llama-guard-3-1b"], # 挂载输入安全防护盾
    "output_shields": ["llama-guard-3-1b"], # 挂载输出安全审查盾
    "tools": [
        {"type": "builtin::code_interpreter"}, # 启用代码解释器沙箱
        {"type": "builtin::brave_search"}       # 启用标准化网页搜索
    ]
}

agent_session = client.agents.create(
    agent_config=agent_config
)
print(f"🤖 标准化 Agent 会话创建成功: Session ID = {agent_session.agent_id}")

# 4. 发起带有复杂工具计算诉求的交互轮次 (Turn)
response_turn = client.agents.turns.create(
    agent_id=agent_session.agent_id,
    session_id=agent_session.session_id,
    messages=[
        UserMessage(
            content="请查询 2026 年初美联储的最新基准利率区间，并用 Python 计算如果企业借款 500 万美元，年化利息支出是多少？"
        )
    ],
    stream=False
)

# 5. 打印遵循标准化契约输出的完整执行结果
print("\n📋 Agent 结构化输出结果:")
print(response_turn.output_message.content)

# 6. 查看工具调用与安全执行足迹 (Steps Audit)
for step in response_turn.steps:
    print(f"👉 步骤类型: {step.step_type} | 状态: {step.status}")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **企业级标准化大模型网关与开发中台**：需要统一规范公司内部各个业务线对 LLM、RAG、Agent 和 Guardrails 的调用方式，坚决杜绝“各写各的 SDK”；
  - **混合云与边缘端多环境部署**：需要将同一套智能体系统既交付在内网私有 GPU 机房，又交付在公有云甚至客户本地工控机/边缘设备上；
  - **金融与合规强监管场景**：对用户输入与模型输出具有严苛的合规审查要求，依赖标准化安全切面实现无感拦截与审计追踪。
- **与同类工具对比（vs LangChain / LiteLLM / FastMCP）**：
  - 相比于 **LiteLLM**：LiteLLM 专注于“推理层 Proxy 网关（Inference Gateway）”，解决多模型路由与计费；而 Llama Stack 覆盖了 Agent 状态机、Memory 存储、安全 Shield 和工具沙箱的全生命周期；
  - 相比于 **LangChain**：LangChain 抽象层次极多、层层包裹且版本频繁不兼容；Llama Stack 是由底座模型厂商（Meta）主导制定的声明式、轻量化 OpenAPI 开放协议，更加纯粹、工业化与契约化；
  - 相比于 **FastMCP**：FastMCP 专注于 Anthropic 主导的模型上下文工具协议（MCP）；Llama Stack 则全面包容并统一了工具、记忆、推理与安全的所有上层契约。
- **个人评估结论**：**强烈推荐采纳 (status/adopted)**。作为 Meta 官方推动的行业规范，Llama Stack 正在重塑开源大模型应用的基础设施格局，是构建高内聚、低耦合企业级 AI 底座的必选架构参考。
