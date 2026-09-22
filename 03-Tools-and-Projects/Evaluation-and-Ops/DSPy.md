---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/stanfordnlp/dspy
stars: "24k+"
license: MIT
date_added: 2026-09-22
---

# 📦 DSPy

> **一句话简介**：斯坦福大学 NLP 团队推出的“声明式编程取代手工提示词”框架（Declarative Self-improving Python），通过签名定义接口，并利用自动优化器（Compiler / Optimizer）针对特定评价指标算法化合成最优 Prompt 与 Few-Shot 示例。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: stanfordnlp/dspy](https://github.com/stanfordnlp/dspy) |
| **主创团队** | 斯坦福大学 NLP 实验室（Omar Khattab 等） |
| **项目定位** | 算法驱动的 LLM 提示词与流水线编译优化框架 |
| **核心机制** | 签名（Signatures）、模块（Modules）、优化器（MIPROv2 / BootstrapFewShot） |
| **关联实践** | [[告别手工调优：基于 DSPy 的提示词与 Few-Shot 自动编译优化实践]], [[Langfuse]] |

---

## 🚀 为什么说“手工调 Prompt”是死胡同？

传统基于手工调整 Prompt（Prompt Engineering）面临严峻的工程困境：
1. **模型锁定与极其脆弱**：针对 GPT-4 精心打磨了两个星期的提示词，一旦切换到 DeepSeek 或 Claude，表现立刻大幅滑坡；
2. **缺乏数学可解释性与梯度更新**：手工修修补补往往是“按下葫芦浮起瓢”，修复了一个 Edge Case 却悄悄破坏了另一个已知用例；
3. **Few-Shot 挑选全凭感觉**：人工挑选的几个示范用例往往并非模型最容易泛化的分布。

**DSPy 的编译优化哲学**：
- **代码结构与提示词分离**：开发人员只需像写函数签名一样声明输入是什么、输出是什么（`question -> answer`），具体的 Prompt 文本交由框架负责；
- **优化器作为编译器（Teleprompter / Optimizer）**：输入少量的标注样本（哪怕只有 10~20 条）和评估指标函数（如准确率、相似度），优化器（如 `MIPROv2`、`BootstrapFewShot`）会自动搜索最有效的指令措辞，并自动提炼出最佳 Few-Shot 样本组合。

---

## 🛠️ 快速上手与示例

### 1. 安装
```bash
pip install dspy
```

### 2. 构建并自动优化一个推理问答模块
```python
import dspy

# 1. 配置模型后端
lm = dspy.LM("openai/gpt-4o-mini")
dspy.configure(lm=lm)

# 2. 声明签名（定义输入输出契约）
class MultiHopQA(dspy.Signature):
    """根据问题拆解复杂推理步骤并给出准确简短的答案。"""
    context = dspy.InputField(desc="相关参考事实")
    question = dspy.InputField(desc="待解答的问题")
    rationale = dspy.OutputField(desc="逐步推理过程")
    answer = dspy.OutputField(desc="最终简短结论")

# 3. 组装为模块
class RAGPipeline(dspy.Module):
    def __init__(self):
        super().__init__()
        self.generate_answer = dspy.ChainOfThought(MultiHopQA)

    def forward(self, context, question):
        return self.generate_answer(context=context, question=question)

# 4. 自动编译优化（BootstrapFewShot 自动提炼高质量 Few-Shot）
from dspy.teleprompt import BootstrapFewShot

def validation_metric(example, pred, trace=None):
    # 评价指标：输出答案包含标准答案
    return example.answer.lower() in pred.answer.lower()

# 准备 10~20 条小型训练集
trainset = [
    dspy.Example(context="巴黎是法国首都，埃菲尔铁塔位于巴黎。", question="埃菲尔铁塔在哪个国家？", answer="法国").with_inputs("context", "question")
]

teleprompter = BootstrapFewShot(metric=validation_metric, max_bootstrapped_demos=3)
compiled_pipeline = teleprompter.compile(RAGPipeline(), trainset=trainset)

# 5. 调用已自动编译最佳 Prompt 的流水线
result = compiled_pipeline(context="李白出生于碎叶城，是唐代著名浪漫主义诗人。", question="李白是哪个朝代的？")
print(result.answer)
```

---

## 💡 工程落地建议与选型评估

- **适用场景**：
  - **复杂多跳 RAG 与长链路推理**：手工写 Prompt 极难平衡多个子步骤时；
  - **跨模型平滑迁移**：当团队需要将业务从高成本商业模型迁移到开源小模型时，使用 DSPy 重新针对小模型“编译（Compile）”一次，效果往往能跨越式提升。
- **与 LangChain / LlamaIndex 的关系**：
  - DSPy 不是 LangChain 的对立编排框架，而是它的“优化内核”。你可以用 LangChain 做工具连接，用 DSPy 优化核心提示词与决策模块。
- **综合评估结论**：大模型流水线算法化调优的先锋利器，强烈建议团队掌握（Adopted）。
