---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/hiyouga/LLaMA-Factory
stars: "40k+"
license: Apache-2.0
date_added: 2026-10-04
---

# 📦 LLaMA-Factory

> **一句话简介**：工业级全功能大模型高效微调与强化学习对齐（RLHF/DPO）一站式工程框架，统一覆盖 100+ 主流开源语言与多模态大模型，集成了 LoRA/QLoRA、DeepSpeed、FlashAttention-2、零代码 WebUI 与完整工业落地部署导出链路。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: hiyouga/LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) |
| **核心特点** | 100+ 模型即插即用统一适配、全阶段训练流水线（预训练/SFT/DPO/ORPO/PPO）、极致显存压榨（QLoRA/ZeRO-3）、零代码 LLaMA Board WebUI、一键权重合并与 vLLM/GGUF 导出 |
| **技术栈** | Python / PyTorch / Hugging Face Transformers / DeepSpeed / PEFT / FlashAttention-2 / Gradio |
| **关联实践** | [[大模型高效微调与强化学习对齐：基于 LLaMA-Factory 的多模型全流程训练实战]], [[大模型极致后训练调优：基于 Unsloth 的显存节省与 LoRA 极速微调工程实战]], [[端侧高吞吐低延迟推理：基于 llama.cpp 的 GGUF 极限混合量化与投机解码工程实战]] |

---

## 🚀 核心特性与技术亮点

1. **统一的全栈训练闭环（All-in-One Training Pipeline）**：
   - 覆盖从**预训练（Continual Pre-Training）**、**有监督微调（SFT）**、**奖励模型构建（Reward Modeling）**，到**偏好对齐（DPO / ORPO / SimPO / KTO / PPO）**的全生命周期；
   - 原生支持包括 LLaMA-3、Qwen-2.5、DeepSeek-V2/V3、Mistral、Gemma-2、ChatGLM-4 等业界所有主流开源架构及主流视觉多模态大模型（VLM）。
2. **丰富的参数高效微调方法（PEFT）与极限显存优化**：
   - 深度集成 **LoRA、QLoRA（4-bit BitsAndBytes）、DoRA（权重解耦 LoRA）、GaLore 与 BAdam**；
   - 支持 **FlashAttention-2**、**Unsloth 算子内核加速**以及 **DeepSpeed ZeRO-2 / ZeRO-3 Offload**，仅需一张 24GB 消费级显卡（如 RTX 3090/4090）即可轻松微调 70B 级别大模型。
3. **低门槛交互式界面与可重现的声明式配置**：
   - 内置 **LLaMA Board（基于 Gradio 的可视化 WebUI）**，支持一键上传数据集预览、超参数滑动调整、实时训练 Loss 曲线打点及一键推理测试；
   - 全面支持生产级声明式 **YAML 配置文件**，便于纳入 Git 版本控制与 CI/CD 自动化流水线。
4. **无缝衔接生产级推理生态**：
   - 支持一键将 LoRA 适配器权重无损合入基座权重，生成标准 Hugging Face 权重；
   - 支持将模型直接转为 GGUF 格式供 `llama.cpp` / `Ollama` 运行，或通过内置的高性能 `vLLM` 引擎直接拉起 OpenAI 兼容接口服务。

---

## 🛠️ 快速上手与配置

### 1. 环境安装
```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics,deepspeed,bitsandbytes]"
```

### 2. 声明式 YAML 启动 QLoRA 微调（以 Qwen-2.5 为例）
创建配置文件 `train_qwen_qlora.yaml`：
```yaml
### 模型与量化设置
model_name_or_path: Qwen/Qwen2.5-7B-Instruct
quantization_bit: 4

### 方法与适配器
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### 数据集与超参数
dataset: identity,alpaca_en_demo
template: qwen
cutoff_len: 2048
max_samples: 1000
overwrite_output_dir: true
output_dir: ./saves/qwen2.5-7b/lora/sft

### 训练策略
learning_rate: 1.0e-4
num_train_epochs: 3.0
per_device_train_batch_size: 2
gradient_accumulation_steps: 4
lr_scheduler_type: cosine
warmup_ratio: 0.1
bf16: true
logging_steps: 10
save_steps: 100
plot_loss: true
```

启动训练命令：
```bash
llamafactory-cli train train_qwen_qlora.yaml
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **企业专有领域模型孵化**：结合行业数据对开源基座进行专业知识注入与指令对齐；
  - **偏好对齐与安全性改造**：利用 DPO / ORPO 消除有害输出或使模型风格完全贴合业务标准；
  - **中小团队与算力受限研发**：无需手写分布式并行代码，通过声明式配置最大化单卡/多卡算力性价比。
- **潜在不足 / 局限性**：
  - 封装层次较为严密，对需要深度修改底层 Transformer Forward 计算图或注意力机制底层研究的极客来说可能不够自由。
- **个人评估结论**：**建议全面采用（status/adopted）**。目前业界最成熟、支持模型最多、工程化完善度最高的一站式微调对齐利器。
