---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/microsoft/DeepSpeed
stars: "36.2k+"
license: Apache-2.0
date_added: 2026-11-05
---

# 📦 DeepSpeed

> **一句话简介**：由微软（Microsoft）开源的大规模分布式大模型训练、微调与极速推理优化基础设施（Extreme-Scale Distributed Deep Learning Library）。首创突破性的 **ZeRO (Zero Redundancy Optimizer)** 显存解耦技术体系（ZeRO-1/2/3、ZeRO-Offload 与 ZeRO-Infinity），通过在分布式节点间智能分片优化器状态、梯度与模型参数，并结合 CPU/NVMe 异构存储分层卸载，彻底击碎大模型预训练与全量微调的显存墙，成为全球训练万亿参数超大规模模型的事实标准底层基石之一。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: microsoft/DeepSpeed](https://github.com/microsoft/DeepSpeed) |
| **研发机构** | 微软人工智能研究院（Microsoft AI & DeepSpeed Team） |
| **技术栈** | Python, C++, CUDA, Triton, PyTorch, NCCL, MPI |
| **许可证** | Apache-2.0 License |
| **核心算法与技术** | ZeRO (Stage 1/2/3), ZeRO-Offload (CPU/NVMe), ZeRO-Infinity, ZeRO++ (通信量化压缩), Activation Checkpointing, 3D 并行 (DP+TP+PP), DeepSpeed-MoE, DeepSpeed-Inference |
| **显存压缩比** | ZeRO-1 降低 4x 显存；ZeRO-2 降低 8x 显存；ZeRO-3 实现显存与卡数线性成正比缩减（$1/N$） |
| **核心应用场景** | 7B-405B+ 大模型分布式预训练、长上下文全参数微调、RLHF 后训练底座、消费级单卡/工作站离线大模型训练 |
| **相关关联** | 关联实践: [[大规模分布式大模型显存优化架构：基于 DeepSpeed ZeRO-3、Offload 与 Activation Checkpointing 极限训练实战]], [[2026-11-05-AI-Digest]], [[veRL]], [[vLLM]], [[LLaMA-Factory]], [[Unsloth]], [[TRL]], [[TensorRT-LLM]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph TraditionalDP [传统数据并行 (Standard Data Parallelism)]
        DP_Node1["GPU 1: 模型参数 (P) + 梯度 (G) + 优化器状态 (OS)"]
        DP_Node2["GPU 2: 模型参数 (P) + 梯度 (G) + 优化器状态 (OS)"]
        DP_Note["每张 GPU 独立完整冗余拷贝一份全部状态 (巨大显存浪费)"]
    end

    subgraph ZeRO_Stages [DeepSpeed ZeRO 分层解耦显存消除]
        direction TB
        ZeRO1["ZeRO-1 (P_os): 优化器状态分片 (Optimizer State Partitioning)<br>显存节省 4x，通信量 0 增加"]
        ZeRO2["ZeRO-2 (P_os+g): 优化器状态 + 梯度分片 (Gradient Partitioning)<br>显存节省 8x，通信量 0 增加"]
        ZeRO3["ZeRO-3 (P_os+g+p): 优化器状态 + 梯度 + 模型参数全量分片<br>显存缩减至 1/N，引入 1.5x 动态通信开销"]
    end

    subgraph MemoryOffloading [异构分层卸载 (Heterogeneous Offload)]
        CPURAM["主机内存 (CPU DRAM): CPU Adam 异步优化器更新"]
        NVMeSSD["非易失性高速闪存 (NVMe SSD): ZeRO-Infinity 极限虚拟显存"]
        GPUVRAM["显卡显存 (GPU HBM/VRAM): 仅保留即时计算 Forward/Backward Layer"]
    end

    TraditionalDP -.->|技术革命| ZeRO_Stages
    ZeRO_Stages --> ZeRO3
    ZeRO3 <--> MemoryOffloading
```

### 1. ZeRO 技术体系：打破显存冗余的数学革新
在大模型混合精度（FP16/BF16）训练中，以 Adam 优化器为例，显存被以下三大块牢牢霸占：
- **模型参数（Parameters）**：1 个参数占 2 Bytes（FP16）；
- **梯度（Gradients）**：1 个参数占 2 Bytes（FP16）；
- **优化器状态（Optimizer States）**：Adam 需要维护 FP32 参数主备份（4 Bytes）、一阶动量 Momentum（4 Bytes）和二阶动量 Variance（4 Bytes），合计每个参数高达 **12 Bytes**。
- **总静态显存开销**：$2 + 2 + 12 = 16 \text{ Bytes / 参数}$。这意味着一个 70B 模型，仅静态权重和优化器状态就需要消耗 $70 \times 16 = 1120 \text{ GB}$ 显存！
DeepSpeed 通过优雅的分片算法彻底消除这一冗余：
- **ZeRO Stage 1 ($P_{os}$)**：将优化器状态均匀打散切分到 $N$ 张 GPU 上，每张卡仅维护 $1/N$ 优化器状态，直接消除 75% 的静态开销，且**通信量与传统数据并行完全一致**；
- **ZeRO Stage 2 ($P_{os+g}$)**：进一步将计算出的梯度也按卡分片，反向传播计算完立即 Reduce-Scatter 并释放冗余梯度，显存进一步砍半；
- **ZeRO Stage 3 ($P_{os+g+p}$)**：连模型参数也彻底分片。每张 GPU 仅常驻 $1/N$ 的模型权重；在前向传播与反向传播计算到特定算子层（Layer）时，通过高速 All-Gather 动态汇聚所需权重，计算完毕后**即刻在显存中销毁**，显存需求随集群 GPU 数量 $N$ 严格线性缩减。

### 2. ZeRO-Offload 与 ZeRO-Infinity：突破单机物理极限
- **ZeRO-Offload**：将计算开销较小但显存占比高达 75% 的 Adam 优化器状态和 CPU 计算卸载至主机物理内存（Host RAM），利用高度优化的 C++/AVX 指令集 `DeepSpeedCPUAdam` 在 CPU 侧完成参数更新，利用 PCIe 高速总线在 GPU 与 CPU 间流水线双缓冲传输，使得单机 8 卡工作站即可全量训练 13B-70B 大模型；
- **ZeRO-Infinity**：将显存卸载边界进一步推向廉价的 NVMe SSD，构建“GPU HBM - CPU DRAM - NVMe”三级存储架构，利用异步预取（Asynchronous Prefetching）技术隐藏 I/O 延迟，让单台消费级 PC 也能加载和运行百亿甚至千亿模型训练。

### 3. ZeRO++：破解跨节点低带宽通信瓶颈
- ZeRO-3 带来了 $1.5\times$ 的额外通信开销，在缺乏 InfiniBand 高速网络的低端以太网环境下容易造成 GPU 计算停顿（Bubble）；
- DeepSpeed 演进出的 **ZeRO++** 引入了三大通信优化：
  - **qwZ (Quantized Weights in ZeRO)**：将 All-Gather 汇聚的模型权重压缩为 INT8，通信量骤降 50%；
  - **qgZ (Quantized Gradients in ZeRO)**：在 Reduce-Scatter 阶段对梯度进行分块 INT8 量化传输；
  - **hpZ (Hierarchical Partitioning)**：在节点内利用高速 NVLink 进行全量分片，跨节点维持数据副本，兼顾显存节省与低带宽通信。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

```bash
# 推荐基于 CUDA 12.x 与 PyTorch 2.x 环境安装 DeepSpeed
pip install deepspeed

# 验证 C++ 与 CUDA 扩展内核是否编译成功 (如 cpu_adam, fused_lamb 等)
ds_report
```

### 2. 生产级 ZeRO-3 + Offload 配置文件 (`ds_config_zero3.json`)

```json
{
  "train_batch_size": "auto",
  "train_micro_batch_size_per_gpu": "auto",
  "gradient_accumulation_steps": "auto",
  "gradient_clipping": 1.0,
  "bf16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 3,
    "offload_optimizer": {
      "device": "cpu",
      "pin_memory": true
    },
    "offload_param": {
      "device": "none"
    },
    "overlap_comm": true,
    "contiguous_gradients": true,
    "sub_group_size": 1e9,
    "reduce_bucket_size": "auto",
    "stage3_prefetch_bucket_size": "auto",
    "stage3_param_persistence_threshold": "auto",
    "stage3_max_live_parameters": 1e9,
    "stage3_max_reuse_distance": 1e9,
    "stage3_gather_16bit_weights_on_model_save": true
  },
  "activation_checkpointing": {
    "partition_activations": true,
    "cpu_checkpointing": false,
    "contiguous_memory_optimization": true,
    "number_checkpoints": null,
    "synchronize_checkpoint_boundary": false,
    "profile": false
  },
  "wall_clock_breakdown": false
}
```

### 3. 基于 PyTorch 原生脚本极速启动 DeepSpeed 引擎

```python
import torch
import deepspeed
from transformers import AutoModelForCausalLM, AutoTokenizer

# 1. 初始化模型拓扑 (在 ZeRO-3 下可通过零显存初始化的 context 构建)
model_name = "Qwen/Qwen2.5-7B-Coder"
tokenizer = AutoTokenizer.from_pretrained(model_name)

# 2. 注入 DeepSpeed 引擎 (零侵入式替代标准 PyTorch 优化器与梯度逻辑)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.bfloat16)

parameters = filter(lambda p: p.requires_grad, model.parameters())

model_engine, optimizer, _, _ = deepspeed.initialize(
    model=model,
    model_parameters=parameters,
    config="ds_config_zero3.json"
)

# 3. 标准分布式训练循环
for step, batch in enumerate(data_loader):
    inputs = {k: v.to(model_engine.local_rank) for k, v in batch.items()}
    
    # 前向计算 (ZeRO-3 会自动在算子触发前 All-Gather 参数，并在计算后立即丢弃)
    outputs = model_engine(**inputs)
    loss = outputs.loss
    
    # 梯度反向传播 (DeepSpeed 自动分片管理梯度与激活)
    model_engine.backward(loss)
    
    # 优化器步进 (自动触发 CPU-Adam 或分布式参数更新)
    model_engine.step()
```

---

## 💡 工程实战点评与落地选型建议

- **生产选型定位**：
  - **中全量微调与预训练**：DeepSpeed 是大模型后训练、指令对齐与全参数微调绝对的中流砥柱，已被 HuggingFace Accelerate、LLaMA-Factory、TRL 等顶级工具作为官方默认后端全面集成；
  - **对比 PyTorch FSDP**：PyTorch 原生 FSDP（Fully Sharded Data Parallel）与 ZeRO-3 原理完全一致且侵入性略低，但 DeepSpeed 在**异构卸载（ZeRO-Offload）**、**CPU-Adam 极致优化**、**万亿 MoE 支持**以及**小显存机器单机极限吞吐**方面依然具有独一无二的工程优势。
- **最佳硬件搭配与避坑原则**：
  - 若在缺乏 NVLink 的消费级 GPU（如 4x RTX 4090）或云端 10GbE 以太网集群训练，**首选 ZeRO-2 + Offload**；因为 ZeRO-3 频繁的 AllGather 会在慢速总线上产生严重的通信气泡；
  - 务必开启 `pin_memory: true` 与 `overlap_comm: true`，借助锁页内存与 CUDA Streams 实现算力计算与张量通信的完美流水线重叠。

---

## 🔗 关联卡片与生态导航
- 关联实战：[[大规模分布式大模型显存优化架构：基于 DeepSpeed ZeRO-3、Offload 与 Activation Checkpointing 极限训练实战]]
- 关联工具：[[veRL]]、[[vLLM]]、[[LLaMA-Factory]]、[[Unsloth]]、[[TRL]]、[[TensorRT-LLM]]
- 今日日报：[[2026-11-05-AI-Digest]]
