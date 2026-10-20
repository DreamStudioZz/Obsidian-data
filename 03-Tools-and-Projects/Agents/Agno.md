---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/agno-agi/agno
stars: "18.5k+"
license: Apache-2.0
date_added: 2026-10-20
---

# 📦 Agno

> **一句话简介**：专为构建**多模态、超轻量、低延迟自主智能体系统**打造的高性能纯 Python 编排底座。前身为 Phidata，彻底抛弃了重型抽象层，相比传统编排框架执行速度提升数千倍，原生支持多智能体团队（Teams）协同、多模态音视频推理、长期记忆存储（Memory & Storage）与链式自我验证（Reasoning）。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: agno-agi/agno](https://github.com/agno-agi/agno) (原 Phidata) |
| **技术栈** | Python 3.10+ / Pydantic v2 / PostgreSQL / SQLite / DuckDB |
| **许可证** | Apache-2.0 |
| **核心特性** | 纯 Python 零多余依赖、毫秒级执行开销、原生多智能体团队编排、原生支持音视频与图文多模态推理、持久化记忆与向量知识库无缝挂载、Reasoning 链式思维推演 |
| **相关关联** | 关联实践: [[高性能多模态智能体架构：基于 Agno 的轻量化多智能体团队编排与持久化记忆实战]], [[2026-10-20-AI-Digest]], [[CrewAI]], [[LangGraph]], [[PydanticAI]], [[smolagents]] |

---

## 🚀 核心特性与架构亮点

### 1. 极致轻量与零抽象负担（Zero Overhead & Pure Python）
传统智能体框架（如 LangChain）为了追求通用性引入了多达数十层的抽象基类、回调链和事件总线，导致单次智能体决策前置开销高达数百毫秒，且 Traceback 极难定位排查。
- **Agno 的极简设计哲学**：核心类仅为 `Agent`、`Team`、`Tool` 与 `Storage`，代码全部直观透明；
- **微秒级运行时延迟**：在基础调用链路中，Agno 的 Python 层封装开销几乎趋近于零，大模型流式响应直接透传，吞吐量相比重型封装框架提升数十倍；
- **开箱即用原生类型系统**：全面拥抱 Pydantic v2，天然具备强类型输入校验与确定性结构化抽取能力。

### 2. 原生多智能体团队编排（Multi-Agent Teams）
不同于其他框架需要显式定义状态图或复杂的 Actor 消息邮箱，Agno 原生提供直观的团队模式：
- **团队领队（Team Leader / Router）**：自动分析用户意图，将子任务分发给拥有对应专长的智能体成员；
- **专家成员（Team Members）**：各司其职，分别绑定独立专业工具（如金融财报查询、学术文献检索、Python 代码执行）；
- **协作模式灵活可配置**：支持轮询委托、层级汇报、结果汇总评审等多种团队协作范式。

### 3. 全模态原生感知（Native Multimodal Capabilities）
Agno 是业界首批将图像、音频、视频和结构化文档作为一等公民处理的开源 Agent 框架：
- **视频与长音频理解**：直接向 Agent 输入本地视频文件路径或 YouTube 链接，框架自动完成多模态帧抽取、音频时间轴对齐，配合 Gemini / GPT-4o 进行细粒度帧级问答；
- **多模态图表交互**：Agent 可直接审视数据可视化图表并输出趋势推断，无需多阶段串行 OCR 拼接。

### 4. 分层长期记忆与持久化会话（Persistent Memory & Sessions）
- **会话存储（Session Storage）**：内置对 SQLite、PostgreSQL 的开箱即用支持，自动记录多轮对话与中间工具调用记录；
- **用户画像与个性化记忆（Personalized Memory）**：不仅记录 Raw Message，更自动沉淀用户偏好、业务实体与关键事实（Fact Extraction），跨会话无缝注入上下文。

### 5. 显式推演机制（Reasoning Agents）
内置两阶段决策模型：在执行高风险工具前，Agent 会自主开启 `show_reasoning=True` 思维链分支，推导前置依赖与逻辑一致性，从根源上降低多步复杂任务中的幻觉率。

---

## 🛠️ 快速上手与典型代码示例

### 1. 依赖安装与环境准备

```bash
pip install agno openai duckduckgo-search yfinance sqlalchemy
```

### 2. 构建协同多智能体团队（Web 搜索 Agent + 金融分析 Agent）

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.duckduckgo import DuckDuckGoTools
from agno.tools.yfinance import YFinanceTools

# 1. 互联网实时资讯检索智能体
web_agent = Agent(
    name="Web Searcher",
    role="负责检索全球互联网最新行业动态与突发新闻",
    model=OpenAIChat(id="gpt-4o"),
    tools=[DuckDuckGoTools()],
    instructions=["始终提供权威可靠的信息来源链接", "优先总结最近 24 小时内的关键进展"],
    show_tool_calls=True,
    markdown=True,
)

# 2. 金融数据深度分析智能体
finance_agent = Agent(
    name="Financial Analyst",
    role="负责获取股票行情、财报数据与估值指标",
    model=OpenAIChat(id="gpt-4o"),
    tools=[YFinanceTools(stock_price=True, analyst_recommendations=True, company_info=True)],
    instructions=["使用表格形式清晰呈现财务指标与分析师评级", "标注币种与交易时间点"],
    show_tool_calls=True,
    markdown=True,
)

# 3. 组建智能体团队（由 Leader 自动统筹决策与结果融合）
agent_team = Agent(
    team=[web_agent, finance_agent],
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "首先让 Web Searcher 获取标的公司近期重大利好/利空新闻",
        "随后让 Financial Analyst 调取最新股价变动与分析师目标价",
        "最后由你汇总输出一份包含定量指标与定性分析的综合投资备忘录",
    ],
    show_tool_calls=True,
    markdown=True,
)

# 4. 执行综合任务
agent_team.print_response(
    "分析英伟达（NVDA）最新的财报表现以及市场对其 Blackwell 架构量产交付的最新反应",
    stream=True
)
```

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **高并发智能体 API 服务**：需要极低框架层延迟、高吞吐纯 Python 异步运行的生产中台；
  - **多模态分析应用**：涉及长视频内容摘要、会议录音审查、数据图表多模态解读的业务系统；
  - **垂直领域专家团队**：需将复杂研报、合规风控或代码重构拆解给多个轻量级智能体协同解决的场景。
- **潜在局限性与避坑点**：
  - 由 Phidata 更名为 Agno 后，部分早期的社区包命名空间有微调，需锁定 `>=1.0.0` 统一规范导入；
  - 团队模式由 Leader LLM 进行自主任务路由，如果底层模型指令遵循能力较弱（如小尺寸量化模型），可能会出现调用子智能体不精准的问题，建议领队模型选用 GPT-4o 或 Claude 3.5 Sonnet。
- **综合评估结论**：**强烈推荐作为新一代生产级智能体主力框架（Adopt）**。在代码可读性、执行性能与多模态原生支持上达到了当前开源生态的巅峰水准。
