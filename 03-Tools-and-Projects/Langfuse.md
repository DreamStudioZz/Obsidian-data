---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation
github: https://github.com/langfuse/langfuse
stars: "14k+"
license: MIT
date_added: 2026-09-21
---

# 📦 Langfuse

> **一句话简介**：开源大模型工程平台，专为生产级 AI Agent 与 LLM 应用提供全链路可观测性（Tracing）、自动化评估（Evals）、Prompt 版本管理与成本延迟监控。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: langfuse/langfuse](https://github.com/langfuse/langfuse) |
| **部署方式** | 开源私有化部署（Docker Compose / Helm）或官方 Managed Cloud |
| **核心协议** | 原生兼容 OpenTelemetry (OTel)，支持 API / SDK 异步上报 |
| **生态集成** | LiteLLM, PydanticAI, LangChain, LlamaIndex, OpenAI, Anthropic |
| **关联实践** | [[Agent 生产级可观测性与评估：基于 Langfuse 的全链路追踪实践]], [[LiteLLM]] |

---

## 🚀 为什么说可观测性是 Agent 迈向生产的生命线？

原型阶段的 Demo 往往难以应对复杂的生产场景：
- **Agent 陷入死循环或调用错误工具时，黑盒调用无法定位原因；**
- **无法统计多步骤复杂 Chain 的 Token 消耗与计费归属；**
- **模型微调或 Prompt 迭代后，缺乏可靠的自动化评测集来验证是否产生能力退化。**

**Langfuse 的核心价值**：
1. **全链路树状 Tracing（Hierarchical Traces）**：
   - 完整捕获从最顶层的用户请求，到中间的任务拆解规划（Thought/Reasoning）、检索阶段（RAG Context）、工具执行（Tool Call & Params）、直至最终输出的完整调用树。
2. **多维度成本与延迟监控（Cost & Latency Breakdown）**：
   - 内置主流模型计费规则字典，自动计算每一步 Span 的精确 Token 与美金费用，精准定位耗时最高的木桶短板。
3. **自动化评估与打分体系（Evals & LLM-as-a-judge）**：
   - 支持在线生产流量抽样评估、离线黄金测试集批量回归，结合用户正负反馈（👍/👎）与大模型打分，量化应用质量。
4. **Prompt 版本生命周期管理（Prompt CMS）**：
   - 将 Prompt 模版从业务硬编码中剥离，支持线上版本热更新、A/B 分流测试与快速回滚。

---

## 🛠️ 快速上手与集成

### 1. 本地一键启动 Langfuse 服务
```bash
# 通过官方 Docker Compose 极速启动
git clone https://github.com/langfuse/langfuse.git
cd langfuse
docker compose up -d
# 访问 http://localhost:3000 进入 Web 控制台
```

### 2. Python 极速接入 Tracing
```python
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context

langfuse = Langfuse()

# 1. 追踪外层 Agent 会话
@observe()
def handle_user_query(user_id: str, query: str):
    # 丰富追踪元数据
    langfuse_context.update_current_trace(
        name="customer_support_agent",
        user_id=user_id,
        tags=["production", "v2"]
    )
    
    # 模拟检索阶段
    context = retrieve_knowledge(query)
    # 模拟大模型推理与工具调用
    response = call_llm(context, query)
    return response

# 2. 追踪嵌套子任务
@observe()
def retrieve_knowledge(query: str):
    return "Relevant context docs..."

@observe(as_type="generation")
def call_llm(context: str, query: str):
    # 此处自动记录 Prompt、Model、Token 使用量与延迟
    return "Generated Answer."
```

---

## 💡 工程落地评估与建议

- **生产适用场景**：
  - **企业级 AI 应用必备基础设施**：对数据合规、隐私安全有要求的团队，Langfuse 纯开源私有部署是目前最佳方案。
  - **复杂多代理协调（Multi-Agent System）**：排查死循环、超长上下文截断和偶发性工具异常的神器。
- **与 LangSmith 对比**：
  - LangSmith 深度绑定 LangChain 商业云服务，Langfuse 完全开源、支持私有化自建、原生兼容 OpenTelemetry，架构更加开放解耦。
- **综合评估结论**：生产环境强烈建议作为标准化可观测底座采纳（Adopted）。
