---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/langflow-ai/langflow
stars: "55k+"
license: MIT
date_added: 2026-10-22
---

# 📦 Langflow

> **一句话简介**：面向多智能体（Multi-Agent）与复杂 RAG 系统的开源可视化低代码流水线编排与生产级服务化框架。以 **Python 原生优先（Python-First）** 与 **React Flow 动态交互** 为核心基底，支持以完全模块化、强类型的自定义组件（Custom Components）在画布上拖拽装配复杂 DAG 图，并原生支持组件热重载、MCP（Model Context Protocol）协议集成与一键生产级 FastAPI / RESTful 端点导出。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: langflow-ai/langflow](https://github.com/langflow-ai/langflow) |
| **技术栈** | Python 3.10+ / FastAPI / React Flow / Celery / Pydantic v2 / SQLAlchemy |
| **许可证** | MIT License |
| **核心特性** | 可视化拖拽 DAG 画布、Python 原生自定义组件与代码热重载、一键导出生产级 RESTful API / Python SDK、原生 MCP 协议与多智能体分层编排、多会话（Session）与流式（SSE）响应治理 |
| **相关关联** | 关联实践: [[企业级可视化智能体编排：基于 Langflow 的低代码组件定制与生产级 API 服务化实战]], [[2026-10-22-AI-Digest]], [[Dify]], [[LangGraph]], [[FastMCP]], [[CopilotKit]], [[CrewAI]], [[PydanticAI]] |

---

## 🚀 核心特性与架构亮点

### 1. “Python-Native 优先”的自定义组件与热重载体系
与许多将用户锁死在预设黑盒节点中的无代码/低代码平台不同，Langflow 彻底践行“代码即配置，组件即代码”的设计准则：
- **纯 Python 类声明组件**：每个节点在底层均是一个继承自 `Component` 或 `CustomComponent` 的标准 Python 类，使用强类型类型注解与 Pydantic 校验；
- **UI 内实时在线热编写（In-Browser Hot Reload）**：开发者可直接在前端点击“Edit Code”修改节点的 Python 核心逻辑，保存后 Langflow 后端动态重新加载 AST 语法树并即时更新节点端口与连接规约，无需停机或重启进程；
- **全生态 Python 库任意引用**：可在组件内部直接导入 PyTorch、Pandas、本地私有 SDK 或公司内部微服务客户端，彻底消除平台封闭性壁垒。

### 2. 双模驱动：从“画布原型”到“生产级微服务”无缝收敛
Langflow 从设计之初就兼顾了算法原型验证与后端工程化部署：
- **JSON AST 描述与版本化**：整套画板拓扑结构被序列化为标准的 JSON 图描述文件（Flow AST），天然支持 Git 版本控制与 CI/CD 流水线同步；
- **生产级 API 暴露与无头运行（Headless Engine）**：完成连线后，无需依赖前端 UI，后端可直接通过 `langflow.load_flow_from_json()` 将其作为纯 Python 库嵌入业务后端，亦可通过内置 FastAPI 网关一键提供标准 OpenAPI 端点与 cURL 接口；
- **原生 Server-Sent Events (SSE) 流式传输**：天然支持 Token 级流式生成与中间思考过程（Reasoning Steps）的实时推送。

### 3. 多智能体架构（Multi-Agent Orchestration）与分层协作
Langflow 深度整合了现代智能体系统的核心范式：
- **工具调用与动态路由**：支持基于意图识别与语义向量的动态 Router 节点，精准分流用户诉求至垂直领域 Sub-Agent；
- **分级多 Agent 团队**：提供 Supervisor-Worker 模式节点，主智能体负责任务拆解与全局状态机流转，工作智能体并行调用工具执行；
- **原生支持 MCP（Model Context Protocol）**：支持作为 MCP Client 动态发现、挂载和调用本地或远程的 MCP Server 工具，打通本地文件系统、数据库与第三方 API。

### 4. 企业级状态持久化与多租户会话隔离
- **基于会话的内存治理**：内置支持 Redis、PostgreSQL 与 SQLite 存储引擎，支持按 `session_id` 自动管理对话历史、长效记忆与变量上下文；
- **Playground 实时沙箱调试**：支持在画板内直接打开交互式对话框，逐步断点单步调试节点输入输出与 Token 消耗，可视化分析调用耗时瀑布图。

---

## 🛠️ 快速上手与典型代码示例

### 1. 快速启动 Langflow 本地服务

```bash
# 1. 推荐在独立虚拟环境中安装
pip install langflow

# 2. 启动本地可视化服务器（默认监听 http://127.0.0.1:7860）
langflow run --host 0.0.0.0 --port 7860

# 3. 亦可通过官方精简 Docker 容器直接拉起
docker run -d -p 7860:7860 --name langflow langflowai/langflow:latest
```

### 2. 编写自定义 Python 原生组件（Custom Component）

在 Langflow 画布中，开发者可以通过以下精简代码定义一个具备业务特定规则过滤与向量增强的自定义节点：

```python
from langflow.custom import Component
from langflow.io import Output, StrInput, HandleInput
from langflow.schema import Message
import re

class EnterpriseSanitizerComponent(Component):
    display_name = "企业级敏感词清洗器"
    description = "在将 Prompt 投递至 LLM 之前执行正则脱敏与合规校验"
    icon = "shield-check"

    inputs = [
        StrInput(
            name="raw_query",
            display_name="原始用户输入",
            info="待检测与清洗的自然语言 Query"
        ),
        StrInput(
            name="mask_pattern",
            display_name="脱敏正则规则",
            value=r"(\d{3})\d{4}(\d{4})",
            info="默认匹配手机号"
        )
    ]

    outputs = [
        Output(display_name="清洗后文本", name="sanitized_text", method="sanitize")
    ]

    def sanitize(self) -> Message:
        text = self.raw_query
        pattern = self.mask_pattern
        # 执行手机号中间 4 位掩码处理
        sanitized = re.sub(pattern, r"\1****\2", text)
        return Message(text=sanitized)
```

### 3. 在企业外部 Python 业务工程中以无头（Headless）方式调用 Flow

无需启动 Langflow Web UI，直接将保存导出的 `enterprise_agent.json` 作为无头服务嵌入业务代码中：

```python
import asyncio
from langflow.load import run_flow_from_json

async def call_agent_pipeline(user_query: str, session_id: str):
    # 直接加载导出的 Flow 图并异步执行
    result = await run_flow_from_json(
        flow="flows/enterprise_agent.json",
        input_value=user_query,
        session_id=session_id,
        fallback_to_env_vars=True,
        tweaks={
            "EnterpriseSanitizerComponent-xxx": {"raw_query": user_query},
            "OpenAIModel-yyy": {"temperature": 0.2}
        }
    )
    
    # 提取最终输出节点的文本内容
    output_text = result[0].outputs[0].results["text"].data["text"]
    return output_text

# 运行测试
# response = asyncio.run(call_agent_pipeline("请帮我查询订单 13800138000 的状态", "sess_001"))
# print(response)
```

---

## 💡 工程实战点评与适用场景

### 1. 推荐使用场景
- **AI 产品原型与算法工程师快速验证**：以往手写 LangChain 或 LangGraph 代码需要频繁调整链路，通过 Langflow 可视化画板能将 Agent 逻辑原型搭建周期从几天压缩至数小时；
- **业务团队与技术团队协作中台**：业务架构师在画板上梳理逻辑拓扑，算法工程师负责编写高门槛的 Python 业务节点，实现高度解耦；
- **复杂混合 RAG 与多模型动态路由编排**：直观展示文档切块、Embedding 向量化、重排过滤与模型生成的全链路数据流。

### 2. 潜在不足与局限性
- **大规模高并发下的内存管理**：若画布中引入了重型本地模型（如本地加载 sentence-transformers），需配置多进程或 Celery 异步队列，避免单进程主事件循环阻塞；
- **复杂循环图（Cyclic Graph）调试心智负担**：对于高度动态的递归反思回路，可视化连线容易变得密集复杂（“意大利面条式”连线），建议通过子图（Sub-flows）进行模块化封装。

### 3. 个人评估结论
**【强烈建议引入 (Adopted)】**。Langflow 是当前开源界与 Dify、LangGraph 并驾齐驱的顶级 Agent 编排框架。其最大的杀手锏在于“Python-First”理念与无头导出能力，让开发者既能享受低代码的可视化红利，又绝不会丧失编写纯粹、高性能 Python 代码的自由度。
