---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/explodinggradients/ragas
stars: "11k+"
license: Apache-2.0
date_added: 2026-09-30
---

# 📦 Ragas

> **一句话简介**：业界首屈一指的 RAG 与 Agent 管道量化评估与合成测试集生成框架，通过解耦“检索（Retrieval）”与“生成（Generation）”两大阶段，提供无需昂贵人工标注的忠实度、回答相关度、上下文精确度与召回率等端到端核心度量指标。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: explodinggradients/ragas](https://github.com/explodinggradients/ragas) |
| **核心特点** | RAG 四维核心指标体系、无人工标注自动化评测、基于演化算法的多难度合成测试集生成器、Agent 多工具执行轨迹评估 |
| **技术栈** | Python / LangChain / LlamaIndex / Pydantic / Datasets |
| **关联实践** | [[RAG 体系化量化评估：基于 Ragas 的核心四维指标与自动化合成测试集实战]], [[大模型应用质量工程：基于 DeepEval 的端到端指标评测与 CI-CD 自动化门禁实战]], [[Agent 生产级可观测性与评估：基于 Langfuse 的全链路追踪实践]] |

---

## 🚀 核心特性与技术亮点

1. **RAG 经典四维核心指标解耦评估**：
   - 彻底打破“整体黑盒打分”的盲目性，将 RAG 管道精细拆分为两组核心正交维度：
     - **检索层评估**：
       - `Context Precision`（上下文精确度）：检索出的 Chunk 中，真正与问题相关的优质内容是否排在前面（衡量 Rerank 质量）；
       - `Context Recall`（上下文召回率）：回答问题所需的关键事实是否已被检索上下文全量覆盖（衡量 Embedding 召回能力）。
     - **生成层评估**：
       - `Faithfulness`（忠实度 / 抗幻觉度）：生成的回答能否由检索到的上下文充分推导（检测无中生有的幻觉）；
       - `Answer Relevance`（回答相关度）：生成的回答是否切题，是否存在废话或顾左右而言他。
2. **基于演化算法（Evolutionary Approach）的合成测试集生成（Testset Generator）**：
   - 解决企业没有现成标注测试集（Ground Truth）的冷启动痛点；
   - 能够输入一批企业原始 Markdown/PDF 文档，自动合成具有挑战性的多层次测试样本：简单事实问答（Simple）、多跳逻辑推理（Reasoning）、跨段落关联（Multi-context）与条件限定问答（Conditional）。
3. **Agent 多步工具调用与多轮对话评估支持**：
   - 全新版本全面拓展至 Agentic RAG 范式，支持度量 Agent 工具选择准确率（Tool Call Accuracy）、目标达成度（Goal Completion）与多轮会话状态一致性。
4. **与 CI/CD 及可观测性工具链无缝集成**：
   - 提供 Hugging Face `datasets` 标准格式输入输出，能无缝对接 Langfuse、Phoenix 等可观测追踪平台，在发布 PR 前自动产出量化质量对比报表。

---

## 🛠️ 快速上手与集成

### 1. 安装 Ragas

```bash
pip install ragas langchain-openai
```

### 2. 核心量化指标评测代码

```python
import os
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
)

os.environ["OPENAI_API_KEY"] = "sk-..."

# 构建评估数据样本（包含问题、生成的回答、检索到的上下文、标准答案）
data = {
    "question": [
        "企业级向量数据库中标量量化有什么作用？",
    ],
    "answer": [
        "标量量化通过将 float32 高维向量转换为 int8 整数向量，能够缩减大约 75% 的内存占用，同时保留 99% 的检索精度。"
    ],
    "contexts": [
        [
            "在向量检索中，标量量化（Scalar Quantization）将 32 位浮点数映射为 8 位整数，能将内存占用压缩至 25%，并保持 99% 以上的召回率。"
        ]
    ],
    "ground_truth": [
        "标量量化将 float32 压缩为 int8，减少 75% 内存占用，同时保证高检索准确性。"
    ]
}

dataset = Dataset.from_dict(data)

# 执行四维全量自动化评估
score = evaluate(
    dataset=dataset,
    metrics=[
        faithfulness,
        answer_relevancy,
        context_precision,
        context_recall,
    ],
)

# 打印各项指标得分（0.0 ~ 1.0）
print(score.to_pandas()[["faithfulness", "answer_relevancy", "context_precision", "context_recall"]])
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 研发团队进行 RAG 算法升级（更换 Embedding 模型、调整分块策略、引入 Rerank）时的 A/B 测试基准度量；
  - 缺少人工标注时，利用原始文档一键生成几百条带标准答案的高质量单元测试集；
  - 上线前流水线自动化质量门禁（如：CI 自动化校验 `faithfulness > 0.90` 才能允许发布）。
- **潜在不足 / 局限性**：
  - 评估过程依赖 LLM-as-a-Judge，批量评测大型数据集时会消耗一定量的评测模型 Token；
  - 评估模型的推理能力直接影响评分准确性，建议使用 GPT-4o 或同级别强模型作为评估裁判。
- **评估结论**：**极力推荐引入（Adopted）**。量化评估是 RAG 从“玩具 Demo”迈向“企业级高可靠交付”的唯一科学指南针。
