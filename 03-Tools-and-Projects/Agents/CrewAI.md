---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/crewAIInc/crewAI
stars: "58k+"
license: MIT
date_added: 2026-09-23
---

# 📦 CrewAI

> **一句话简介**：业界领先的角色扮演型多智能体协作编排框架，通过“角色（Role）、目标（Goal）、背景故事（Backstory）”的高级抽象，让多个专精智能体像人类团队一样进行分工沟通、委派授权、记忆共享与事件流协同（Crew Flows）。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) |
| **技术栈** | Python 原生、支持 LangChain / LiteLLM 工具链 |
| **核心特点** | 角色化多 Agent 协同、层级化（Hierarchical）委派、事件流（Flows）状态编排 |
| **生态集成** | OpenAI, Anthropic, Ollama, LiteLLM, Langfuse, Serper |
| **关联实践** | [[多智能体协同工程：基于 CrewAI 的角色编排与层级流（Flows）实践]], [[LangGraph]], [[PydanticAI]] |

---

## 🚀 核心架构与创新亮点

1. **拟人化角色建模（Role-Based Agents）**：
   - 彻底摆脱僵硬冰冷的 System Prompt，通过 `Role`（身份）、`Goal`（职责定位）和 `Backstory`（背景经历与行为准则）塑造性格专精的专业 Agent。
2. **多模式协同流程（Process Orchestration）**：
   - **顺序模式（Sequential）**：按任务列表线性传递产物，前序输出作为后序输入；
   - **层级管理模式（Hierarchical）**：自动指定或创建 Manager Agent（项目经理），由 Manager 根据任务全景动态指派工作、评估交付质量并决定是否打回重做。
3. **Crew Flows（事件驱动工作流）**：
   - 引入确定性状态管理与事件监听（`@start()`, `@listen()`, `@router()`），将确定性 Python 业务逻辑与非确定性多 Agent 思考无缝拼装。
4. **长短期记忆体系（Memory Systems）**：
   - 内置短时工作记忆（Short-term Memory）、长期事实记忆（Long-term Memory）与实体记忆（Entity Memory），确保多轮迭代中角色不遗忘核心信息。

---

## 🛠️ 快速上手示例

### 1. 安装
```bash
pip install crewai crewai-tools
```

### 2. 组建“前沿调研员 + 技术作家”协作团队
```python
from crewai import Agent, Task, Crew, Process

# 1. 定义角色专精 Agent
researcher = Agent(
    role="前沿 AI 技术调研员",
    goal="挖掘行业最新的开源项目突破与核心架构变革",
    backstory="你是一名资深技术猎头和科技记者，嗅觉敏锐，擅长从代码仓库和技术白皮书中提炼干货。",
    verbose=True,
    memory=True
)

writer = Agent(
    role="技术架构专栏作家",
    goal="将晦涩的技术调研结果提炼成通俗易懂且富有工程深度的落地卡片",
    backstory="你是一名在大厂工作多年的技术布道师，擅长写作清晰的代码示例和避坑指南。",
    verbose=True
)

# 2. 定义阶段任务
task1 = Task(
    description="调研 2026 年最热门的多智能体协同框架，重点总结其状态机与委派机制。",
    expected_output="一份包含 3 个核心框架对比与架构亮点的 markdown 调研报告。",
    agent=researcher
)

task2 = Task(
    description="基于技术调研员的报告，撰写一篇针对软件工程师的实战避坑指南。",
    expected_output="一篇包含架构图说明、代码示例与防死循环策略的结构化实战文章。",
    agent=writer
)

# 3. 组装团队并启动执行
tech_crew = Crew(
    agents=[researcher, writer],
    tasks=[task1, task2],
    process=Process.sequential # 顺序执行流
)

result = tech_crew.kickoff()
print(result)
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **内容生成与长文产出流水线**：调研 -> 撰写 -> 审校 -> 排版；
  - **市场竞品与商业智能分析**：数据抓取 Agent -> 财报分析 Agent -> 风险评估 Agent -> 汇报输出；
  - **跨领域多模态协作系统**：不同 Agent 各自负责特定领域的专属工具调用。
- **与 LangGraph 对比**：
  - **CrewAI**：高层抽象丰富，组建团队仅需几行声明，天然契合“人类组织分工协作”的业务心理模型；
  - **LangGraph**：底层图控制力极强，更适合底层状态机极其严苛、分支循环复杂的工业系统。
- **综合评估结论**：角色协作与团队模拟类应用首选框架，强烈推荐采纳（Adopted）。
