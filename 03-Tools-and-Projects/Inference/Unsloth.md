---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/unslothai/unsloth
stars: "35k+"
license: Apache-2.0
date_added: 2026-09-24
---

# 📦 Unsloth

> **一句话简介**：开源大模型极致高效微调与对齐加速引擎，通过底层手写 Triton 内核实现 2~5 倍训练加速并缩减 70%~80% 显存占用，让消费级单卡微调 14B/32B 大模型与 GRPO 强化学习对齐成为现实。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: unslothai/unsloth](https://github.com/unslothai/unsloth) |
| **核心特点** | 手写 Triton 原语、极低显存 QLoRA/LoRA、0% 精度损失、GRPO/DPO 对齐、原生 GGUF/Ollama 导出 |
| **支持模型** | Qwen-2.5, Llama-3.1/3.2, DeepSeek-R1/V3, Mistral, Gemma 2 |
| **关联实践** | [[大模型极致后训练调优：基于 Unsloth 的显存节省与 LoRA 极速微调工程实战]], [[Ollama]], [[vLLM]] |

---

## 🚀 核心特性与技术亮点

1. **手写 Triton 原语与计算图重写**：
   - PyTorch 原生 Autograd 在反向传播时会保留海量激活值张量，导致显存极易 OOM；
   - Unsloth 手工重写了 RoPE、Cross-Entropy Loss、MLP 激活函数与注意力机制的 Triton 内核，不仅显存节省 80%，反向传播速度更是提升数倍。
2. **0% 精度损失的数学等价性**：
   - 区别于剪枝或有损量化，Unsloth 的加速来自精确的数学矩阵分解与显存重用，微调后的模型权重与 HuggingFace 标准 Transformers 完全等价。
3. **单卡平民化算力门槛**：
   - 在一张 24GB 显存的 RTX 3090 / 4090 上即可轻松完成 14B 甚至 32B 模型的 4-bit QLoRA 全量上下文微调；
   - 8GB 显卡即可微调 7B / 8B 模型。
4. **全套对齐与工业级一键交付**：
   - 原生支持 SFT、DPO（直接偏好优化）以及前沿的 **GRPO（群组相对策略优化，DeepSeek-R1 核心强化算法）**；
   - 训练完成后，支持一键保存为标准 HuggingFace 格式，或自动量化导出为 GGUF 格式，直接推送到 Ollama 运行。

---

## 🛠️ 快速上手与集成

### 1. 极速安装

```bash
pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
pip install --no-deps "trl<0.9.0" peft accelerate bitsandbytes
```

### 2. 极简 4-bit 微调与 GGUF 导出示例

```python
from unsloth import FastLanguageModel
import torch
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# 1. 极速载入 4-bit 量化基座模型
max_seq_length = 2048
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-7B-Instruct-bnb-4bit",
    max_seq_length=max_seq_length,
    load_in_4bit=True
)

# 2. 注入 Unsloth 加速的 LoRA 适配器
model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16,
    lora_dropout=0, # 官方实测 0 dropout 优化更佳且训练更快
    bias="none",
    use_gradient_checkpointing="unsloth" # 显存节省关键
)

# 3. 准备微调数据与执行训练
dataset = load_dataset("yahma/alpaca-cleaned", split="train[:1000]")
trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=max_seq_length,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        warmup_steps=5,
        max_steps=60,
        learning_rate=2e-4,
        fp16=not torch.cuda.is_bf16_supported(),
        bf16=torch.cuda.is_bf16_supported(),
        logging_steps=10,
        output_dir="outputs",
    ),
)
trainer.train()

# 4. 一键导出为 Ollama GGUF 格式
model.save_pretrained_gguf("my_finetuned_model", tokenizer, quantization_method="q4_k_m")
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **垂直领域小模型后训练（Post-Training）**：医疗、金融、法律等高私密领域，用低成本消费级算力快速蒸馏对齐私有专属模型；
  - **前沿强化对齐探索**：复现类似 DeepSeek-R1 的自我反思推理与 GRPO 规则驱动链条；
  - **本地边缘设备落地**：训练完直接通过 GGUF 部署到 Ollama / 树莓派 / Mac M 系列芯片。
- **与原生 Hugging Face PEFT 对比**：
  - 速度直接提高 2~5 倍，显存降低高达 70%，无任何功能降级，是开源微调领域的绝对第一梯队首选工具。
- **综合评估结论**：大模型高效微调的事实标准框架，强烈建议采用（Adopted）。
