---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/langchain-ai/langgraph
stars: "22k+"
license: MIT
date_added: 2026-09-22
---

# 📦 LangGraph

> **一句话简介**：LangChain 官方主导的有状态、可持久化多智能体编排引擎，通过图（Graph）结构管理循环执行、状态快照（Checkpointing）与人机协作（Human-in-the-Loop）。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| **项目定位** | 生产级有状态多智能体图引擎（Stateful Agent Graph） |
| **核心机制** | Pregel 计算模型、状态流转（StateGraph）、检查点持久化 |
| **存储适配** | Memory, SQLite, PostgreSQL, Redis |
| **关联概念** | [[000-AI-Index]], [[2026-09-22-AI-Digest]] |

---

## 🚀 为什么复杂企业级 Agent 必须依赖图架构？

简单的线性 Agent（如单向 Chain 或朴素 ReAct 循环）无法应对工业生产级需求：
1. **无法稳定循环与条件分支**：生产流程中经常需要“评估不合格时回退到上一个节点重试”，树状或线性链极难维护；
2. **缺乏状态恢复与持久化（Durability）**：服务器宕机或网络抖动后，跑了 10 分钟的长流程全部丢失，必须从头开始；
3. **无法实现真正的人机协作（Human-in-the-Loop）**：在敏感节点（如转账确认、发布上线代码）需要暂停等待人工审批，审批通过后再从断点继续推进。

**LangGraph 的核心优势**：
1. **基于图的状态机（StateGraph）**：
   - 将工作流建模为节点（Node，计算步骤）和边（Edge，流转逻辑），原生支持循环、分支合并与动态路由；
2. **生产级检查点（Checkpointing）**：
   - 每执行完一个 Node，自动将当前完整状态序列化持久化至数据库（如 PostgreSQL / Redis），支持任务断点续跑与“时间旅行调试（Time Travel）”；
3. **断点中断与人机接管（Breakpoints & Approvals）**：
   - 可以在任意敏感操作节点前声明 `interrupt_before=["deploy_node"]`，流程暂停等待外部人工干预并修改状态后继续。

---

## 🛠️ 快速上手示例

### 1. 安装
```bash
pip install langgraph langchain-core langchain-openai
```

### 2. 构建一个带条件反馈的审核图流程
```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver

# 1. 定义全局共享状态
class WorkflowState(TypedDict):
    content: str
    feedback: str
    is_approved: bool
    iteration_count: int

# 2. 定义业务执行节点
def generate_draft(state: WorkflowState):
    """草稿生成/修改节点"""
    return {
        "content": f"{state.get('content', '')} [已根据反馈修订]",
        "iteration_count": state.get("iteration_count", 0) + 1
    }

def review_draft(state: WorkflowState):
    """质检审核节点"""
    # 模拟审核规则：迭代达到2次则通过
    if state["iteration_count"] >= 2:
        return {"is_approved": True, "feedback": "质量达标"}
    return {"is_approved": False, "feedback": "内容不够严密，请继续补充案例"}

# 3. 定义条件流转路由
def should_continue(state: WorkflowState):
    if state["is_approved"]:
        return END  # 结束
    return "generate_draft"  # 打回修改形成有界循环

# 4. 组装状态图
workflow = StateGraph(WorkflowState)
workflow.add_node("generate_draft", generate_draft)
workflow.add_node("review_draft", review_draft)

workflow.set_entry_point("generate_draft")
workflow.add_edge("generate_draft", "review_draft")
workflow.add_conditional_edges("review_draft", should_continue)

# 5. 绑定持久化检查点并编译运行
checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "session_1001"}}
final_state = app.invoke({"content": "初始AI架构方案"}, config=config)
print("最终输出:", final_state)
```

---

## 💡 工程落地建议与适用场景

- **适用场景**：
  - **复杂多代理协同（Multi-Agent Supervisor）**：规划者、执行者、评审者之间的状态交接与任务调度；
  - **长生命周期异步工作流**：跨越数小时乃至数天的人工审核流水线；
  - **高可靠性要求业务**：必须防范服务器意外重启导致的状态丢失。
- **与普通 LangChain 区别**：
  - 彻底解除了臃肿的黑盒链，以更具掌控力的数据流图驱动，适合复杂工业级系统。
- **综合评估结论**：生产级复杂状态化 Agent 事实标准，强烈推荐采纳（Adopted）。
