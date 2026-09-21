---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agent
github: https://github.com/mem0ai/mem0
stars: "26k+"
license: Apache-2.0
date_added: 2026-09-21
---

# 📦 Mem0

> **一句话简介**：专为 AI Agent 与个性化助手打造的自适应持久化记忆层（The Memory Layer for AI），解决大模型无状态、跨会话遗忘与记忆无序膨胀难题。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: mem0ai/mem0](https://github.com/mem0ai/mem0) |
| **项目定位** | 智能体长短期记忆中间件（原 Embedchain 团队孵化） |
| **技术架构** | 语义向量检索 + BM25 关键词匹配 + 知识图谱实体链接 + 时序推理 |
| **存储后端** | Qdrant, Milvus, Chroma, PgVector, Neo4j 等 |
| **关联概念** | [[000-AI-Index]], [[2026-09-21-AI-Digest]] |

---

## 🚀 核心架构与创新亮点

普通的 RAG 向量库只能做到“被动检索已有文档”，但无法模拟人类记忆的**事实提炼、偏好演化、遗忘与时序修正**。Mem0 引入了完整的记忆生命周期管理：

1. **多层级记忆隔离（Multi-Level Hierarchy）**：
   - **User Level（用户层）**：沉淀跨会话的长期偏好、习惯与事实背景（如“用户偏好 Python 异步编程，讨厌冗长解释”）。
   - **Session Level（会话层）**：维护当前交互对话的短程上下文与任务焦点。
   - **Agent Level（智能体层）**：记录智能体自身在执行任务时的历史经验与环境状态。
2. **自适应事实抽取与冲突更新（Intelligent Extraction & Deduplication）**：
   - 当用户说“我最近把数据库从 MySQL 迁移到 PostgreSQL 了”，Mem0 会自动提取出新事实，并将旧有相关记忆标记为废弃或更新，而不是简单无脑追加。
3. **混合多路召回（Hybrid Retrieval）**：
   - 融合 Dense Vector（语义泛化）、BM25（精准标识符）与 Entity Linking（实体关系），保证精准命中记忆事实。
4. **时序感知与衰减推理（Temporal Reasoning）**：
   - 赋予记忆时间戳与关联权重，大模型能明确区分“两年前的旧方案”与“上周确定的新方案”。

---

## 🛠️ 快速上手与代码示例

### 1. 安装
```bash
pip install mem0ai
```

### 2. 在 Agent 中管理与检索用户记忆
```python
from mem0 import Memory

# 1. 初始化记忆组件
m = Memory()

# 2. 存入对话（Mem0 会自动在后台提取核心事实并结构化存储）
conversation = [
    {"role": "user", "content": "你好，我是架构师张伟，平时主要负责高并发支付系统研发。"},
    {"role": "assistant", "content": "收到，张伟老师，很高兴认识你！"},
    {"role": "user", "content": "我们团队目前技术栈已全面由 Java 迁移到 Go 语言，请在后续方案中默认采用 Go。"}
]

m.add(conversation, user_id="zhangwei_001")

# 3. 在未来的新会话中智能检索记忆
memories = m.search(
    query="帮我设计一个分布式扣库存的高性能方案",
    user_id="zhangwei_001",
    limit=3
)

for mem in memories["results"]:
    print(f"🧠 检索到记忆: {mem['memory']}")
# 输出示例: 
# 🧠 检索到记忆: 张伟主要负责高并发支付系统研发。
# 🧠 检索到记忆: 团队技术栈已全面由 Java 迁移至 Go 语言。
```

---

## 💡 工程落地建议与适用场景

- **最佳适用场景**：
  - **个性化 AI 个人助理 / Copilot**：跨会话记住用户偏好、编码习惯、系统架构约定。
  - **复杂客服与专属顾问 Agent**：避免客户在每一次新会话中重复交代个人背景。
- **性能与存储优化建议**：
  - 生产环境下避免使用默认的本地内存存储，建议接入 Qdrant 或 PgVector 作为向量存储后端，搭配 Redis 缓存高频热点用户记忆。
- **综合评估结论**：构建具备“持续陪伴感”与“深度上下文理解”的 Agent 核心组件，强烈推荐（Adopted）。
