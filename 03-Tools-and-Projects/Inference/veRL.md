---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/volcengine/verl
stars: "8.5k+"
license: Apache-2.0
date_added: 2026-11-04
---

# 📦 veRL (Volcano Engine Reinforcement Learning for LLM)

> **一句话简介**：由字节跳动火山引擎（Volcano Engine）联合学术界开源的高性能分布式大模型强化学习后训练框架（Post-Training RL Framework）。首创 **HybridFlow 编程模型**，将生成采样（Rollout）与训练反向传播（Actor/Critic Update）的资源拓扑与数据流彻底解耦，原生支持大规模 GRPO、PPO 与 REINFORCE++，成为全球开源复现 DeepSeek-R1 与构建长思维链推理涌现模型的事实标准底层引擎。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: volcengine/verl](https://github.com/volcengine/verl) |
| **研发机构** | 字节跳动火山引擎团队（ByteDance Volcano Engine）与开源社区 |
| **技术栈** | Python, PyTorch, Ray, vLLM, SGLang, Megatron-LM, FSDP, FlashAttention |
| **许可证** | Apache-2.0 License |
| **核心算法** | GRPO (Group Relative Policy Optimization), PPO, REINFORCE++, DPO, KTO, RLVR |
| **核心特性** | HybridFlow 异构流式解耦架构、Actor/Rollout 分布式异步权重乒乓同步、3D 并行 (TP/PP/DP/SP) 弹性组装、vLLM/SGLang 连续批处理极速采样、可验证奖励函数 (RLVR) 自动化校验 |
| **相关关联** | 关联实践: [[大模型强化学习与推理涌现架构：基于 veRL 的 HybridFlow 分布式解耦与大规模 GRPO 训练实战]], [[2026-11-04-AI-Digest]], [[TRL]], [[vLLM]], [[SGLang]], [[LLaMA-Factory]], [[Unsloth]] |

---

## 🚀 核心特性与架构亮点

### 1. HybridFlow 异构解耦编程模型（Decoupled Computation & Dataflow）
在大模型强化学习（RLHF / RLAIF / RLVR）生命周期中，存在两个计算特征截然相反的核心阶段：
- **Rollout 阶段（自回归生成）**：受限于显存带宽，需要采用小批次、高吞吐的推理加速引擎（如 vLLM 或 SGLang）以及 PagedAttention / RadixAttention 维护长上下文 KV Cache；
- **Train 阶段（前向反向与梯度更新）**：计算密集型（Compute-bound），需要采用 Megatron-LM 3D 并行或 PyTorch FSDP 进行显存参数切分、激活重计算与优化器更新。
传统 RL 框架（如早期的 DeepSpeed-Chat 或标准 TRL）强行将训练引擎与推理逻辑绑定在同一批 GPU 显存拓扑上，导致显存碎片化严重、推理吞吐低、甚至频繁遭遇 OOM。veRL 首创的 **HybridFlow** 允许工程师以声明式 DAG 定义 Actor、Critic、Reference Model 与 Rollout Workers 的独立资源组（Ray WorkerGroups），计算与生成资源自由配比，互不干扰。

### 2. 原生高效 GRPO 架构：彻底剔除 Critic 模型
- 传统 PPO 算法需要维护与 Actor 同等参数规模的 Critic（价值模型），不仅吞吐减半，且额外占用近 50% 显存；
- veRL 原生深度支持 **GRPO（Group Relative Policy Optimization）** 算法：针对同一 Prompt 并发采样一组候选回答（如 $G=8$），通过可验证规则（代码测试用例通过率、数学公式断言、结构化标记）打分，直接利用组内候选解的均值和标准差归一化出相对优势值（Advantage），在完全零 Critic 显存开销的前提下稳定实现思维链自进化。

### 3. 权重乒乓广播（Weight & State Ping-Pong）与高吞吐显存复用
- veRL 基于 NCCL 实现了亚秒级的主权重广播通道：Actor 训练完成一个 Step 后，最新参数通过高带宽环形通信瞬间同步给各个分布式 Rollout Workers；
- 在 Rollout 阶段，训练优化器状态（Optimizer States）自动卸载至 CPU 内存或分片存储，释放全部 GPU 显存用于大批次 KV Cache；在 Train 阶段，推理显存快速释放，训练算子拉满算力，实现 GPU 模型算力利用率（MFU）大幅攀升。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

```bash
# 创建独立虚拟环境并安装核心依赖
conda create -n verl python=3.10 -y
conda activate verl

# 安装支持 FlashAttention 与 CUDA 12 的 PyTorch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 安装 veRL 核心与分布式推理加速后端 (vLLM)
pip install verl[vllm] flash-attn --no-build-isolation
```

### 2. 基于 Ray 与 GRPO 启动长思维链自进化训练

```python
import os
import ray
from verl import DataProto
from verl.trainer.ppo.ray_trainer import RayPPOTrainer
from verl.utils.reward_score import math_compute_score

# 1. 初始化 Ray 分布式集群上下文
ray.init(ignore_reinit_error=True)

# 2. 定义可验证规则奖励函数 (Rule-based Verifiable Reward)
def reward_fn(data_item: DataProto) -> float:
    """针对数学竞赛或代码生成任务的可验证规则奖励函数"""
    prediction = data_item.non_tensor_batch["pred_str"]
    ground_truth = data_item.non_tensor_batch["ground_truth"]
    
    # 提取 <think>...</think> 之后的最终答案
    score = math_compute_score(prediction, ground_truth)
    
    # 格式规范惩罚/奖励机制：鼓励长思维链并严格闭合思考标签
    if "<think>" in prediction and "</think>" in prediction:
        score += 0.2
    else:
        score -= 0.5
    return score

# 3. 声明式配置 GRPO 训练流水线参数
config = {
    "data": {
        "train_files": ["gsm8k_train.parquet"],
        "val_files": ["gsm8k_test.parquet"],
        "train_batch_size": 256,
        "max_prompt_length": 1024,
        "max_response_length": 4096, # 支持超长 CoT 思考链
    },
    "actor_rollout_ref": {
        "hybrid_engine": True,
        "model": {
            "path": "Qwen/Qwen2.5-7B-Instruct",
            "enable_gradient_checkpointing": True,
        },
        "actor": {
            "strategy": "fsdp", # 采用 PyTorch 原生 FSDP 训练
            "lr": 1e-6,
            "ppo_mini_batch_size": 64,
            "use_kl_loss": True,
            "kl_loss_coef": 0.001,
        },
        "rollout": {
            "name": "vllm",      # 采用 vLLM 高速采样引擎
            "temperature": 0.8,
            "n": 8,              # GRPO 每组采样 8 个候选解
            "gpu_memory_utilization": 0.6,
        },
    },
    "algorithm": {
        "adv_estimator": "grpo", # 指定使用 GRPO 优势估计器
        "norm_adv_by_group": True,
    },
    "trainer": {
        "total_epochs": 5,
        "save_freq": 50,
        "test_freq": 10,
    }
}

# 4. 实例化 RayPPOTrainer 并启动大规模分布式自对齐训练
if __name__ == "__main__":
    trainer = RayPPOTrainer(config=config, reward_fn=reward_fn)
    trainer.init_workers()
    trainer.fit()
    print("✅ GRPO 强化学习推理自进化训练圆满完成！")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **复杂数学与代码推理自进化（Reasoning Emergence）**：训练类似 OpenAI o1 或 DeepSeek-R1 的具备自主试错、回溯反思与长 CoT 推理能力的大模型；
  - **基于规则的自动化强化学习（RLVR）**：存在明确客观校验标准（编译器执行通过率、SQL 正确性、数学精准结果）的垂直领域任务；
  - **超大规模集群异构资源调度**：拥有数十张乃至数百张 GPU 卡的大型工程团队，需要将采样集群与训练集群弹性分离以榨干算力。
- **与同类工具对比（vs TRL / LLaMA-Factory / DeepSpeed）**：
  - 相比于 **TRL**：Hugging Face TRL 适合单机单卡或中小规模实验，其数据流与计算流耦合度高，面对 4k+ 甚至 16k 极限长思维链 Rollout 时吞吐断崖式下跌；veRL 的 HybridFlow 分布式解耦架构吞吐高出 3~5 倍；
  - 相比于 **LLaMA-Factory**：LLaMA-Factory 是全能型微调工具箱，但在复杂 RL 算法（如跨节点大规模 GRPO、动态 Group 优势归一化、自研 Reward 算子）上的底层定制灵活性不及 veRL。
- **个人评估结论**：**强烈推荐采纳 (status/adopted)**。veRL 是目前开源界在后训练强化学习与推理自进化领域最具工业级工程水准的基石框架，也是深入理解下一代推理大模型（Reasoning Models）训练的必修开源底座。
