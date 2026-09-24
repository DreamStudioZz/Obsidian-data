---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/confident-ai/deepeval
stars: "10k+"
license: Apache-2.0
date_added: 2026-09-24
---

# 📦 DeepEval

> **一句话简介**：被称为“大模型时代的 Pytest”的企业级开源 LLM 评估与单元测试框架，提供基于 G-Eval 的忠实度、回答相关度与幻觉量化指标，支持自动化合成测试集与 CI/CD 质量门禁。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: confident-ai/deepeval](https://github.com/confident-ai/deepeval) |
| **核心特点** | 声明式 LLM 单元测试（Pytest 兼容）、14+ 核心量化指标、合成测试数据生成、CI/CD 自动化阻断门禁 |
| **技术栈** | Python / Pytest / Pydantic / OTel |
| **关联实践** | [[大模型应用质量工程：基于 DeepEval 的端到端指标评测与 CI/CD 自动化门禁实战]], [[Agent 生产级可观测性与评估：基于 Langfuse 的全链路追踪实践]], [[DSPy]] |

---

## 🚀 核心特性与技术亮点

1. **Pytest 原生契约式单元测试**：
   - 将主观、模糊的大模型生成结果转化为工业软件工程中的确定性断言（`assert_test`）。
   - 一行命令 `deepeval test run test_rag.py` 即可在本地或流水线中批量执行，不达标测试用例直接高亮标红并阻断合并。
2. **研究级完备量化指标库（14+ Production Metrics）**：
   - **RAG 三联指标**：忠实度（Faithfulness，是否脱离检索上下文胡编）、回答相关性（Answer Relevancy）、上下文精确度与召回率（Contextual Precision & Recall）；
   - **G-Eval 框架**：支持开发者用自然语言定义专属评价准则（如“专业技术语调”、“简洁度”、“业务合规性”），自动分解为步骤化加权打分；
   - **红队安全性**：幻觉检测（Hallucination）、毒性（Toxicity）、偏见（Bias）。
3. **合成评估数据集生成器（Synthetic Test Data Generation）**：
   - 解决企业落地最缺标注文档的困境。DeepEval 能解析原始业务文档，利用演化算法自动反向构造出数百个具有真实挑战性的（问题、预期答案、召回切片）测试集。
4. **CI/CD 流水线自动化门禁**：
   - 彻底防止“修改了一句 System Prompt，修好了 A 问题却暗中弄坏了 B/C 问题”的**隐形退化（Silent Regression）**问题。

---

## 🛠️ 快速上手与集成

### 1. 安装框架

```bash
pip install deepeval
```

### 2. 编写 RAG 单元测试用例 (`test_rag_pipeline.py`)

```python
import pytest
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.metrics import FaithfulnessMetric, AnswerRelevancyMetric

def test_enterprise_rag_faithfulness():
    # 模拟 RAG 系统的实际输出与检索上下文
    test_case = LLMTestCase(
        input="公司的年假政策是怎样的？",
        actual_output="员工入职满一年享有 5 天带薪年假，满十年享有 10 天带薪年假。",
        retrieval_context=[
            "员工手册第4章：全职员工入职满一年可享受5个工作日带薪年休假；工作年限满十年者，年休假增至10个工作日。"
        ]
    )

    # 1. 忠实度检测（严防无中生有）
    faithfulness_metric = FaithfulnessMetric(
        threshold=0.8,
        model="gpt-4o-mini",
        include_reason=True
    )
    
    # 2. 回答相关性检测
    relevancy_metric = AnswerRelevancyMetric(
        threshold=0.7,
        model="gpt-4o-mini"
    )

    # 断言执行，低于阈值将抛出 AssertionError 并给出扣分推导过程
    assert_test(test_case, [faithfulness_metric, relevancy_metric])
```

### 3. 执行测试套件

```bash
deepeval test run test_rag_pipeline.py
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **Prompt 与模型版本迭代的回归测试**：在 PR 提交触发 GitHub Actions 时自动运行，确保基准能力不倒退；
  - **RAG 检索召回参数调优**：对比不同切块大小（Chunk Size）、Top-K 和 Reranker 模型对最终忠实度与相关性的量化得分。
- **与 Langfuse 观测的区别**：
  - Langfuse 侧重于**线上生产运行态（Runtime）**的全链路 Trace 追踪与真实用户打标；
  - DeepEval 侧重于**线下研发测试态（Testing/CI-CD）**的门禁断言与前置质检，两者是相辅相成的黄金搭档。
- **综合评估结论**：大模型工程化落地必备的质检验收框架，强烈建议采用（Adopted）。
