---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/comet-ml/opik
stars: "6k+"
license: Apache-2.0
date_added: 2026-10-30
---

# 📦 Opik

> **一句话简介**：由 Comet 团队打造的高性能生产级开源大模型与 Agent 端到端可观测性、自动化评测与数据飞轮平台。专为解决生产环境大模型应用“黑盒失控、质量漂移、缺乏真实测试集”三大顽疾而设计，提供无缝全链路 Trace 追踪、启发式规则与 LLM-as-a-judge 在线实时打分、生产故障样例一键沉淀为回归测试集，以及 CI/CD 自动化对比实验矩阵。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: comet-ml/opik](https://github.com/comet-ml/opik) |
| **研发社区** | Comet ML 开源团队 |
| **技术栈** | Python, TypeScript, React, Docker, OpenTelemetry, ClickHouse, FastAPI |
| **许可证** | Apache-2.0 License |
| **核心特性** | 极简 `@track` 装饰器全链路 Span 追踪、生产级双轨在线评测（规则 + LLM-as-a-judge）、幻觉与事实一致性检测、生产失败流量一键沉淀测试集（Feedback Loop）、提示词 Playground 与实验对比矩阵（Experiments & Prompts）、完全支持私有化 Docker 一键自建 |
| **相关关联** | 关联实践: [[生产级 Agent 全链路追踪与自动化评测闭环：基于 Opik 的在线质检与评测飞轮实战]], [[2026-10-30-AI-Digest]], [[Langfuse]], [[Phoenix]], [[AgentOps]], [[DeepEval]], [[Promptfoo]], [[Ragas]] |

---

## 🚀 核心特性与架构亮点

### 1. 生产级全链路深度追踪（Traces & Spans）
在复杂的复合智能体（Agent）与多跳 RAG 流水线中，单次用户交互往往触发数十次底层操作（重写 Prompt、多次向量数据库检索、重排 Rerank、多次模型思考推理与外部工具调用）：
- **零侵入式追踪注入**：Opik 提供优雅的 `@track` Python 装饰器与 OpenTelemetry 兼容标准，开发者只需在核心函数上添加注解，即可自动记录执行耗时、输入输出、Token 消耗以及嵌套调用树；
- **生态无缝对接**：原生集成 LangChain、LlamaIndex、OpenAI SDK、LiteLLM 以及主流自主 Agent 架构，秒级实现从客户端到模型端全生命周期的调用拓扑透视。

### 2. 双轨在线实时打分体系（Online Evaluation Pipeline）
传统的离线测试无法反映真实线上多样化流量的质量波动，Opik 构建了严密的高并发双轨质检机制：
- **轻量确定性规则评测（Heuristic Metrics）**：极速检测模型输出的 JSON 结构合法性、关键词命中率、P99 响应延迟与 Token 消耗上限，保障系统基本运行 SLA；
- **LLM-as-a-judge 深度认知评测**：内置工业级度量指标（如回答相关性 `AnswerRelevance`、幻觉评分 `Hallucination`、适格性检查 `Moderation` 与上下文召回率 `ContextPrecision`），以异步非阻塞模式由裁判模型在线打分，精准捕捉语义劣化与虚假捏造。

### 3. “生产异常流量 -> 回归测试集”的一键闭环飞轮（Data Flywheel）
以往当线上 Agent 发生答非所问、工具调用死循环或格式崩溃时，运维人员往往只能截屏或翻查零散日志，难以转化为工程资产：
- **故障一键沉淀为 Dataset**：在 Opik 控制台中，工程师与业务专家只需点击一次，即可将任何发生低评分、用户点击踩（Thumbs-down）或工具异常的生产 Trace 直接加入命名数据集（Datasets）；
- **CI/CD 自动化对比实验（Experiment Matrix）**：当开发团队尝试调整 System Prompt 或升级底层大模型底座时，可通过 Opik SDK 在该数据集上批量跑测新版本，生成精确的指标雷达图与差异对比报告，彻底杜绝“盲改 Prompt 导致暗度陈仓式质量倒退”。

### 4. 极致现代的技术底座与开箱即用私有化
- **高性能 OLAP 存储引擎**：后端采用 ClickHouse 作为主数据底座，具备单节点每秒摄取数十万条 Trace 的吞吐能力，且提供亚秒级的复杂聚合检索分析；
- **现代化 UI 与 Prompt 管理中枢**：提供直观的调用瀑布流（Waterfall）、Prompt 协同版本控制工作台与测试操练场（Playground），全面赋能算法工程师与产品运营。

---

## 🛠️ 快速上手与典型代码示例

### 1. 本地私有化单命令部署与客户端安装

```bash
# 1. 采用 Docker Compose 本地一键启动 Opik 全栈服务 (包含 ClickHouse, Redis 与 Web UI)
docker compose -f https://raw.githubusercontent.com/comet-ml/opik/main/docker-compose.yml up -d

# 2. 安装 Python 核心 SDK
pip install opik
```

### 2. 生产级全链路追踪与在线实时质检打分实战

```python
import os
import opik
from opik import track
from opik.evaluation.metrics import Hallucination, AnswerRelevance

# 1. 配置 Opik 本地私有化服务地址（默认端口 5173）
os.environ["OPIK_URL_OVERRIDE"] = "http://localhost:5173/api"
os.environ["OPIK_PROJECT_NAME"] = "production-agent-monitoring"

# 2. 初始化核心评测指标
hallucination_metric = Hallucination()
relevance_metric = AnswerRelevance()

# 3. 追踪底层 RAG 检索子节点
@track(name="rag_knowledge_retrieval")
def retrieve_enterprise_context(query: str) -> list[str]:
    # 模拟从知识库检索相关的上下文切片
    return [
        "企业差旅政策规定：国内出差标准中，一线城市住宿费标准上限为每日 550 元。",
        "餐饮补贴为定额包干标准，无论实际消费均为每日 120 元，需随差旅报告一同申报。"
    ]

# 4. 追踪端到端智能体生成主流程，并附加在线评分
@track(name="enterprise_travel_agent", capture_input=True)
def run_travel_assistant(user_prompt: str) -> str:
    # 步骤一：检索上下文
    contexts = retrieve_enterprise_context(user_prompt)
    context_str = "\n".join(contexts)
    
    # 步骤二：模拟大模型生成回答
    # 实际场景可接入 OpenAI、vLLM、LiteLLM 或 DeepSeek
    generated_answer = "根据公司差旅政策，一线城市住宿费上限为 550 元/天，餐饮补贴为每日 120 元定额包干。"
    
    # 步骤三：在线计算质量评分并关联至当前 Trace
    relevance_score = relevance_metric.score(
        input=user_prompt,
        output=generated_answer,
        context=contexts
    )
    
    hallucination_score = hallucination_metric.score(
        input=user_prompt,
        output=generated_answer,
        context=contexts
    )
    
    # 将评分绑定至当前上下文 Span
    opik.log_metric(name="answer_relevance", value=relevance_score.value)
    opik.log_metric(name="hallucination_risk", value=hallucination_score.value)
    
    return generated_answer

# 5. 执行调用，调用链与分数自动异步投递至 Opik 控制台
if __name__ == "__main__":
    prompt = "请问我去上海出差，住宿和餐费每天分别能报销多少？"
    response = run_travel_assistant(prompt)
    print("模型输出:", response)
    print("全链路 Trace 及在线质检评分已自动同步至 Opik Dashboard!")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **复杂多步骤 Agent 系统的全景白盒化**：清晰定位由于哪一次工具调用传参错误或哪一个 RAG 切片不匹配引发的最终回答崩溃；
  - **生产环境持续在线质检与合规审计**：对企业级核心应用持续监控幻觉率与回答相关度，自动对低分异常进行预警并截流；
  - **大模型应用研发的迭代飞轮**：将生产线上的脏数据转化为黄金评测集，用数据驱动 Prompt 调优与模型选型升级。
- **与同类工具对比（vs Langfuse / Arize Phoenix）**：
  - 相比于 **Langfuse**：Opik 具备更轻便的 Python 原生评测流水线与更聚焦的 LLM-as-a-judge 内置指标，界面交互与数据集迭代飞轮更加紧凑顺滑；
  - 相比于 **Phoenix**：Opik 对应用工程团队更加友好，界面现代直观，Prompt 实验对比工作流（Experiments）开箱体验极佳。
- **个人评估结论**：**强烈推荐引入 (status/adopted)**。现代企业构建严肃、可信、持续进化的工业级大模型应用的必备工程基石。
