---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: "Inference"
github: "https://github.com/deepseek-ai/DualPipe"
stars: "3.3k+"
license: "MIT"
date_added: 2026-11-20
---

# 📦 DualPipe

> **一句话简介**：DeepSeek 官方开源的创新双向流水线并行算法与调度库，专为大规模 MoE 万卡预训练中“计算与通信完全重叠（Full Computation-Communication Overlap）”与极限消除流水线气泡（Pipeline Bubble）而生。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub Repo](https://github.com/deepseek-ai/DualPipe) |
| **技术栈** | Python / PyTorch 2.0+ / CUDA / 分布式通信 |
| **许可证** | MIT License |
| **相关关联** | 关联专题: [[DeepEP]], [[DeepGEMM]], [[3FS]], [[FlashMLA]], [[DeepSpeed]] |

---

## 🚀 核心特性与亮点

1. **首创双向微批次流动调度 (Bidirectional Micro-batch Scheduling)**：
   - 传统流水线并行（如 GPipe、1F1B、Interleaved 1F1B）微批次均沿单一固定方向流动（从 Rank 0 顺序推到 Rank N-1，再反向回传梯度），导致流水线首尾存在显著的起步（Warm-up）与排空（Cool-down）气泡。
   - DualPipe 提出正向与反向流水线**从集群两端对向发起微批次**，首尾 Rank 同时既作为流水线起点、又作为流水线终点，极大地均衡了各节点的内存占用与计算负载。

2. **计算与通信近乎 100% 深度重叠 (Full Computation-Communication Overlap)**：
   - 在超大规模 MoE 模型（如 DeepSeek-V3 / R1 的 671B 参数）训练中，跨节点的流水线激活点对点通信（P2P）以及 MoE 专家路由的 All-to-All 通信极其繁重。
   - DualPipe 将反向传播细粒度拆解为两个阶段：**反向激活梯度计算（Backward for Input, B_input）**与**反向权重梯度计算（Backward for Weights, B_weight）**。在执行 B_weight 密集矩阵乘法（GEMM）计算的同时，异步发射下一微批次的 P2P 通信与 All-to-All 通信，在物理时间轴上实现通信延迟的“无感隐藏”。

3. **流水线气泡比率（Bubble Ratio）逼近理论下界**：
   - 相比标准 1F1B，DualPipe 将流水线空转气泡损耗降低了 50% 以上；
   - 相比 Interleaved 1F1B（虚拟流水线阶段），DualPipe 可以在无需成倍增加显存模型副本的前提下，取得相同甚至更优的低气泡率，大幅释放有效计算时间（MFU 突破 60%+）。

4. **精细化显存管理机制 (WeightGradStore)**：
   - 显式解耦权重梯度累加时机，配合专用轻量容器 `WeightGradStore` 暂存中间张量，避免因为双向流动导致的峰值显存瞬时翻倍。

5. **与同类方案对比**：
   - **相比 Megatron-LM 1F1B / 1F1B-Interleaved**：传统 1F1B 无法掩盖通信开销，遇到高延迟网络或重度 MoE All-to-All 时通信阻塞长达 30%-40%；DualPipe 通过双向交叠与细粒度反向切分，通信耗时被计算几乎完全吸收。
   - **相比 Zero-Bubble Pipeline (ZB-PP)**：DualPipe 专门针对大规模专家并行 MoE 架构的通信特征进行协同设计，与 DeepEP、DeepGEMM 原生无缝融合。

---

## 🛠️ 快速上手与配置

### 1. 环境准备与依赖

```bash
# 要求 PyTorch 2.0+ 及 CUDA 环境
git clone https://github.com/deepseek-ai/DualPipe.git
cd DualPipe
pip install -e .
```

### 2. 核心调度与模型 Chunk 包装

```python
import torch
import torch.nn as nn
from dualpipe import DualPipe, WeightGradStore

# 1. 定义模型拆分阶段（前向与反向必须拆分计算）
class TransformerBlockChunk(nn.Module):
    def __init__(self, config, layer_idx):
        super().__init__()
        self.layer = make_transformer_layer(config, layer_idx)
        
    def forward(self, hidden_states):
        return self.layer(hidden_states)

# 2. 初始化 DualPipe 调度器
# chunks 包含当前 GPU Rank 负责的正向与反向模型切片
model_chunks = [TransformerBlockChunk(config, i) for i in assigned_layers]

weight_grad_store = WeightGradStore()

dual_pipe = DualPipe(
    modules=model_chunks,
    weight_grad_store=weight_grad_store,
    num_chunks=2,               # 每个 Rank 承载的模型块数
    num_microbatches=16,         # 微批次数量
    device=torch.cuda.current_device()
)

# 3. 运行双向交错前向与反向流水
# 双向调度器在底层自动编排 P2P 通信与 CUDA 流异步重叠
dual_pipe.step(input_microbatches_dir0, input_microbatches_dir1)

# 4. 执行梯度归约与优化器步进
dual_pipe.commit_weights()
optimizer.step()
optimizer.zero_grad()
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **万亿/千亿级 MoE 预训练与微调**：网络带宽受限、跨节点 All-to-All 通信和跨机 PP 通信耗时成为主要吞吐瓶颈的超大集群；
  2. **高算力利用率（MFU）追求者**：希望将大规模分布式训练集群的有效算力利用率压榨到物理极值的底层 Infra 团队；
  3. **与 DeepSeek 全家桶协同**：结合 DeepEP（通信加速）、DeepGEMM（FP8 算子加速）以及 3FS（高吞吐存储），构建世界顶尖的智算训练管线。
- **潜在不足 / 局限性**：
  - 代码要求深入理解分布式张量并行与流水线并行的底层切分，对开发者的并行计算与 CUDA 异步流调度功底要求极高；
  - 双向流动增加了调度时序控制的复杂度，必须严格保证双向数据流输入微批次的大小一致与时钟对齐。
- **个人评估结论**：**大模型分布式系统工程必读必用标杆**。DeepSeek 再次用代码证明：算法与底层硬件协同优化（Co-Design）能将集群整体算力效率推升至全新境界。