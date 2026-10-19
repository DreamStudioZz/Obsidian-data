---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/AgentOps-AI/agentops
stars: "3.5k+"
license: Apache-2.0
date_added: 2026-10-19
---

# 📦 AgentOps

> **一句话简介**：专为自主智能体（Autonomous Multi-Agent）设计的开源**全链路可观测性、会话时间旅行回放与可靠性测试套件**。提供一行代码零侵入集成、执行瀑布流追踪、工具调用失败现场还原、死循环递归熔断以及精准 Token 成本度量，破解多智能体黑盒调试难题。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: AgentOps-AI/agentops](https://github.com/AgentOps-AI/agentops) |
| **技术栈** | Python / TypeScript / REST API / OpenTelemetry |
| **许可证** | Apache-2.0 |
| **核心特性** | 零侵入装饰器打点、会话时间旅行（Time-Travel Replay）、工具失败瀑布还原、死循环与成本熔断器、原生支持 CrewAI/AutoGen/LangGraph |
| **相关关联** | 关联实践: [[自主智能体全链路可观测与可复现调试：基于 AgentOps 的会话回放与故障瀑布分析实战]], [[2026-10-19-AI-Digest]], [[Phoenix]], [[Langfuse]], [[CrewAI]], [[AG2]], [[LangGraph]] |

---

## 🚀 核心特性与架构亮点

### 1. 破解智能体黑盒：细粒度瀑布流跟踪（Fine-Grained Waterfall Traces）
随着 Agent 从“单次提示词调用”迈入“多智能体分工自治（Multi-Agent Choreography）”，传统的 APM（如 Datadog）甚至通用 LLM 追踪工具（如仅记录 Request/Response）面临彻底失效：
- **智能体调用的复合非线性**：一个高级目标可能触发数十次模型思考、数十次外部工具调用（API、代码解释器、SQL 查询）以及子智能体间的委托会话；
- **AgentOps 的动作抽象**：显式建模了 Agent、Session、Action（LLM 决策）、Tool Event（外部工具执行）、Error Event（报错捕获）与 Environment State（上下文状态），自动在时间轴上生成交互调用甘特图与依赖拓扑图，使每一次微小状态突变都有据可查。

### 2. 会话时间旅行与故障回放（Time-Travel Session Replay）
- **复现“幽灵 Bug”**：大模型的非确定性（Non-deterministic）输出使得线上偶现的工具参数幻觉、异常状态转移在本地几乎无法重现；
- **完整环境切片还原**：AgentOps 会完整记录 Agent 每一轮决策的输入 Prompt、内部 Scratchpad、工具返回 Payload、以及当时系统环境变量，支持一键“重放（Replay）”，并支持将线上失败的单步 Tool Event 导出为离线单测用例（Regression Tests）。

### 3. 递归死循环检测与成本熔断器（Recursive Loop Detector & Budget Guardrails）
- **终止无限相互对话**：多智能体系统中经常出现 Agent A 与 Agent B 陷入互相提问/互相道歉的死循环，瞬间吞噬数百万 Token；
- **动态距离阈值熔断**：AgentOps 内部计算连续多轮 Action 的语义距离与参数重复度，一旦检测到死循环特征或会话 Token 消耗超过预设预算阈值，立即触发安全中断并抛出特定异常，避免巨额账单。

### 4. 主流生态无缝一键注入
- 原生内置对主流多智能体框架的自动劫持（Auto-Instrument）：
  - **CrewAI**：一行 `agentops.init()` 自动接管全部 Crew、Agent、Task 执行生命周期；
  - **AutoGen / AG2**：自动跟踪 `UserProxyAgent` 与各类助手智能体的多轮对话；
  - **LangGraph**：自动映射节点（Nodes）与条件边（Edges）流转拓扑；
  - **smolagents / LlamaIndex**：全面兼容。

---

## 🛠️ 快速上手与典型代码示例

### 1. 安装核心库

```bash
pip install agentops crewai
```

### 2. 一行代码注入 CrewAI 多智能体工作流

```python
import os
import agentops
from crewai import Agent, Task, Crew, Process

# 1. 初始化 AgentOps 监控服务 (支持本地私有部署或云端 SaaS)
agentops.init(
    api_key=os.getenv("AGENTOPS_API_KEY"),
    default_tags=["prod-environment", "research-crew"]
)

# 2. 构建多智能体角色 (AgentOps 会自动无感打桩跟踪其所有动作)
researcher = Agent(
    role="资深 AI 技术分析师",
    goal="调研最新 Text-to-SQL 框架与性能基准",
    backstory="你精通 RAG 架构与各类数据库元数据检索优化，擅长提炼核心工程价值。",
    verbose=True
)

writer = Agent(
    role="技术主编",
    goal="将研究成果撰写为通俗易懂的高质量工程实战指南",
    backstory="你擅长梳理技术痛点、架构图解与生产落地避坑建议。",
    verbose=True
)

# 3. 定义任务流水线
task1 = Task(
    description="深入分析 Vanna 框架的三维向量元数据架构与自纠错回路，输出分析要点。",
    expected_output="包含核心亮点、痛点解法与架构对比的技术摘要。",
    agent=researcher
)

task2 = Task(
    description="根据分析要点，撰写一份生产级落地实操指南草案。",
    expected_output="结构化 Markdown 格式指南。",
    agent=writer
)

# 4. 组装并启动 Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[task1, task2],
    process=Process.sequential,
    verbose=True
)

try:
    result = crew.kickoff()
    print("=== 执行结果 ===")
    print(result)
    # 正常结束会话标记
    agentops.end_session("Success")
except Exception as e:
    # 捕获异常并标记失败会话，现场保留在仪表盘
    agentops.end_session("Fail", end_state_reason=str(e))
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **多智能体生产系统可靠性保障**：上线 CrewAI / LangGraph / AG2 企业级应用，必须配置全链路追踪以应对线上审计与报警；
  - **长链路自治 Agent（如自主编码、网页爬取）故障排查**：迅速定位第 15 步工具参数序列化报错根因；
  - **团队研发 CI/CD 评估门禁**：在持续集成中比对 Agent 新旧版本在同一任务集上的工具调用成功率与 Token 开销。
- **潜在不足 / 局限性**：
  - 高并发高吞吐场景下，异步上报网络请求可能略微增加网络 I/O，生产部署建议配置内网 Collector 或批量压缩上报（Batch Dispatch）；
  - 私有化部署控制台相比云端 SaaS 功能更新略有滞后。
- **个人评估结论**：**多智能体（Multi-Agent）时代最贴合 Agentic 行为范式的全链路可观测性王牌框架（Adopted）**。
