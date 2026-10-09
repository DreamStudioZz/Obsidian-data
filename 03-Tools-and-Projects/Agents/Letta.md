---
tags:
  - ai-tool
  - open-source
  - agent-os
  - memory
  - status/adopted
category: Agents
github: "https://github.com/letta-ai/letta"
stars: "16k+"
license: "Apache-2.0"
date_added: 2026-10-09
---

# 📦 Letta (Formerly MemGPT)

> **一句话简介**：首个基于操作系统分层架构的有状态智能体运行时（Stateful Agent OS），通过核心记忆、召回记忆与归档记忆的分层管理，赋予 Agent 自主编辑与无限时序记忆能力。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - letta-ai/letta](https://github.com/letta-ai/letta) |
| **技术栈** | Python, FastAPI, PostgreSQL, pgvector |
| **许可证** | Apache-2.0 |
| **相关关联** | 关联专题: [[Mem0]], [[AG2]], [[LangGraph]], [[PydanticAI]] |

---

## 🚀 核心特性与亮点

1. **类操作系统的分层记忆体系（Hierarchical Memory Architecture）**：
   - **Core Memory（核心记忆）**：类似于 CPU 寄存器与 L1 缓存，常驻当前 Prompt 上下文（划分 `<human>` 与 `<persona>` 块），支持 Agent 自主实时重写与扩充；
   - **Recall Memory（召回记忆）**：类似于主存 RAM，完整记录历史多轮对话事件流与时间戳，提供精确的时序与上下文检索；
   - **Archival Memory（归档记忆）**：类似于磁盘持久存储，基于向量数据库与语义检索技术存储大规模知识库与外部文档，Agent 可自主分页查阅。
2. **Agent 自主记忆自编辑机制（Self-Editing Memory Tools）**：
   - 内置 `core_memory_append`、`core_memory_replace`、`archival_memory_insert` 和 `archival_memory_search` 等函数调用，使模型能够像操作系统管理虚拟内存一样自主决定写入与置换。
3. **开箱即用的生产级 Agent 服务端（Letta Server & REST API）**：
   - 提供工业级客户端/服务端架构，支持多用户、多 Agent 实例隔离，集成 RESTful API、WebSocket 流式响应与 CLI 交互式调试面板。
4. **与同类工具对比**：
   - **相比于 Mem0**：Mem0 偏向轻量级语义记忆外挂模块，而 Letta 是完整的**有状态智能体运行时系统（Agent Runtime）**，包含完整的上下文调度器与生命周期控制；
   - **相比于传统 LangChain Agent**：彻底消除会话结束即丢失状态的弊病，原生支持跨会话的长周期连续进化。

---

## 🛠️ 快速上手与配置

```bash
# 安装 Letta 服务与客户端
pip install letta

# 启动本地服务
letta server
```

```python
from letta_client import Letta

# 1. 初始化客户端连接
client = Letta(base_url="http://localhost:8283")

# 2. 创建具备持久记忆块的有状态 Agent
agent = client.agents.create(
    name="enterprise_devops_copilot",
    model="openai/gpt-4o",
    memory={
        "blocks": [
            {
                "label": "human",
                "value": "用户是基础架构运维专家，偏好使用 Terraform 与 Kubernetes 规范配置。"
            },
            {
                "label": "persona",
                "value": "你是有状态的 DevOps 架构助手，始终遵循安全生产红线与幂等设计原则。"
            }
        ]
    }
)

# 3. 发送消息，Agent 在多轮会话中自动维护与自进化记忆
response = client.agents.messages.create(
    agent_id=agent.id,
    input="我们集群刚刚把 Redis 实例升级到了 7.2，以后请注意这一变更并记住我的生产账号是 op-prod-01。"
)

print(response.messages)
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 需要长期伴随用户的个性化助手（如个人助理、一对一导师）；
  - 复杂长周期企业级运维、长流程客服或研究型 Agent（需记住项目背景、历史决策与用户个性化规范）；
  - 多会话连续业务流水线，避免重复提供上下文产生巨额 Token 浪费。
- **潜在不足 / 局限性**：
  - 记忆块的自主更新依赖大模型的 Function Calling 精准度，弱模型（如较小的本地 7B/8B 模型）可能出现记忆误删或过度写入；
  - 依赖关系型数据库与向量插件（PostgreSQL + pgvector），需要运维配套的有状态后端。
- **个人评估结论**：**建议引入**。是当前构建企业级有状态智能体（Stateful Agent）与长周期对话助手的标杆级工程运行时方案。
