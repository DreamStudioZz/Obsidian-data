---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/huggingface/open-r1
stars: "22.8k+"
license: Apache-2.0
date_added: 2026-11-12
---

# 📦 Open-R1

> **一句话简介**：Hugging Face 官方主导发起的全开源复现与训练深度推理大模型（DeepSeek-R1 全流程复现）工业级工程基座。涵盖大规模高质量合成推理数据流水线、长思维链（Long CoT）SFT 冷启动蒸馏、基于 GRPO（Group Relative Policy Optimization，组相对策略优化）的大规模纯强化学习训练引擎，以及基于物理规则的可验证奖励机制（RLVR），为全行业提供了一套彻底透明、人人可复现的顶级推理模型端到端训练与微调配方。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: huggingface/open-r1](https://github.com/huggingface/open-r1) |
| **研发机构** | Hugging Face 团队及全球开源前沿社区 |
| **技术栈** | Python 3.10+, PyTorch, TRL (Transformer Reinforcement Learning), vLLM, DeepSpeed ZeRO-3, FSDP2, Ray, Math-Verify, SymPy |
| **许可证** | Apache-2.0 License |
| **核心机制** | 彻底去黑盒化的 DeepSeek-R1 全流程管线、零 Critic 架构的 GRPO 强化学习算法（降低 50% 显存开销）、基于规则的可验证奖励模型（RLVR）、高质量长思维链合成与蒸馏流水线、`<think>` 内部反思标签结构化约束 |
| **支持模型** | Qwen-2.5-Math-7B, Qwen-2.5-Coder-7B/32B, Llama-3.1-8B/70B, DeepSeek-R1-Distill 全系列 |
| **训练硬件** | 支持从 8 卡 H100/H800/A100 企业级集群训练，到消费级多卡 RTX 4090/5090 借助 LoRA/vLLM 极速实验 |
| **相关关联** | 关联实践: [[大模型强化推理与强化学习对齐：基于 Open-R1 与 GRPO 的可验证奖励函数设计与长思维链 (CoT) 蒸馏实战]], [[2026-11-12-AI-Digest]], [[veRL]], [[TRL]], [[vLLM]], [[DeepSpeed]], [[LLaMA-Factory]], [[Unsloth]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph DataPipeline [1. 高质量合成与推理数据集流]
        RawPrompts["原始数学/代码/逻辑问题库<br/>(MATH, GSM8K, Codeforces, LeetCode)"]
        TeacherModel["前沿巨兽模型 (DeepSeek-R1-671B / o1-preview)"]
        FilterSynthesizer["高质量 Long-CoT 样本提炼与去重"]
        RawPrompts --> TeacherModel --> FilterSynthesizer
    end

    subgraph ColdStartSFT [2. SFT 冷启动思维链初始化]
        BaseModel["通用基座大模型<br/>(如 Qwen-2.5-7B-Base)"]
        SFTTrainer["TRL SFTTrainer (带 <think> 标签的长思考数据微调)"]
        PolicyModelInit["具备基础思考结构的 Policy 模型"]
        FilterSynthesizer ==> SFTTrainer
        BaseModel ==> SFTTrainer
        SFTTrainer --> PolicyModelInit
    end

    subgraph GRPORainbowLoop [3. Open-R1 GRPO 大规模纯强化学习迭代]
        PolicyModelInit ==> PolicyNetwork["当前更新中的策略网络 (Policy $\pi_\theta$)"]
        
        subgraph GroupGeneration [高并发组采样 (vLLM 引擎)]
            Sampler["单 Query 采样 N 个候选输出<br/>($O_1, O_2, ..., O_G$)"]
        end

        subgraph RLVRRewardEngine [基于规则的绝对可验证奖励引擎 (RLVR)]
            MathJudge["数学符号等价判定器 (SymPy / LaTeX 规范化)"]
            CodeSandbox["代码测试用例沙箱执行器 (Docker / Subprocess 沙箱)"]
            FormatChecker["思维标签格式检查器 (必须闭合 <think> 标签)"]
        end

        subgraph AdvantageAndPolicyUpdate [优势归一化与策略更新]
            GroupNorm["组内奖励均值与方差归一化<br/>$A_i = \frac{R_i - \text{mean}(R)}{\text{std}(R)}$"]
            SurrogateLoss["GRPO 剪切代理损失计算 (无 Critic 价值网络)"]
            RefKLPenalty["无参参考模型 KL 散度约束 (防止模型崩溃)"]
        end

        PolicyNetwork --> Sampler
        Sampler --> MathJudge & CodeSandbox & FormatChecker
        MathJudge & CodeSandbox & FormatChecker --> GroupNorm
        GroupNorm --> SurrogateLoss
        RefKLPenalty --> SurrogateLoss
        SurrogateLoss ==>|分布式梯度更新| PolicyNetwork
    end

    subgraph SmallModelDistill [4. 蒸馏到边缘与专用小模型]
        FinalModel["超越前代巨兽的高性能 Reasoning 模型"]
        PolicyNetwork ==> FinalModel
        FinalModel -->|高质量合成推理轨迹蒸馏| EdgeModels["端侧 1.5B / 7B / 14B 推理小钢炮"]
    end
```

1. **彻底终结黑盒迷信：DeepSeek-R1 全流程训练配方的工业级透明化**：
   - 过去，构建类似于 OpenAI o1 或 DeepSeek-R1 这类具备“深度自我反思、回溯重算、Aha Moment 顿悟”的长思维链推理模型，被认为是极少数科技巨头的核心商业机密。开源社区往往只能获取模型蒸馏产物，而对其训练动力学、数据配比与奖励函数细节一无所知。
   - Open-R1 由 Hugging Face 官方工程团队带头攻关，系统性公开并实现了从“SFT 冷启动数据生成 -> 基于纯规则的零 Critic 强化学习（GRPO）探索 -> 最终向小模型蒸馏”的完整工业链路，所有代码、数据处理管道与超参数完全开源透明。
2. **零 Critic 价值网络的 GRPO 极简训练框架与 50% 显存节省**：
   - 传统大模型强化学习（PPO）架构极其庞大且脆弱，除了训练策略模型（Actor/Policy）本身外，必须同时初始化同等规模的评价模型（Critic/Value Network），导致训练显存暴增一倍，且两者的训练同步步调极其敏感、极易崩溃。
   - Open-R1 原生基于 DeepSeek 提出的 **GRPO（Group Relative Policy Optimization）** 算法，利用同一个 Prompt 在 vLLM 高并发支持下采样 $G$ 个候选回答输出，通过组内输出的相对得分标准化直接估算优势函数（Advantage），彻底移除了庞大的 Critic 网络，使企业可以用原本只能跑 SFT 的 GPU 集群直接训练顶尖强化学习推理模型。
3. **基于物理与逻辑规则的可验证奖励引擎（RLVR）杜绝幻觉作弊**：
   - 传统基于人类偏好的奖励模型（Reward Model，如 Bradley-Terry 架构）在面对严谨的数学推导与代码逻辑时，极易被模型学到的“表面客气语言风格”或“冗长但错误的论述”所欺骗（即严重的 Reward Hacking 现象）。
   - Open-R1 彻底采用 **RLVR（Reinforcement Learning with Verifiable Rewards）**，针对数学任务采用符号计算系统（SymPy / Math-Verify）判定最终答案的代数等价性；针对代码任务通过隔离容器沙箱跑真实测试用例（Pass@K）；针对格式严格校验 `<think>...</think>` 闭合性与最终答案块标记。奖励判定非 0 即 1，具备绝对的确定性与零主观偏见。
4. **思维链长度控制与推理自适应演化（Test-Time Compute）**：
   - Open-R1 在训练中观察到了极其清晰的“推理自进化曲线”：随着训练步数递增，策略模型为了最大化物理规则奖励，自发学会了拉长思维链长度，在遇到复杂难题时主动输出“Wait, let me double check...（等一下，让我再检查一遍）”、“Let's rethink this equation（重新推导这个等式）”等长链反思行为，在测试时计算（Test-Time Compute）上实现了认知跃迁。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖安装与环境准备

```bash
# 克隆官方仓库并安装核心依赖
git clone https://github.com/huggingface/open-r1.git
cd open-r1

# 基于 uv 极速安装环境（需包含 vLLM 与最新版 TRL）
uv pip install -e ".[dev]"
uv pip install vllm>=0.7.0 deepspeed ray
```

### 2. 准备可验证数学推理数据集

```bash
# Open-R1 原生集成了精选的高信噪比数学问题集
python src/open_r1/data/prep_math.py \
    --dataset "HuggingFaceH4/MATH-500" \
    --output_dir "./data/math_train"
```

### 3. 基于 GRPO 启动多卡分布式强化学习训练

编写启动脚本 `train_grpo.sh`（支持 DeepSpeed ZeRO-3 与 vLLM 加速生成）：

```bash
#!/usr/bin/env bash
# 启动 8 卡 H100 / A100 GRPO 强化学习训练
ACCELERATE_LOG_LEVEL=info accelerate launch \
    --config_file recipes/accelerate_configs/deepspeed_zero3.yaml \
    --num_processes=8 \
    src/open_r1/grpo.py \
    --model_name_or_path Qwen/Qwen2.5-Math-7B \
    --dataset_name HuggingFaceH4/MATH-500 \
    --output_dir ./output/Qwen2.5-7B-Open-R1-GRPO \
    --learning_rate 2e-6 \
    --num_generations 8 \
    --max_prompt_length 512 \
    --max_completion_length 4096 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 4 \
    --reward_funcs accuracy format \
    --use_vllm true \
    --vllm_device "cuda:7" \
    --vllm_gpu_memory_utilization 0.85 \
    --logging_steps 1 \
    --save_strategy "steps" \
    --save_steps 50 \
    --bf16 true
```

### 4. 推理测试与长思维链抽取 Python 示例

```python
from vllm import LLM, SamplingParams
import re

# 加载由 Open-R1 强化对齐后的推理模型
llm = LLM(model="./output/Qwen2.5-7B-Open-R1-GRPO", tensor_parallel_size=1)

# 设置高 Token 预算以允许模型展开长链推演与自查
sampling_params = SamplingParams(
    temperature=0.6,
    top_p=0.95,
    max_tokens=4096
)

prompt = "Solve the equation: 3^(2x+1) - 10 * 3^x + 3 = 0. Show your step-by-step thinking."
chat_prompt = f"<|im_start|>user\n{prompt}<|im_end|>\n<|im_start|>assistant\n<think>\n"

outputs = llm.generate([chat_prompt], sampling_params)
generated_text = outputs[0].outputs[0].text

print("==== 完整生成输出 ====")
print(generated_text)

# 分离内部思维过程与最终输出答案
think_match = re.search(r"^(.*?)</think>(.*)$", generated_text, re.DOTALL)
if think_match:
    internal_thought = think_match.group(1).strip()
    final_answer = think_match.group(2).strip()
    print("\n🧠 [内部长思维链反思 (CoT)]:")
    print(internal_thought[:500] + "\n...[思维链深度展开]...")
    print("\n🎯 [最终交付答案]:")
    print(final_answer)
```

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **垂类严肃逻辑/数理/代码能力跃迁**：金融量化策略验证、工业控制逻辑规则合成、复杂数学竞赛及自动化单元测试生成，需极高准确率与严密可解释性的业务系统；
  - **企业私有化 DeepSeek-R1 级小模型研发**：无需承担万亿参数模型的庞大部署开销，借助 Open-R1 配方，企业仅需几十至几百张消费级 GPU，即可在自身私有数据上训练出专注于自身业务领域、具备自主纠错能力的高精尖 7B~14B 推理专家；
  - **后训练对齐技术栈升级**：替代原本极其耗费人力标注的 RLHF/DPO 流水线，在具备明确判定标准的任务中全面转向免人工标注的物理规则自进化。
- **潜在不足与工程注意点**：
  - **采样组多样性塌缩（Group Diversity Collapse）**：若温度设得过低（如低于 0.4），同一个 Prompt 采样的 $G$ 个样本可能高度同质化（全对或全错），导致组内方差为 0，使策略梯度更新直接失效，工程中建议设置适中的 Temperature（0.6~0.8）与 Top-P（0.95）；
  - **“无用废话式”思维链膨胀**：如果缺乏对思考长度的有效正则化或奖励折扣，模型可能在训练后期学会用大量重复的车轱辘话来拖长思考文本以博取微小的长度隐式优势，需结合长度惩罚惩戒过度冗余。
- **个人评估结论**：**建议全面引入生产（Adopted）**。作为推动大模型技术由“模仿生成时代”全面迈入“自主推理验证时代”的旗帜级开源项目，Open-R1 彻底解构了大模型推理能力的底层密码，是 2026 年最具价值的模型后训练基础设施之一。
