---
tags:
  - ai-tool
  - open-source
  - multi-agent
  - status/adopted
category: Agents
github: "https://github.com/ag2ai/ag2"
stars: "35k+"
license: "Apache-2.0"
date_added: 2026-10-08
---

# 📦 AG2 (Formerly AutoGen)

> **一句话简介**：原 AutoGen 进化版，专注于生产级多智能体协作、状态会话管理与分布式协同的下一代开放 Agent 框架。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - ag2ai/ag2](https://github.com/ag2ai/ag2) |
| **技术栈** | Python, AsyncIO, WebSocket |
| **许可证** | Apache-2.0 |
| **相关关联** | 关联专题: [[CrewAI]], [[LangGraph]], [[OpenHands]] |

---

## 🚀 核心特性与亮点
1. **架构重构与生产级解耦**：从微软研究原型彻底进化为独立组织孵化的企业级框架，解决原 AutoGen 历史代码中消息死循环与状态不可控问题。
2. **丰富的多智能体通信拓扑**：支持群聊（GroupChat）、层级指挥（Hierarchical）、双人对抗（Two-agent Chat）及事件驱动的异步订阅总线。
3. **原生代码执行与人机协同（Human-in-the-loop）**：提供沙箱化代码运行环境与可插拔的人工审批拦截点。
4. **与同类工具对比**：
   - 相比于 CrewAI：AG2 在自由对话循环与复杂博弈反思场景更具灵活性；
   - 相比于 LangGraph：AG2 提供了更高层级的角色抽象，上手曲线更平缓。

---

## 🛠️ 快速上手与配置

```bash
pip install ag2[openai]
```

```python
from autogen import ConversableAgent, UserProxyAgent

assistant = ConversableAgent(
    name="assistant",
    system_message="你是一个专业的 Python 算法架构师。",
    llm_config={"model": "gpt-4o", "temperature": 0.2},
)

user_proxy = UserProxyAgent(
    name="user_proxy",
    code_execution_config={"work_dir": "coding", "use_docker": False},
    human_input_mode="NEVER",
)

user_proxy.initiate_chat(
    assistant,
    message="请实现一个带 TTL 淘汰机制的高并发本地缓存类。",
)
```

---

## 💡 工程实战点评与适用场景
- **推荐使用场景**：复杂研发流程自动化（Coder + Reviewer + Tester）、企业级多角色推演仿真、研究型自主探索。
- **潜在不足 / 局限性**：多智能体交互产生的 Token 消耗增速较快，生产落地时需严密配合 Token Caching 与最大轮次阈值。
- **个人评估结论**：建议引入作为复杂多 Agent 任务编排的核心选型之一。
