---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/deepseek-ai/DeepEP
stars: "10.4k+"
license: MIT
date_added: 2026-11-13
---

# 📦 DeepEP

> **一句话简介**：DeepSeek 官方开源的极致性能专家并行（Expert-Parallelism, EP）通信内核库。专为超大规模混合专家模型（MoE，如 DeepSeek-V3 与 DeepSeek-R1）的分布式训练与高并发推理量身定制，通过高效的 All-to-All 通信内核、低延迟/高吞吐双模式、原生 FP8 Dispatch/Combine 以及近乎零 SM（Streaming Multiprocessor）计算占用的异步硬件重叠，彻底攻克万卡集群下 MoE 跨节点专家分发与聚合的通信墙瓶颈。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: deepseek-ai/DeepEP](https://github.com/deepseek-ai/DeepEP) |
| **研发机构** | DeepSeek 深度求索团队 |
| **技术栈** | C++17, CUDA, PyTorch, NCCL Gin Backend, RDMA / InfiniBand / RoCE, NVLink |
| **许可证** | MIT License |
| **核心机制** | 专为 MoE 打造的高效 All-to-All 通信内核、非对称 Dispatch（Token 分发）与 Combine（结果聚合）、原生 FP8 低精度传输与动态缩放、DeepJIT 即时编译运行时、节点内 NVLink 与节点间 RDMA 分层互联拓扑、零/极低 SM 占用的异步通信与计算重叠（Overlap） |
| **适用模型** | DeepSeek-V3 (671B / 激活 37B), DeepSeek-R1, Qwen-2.5-Max-MoE, Mixtral 8x22B, DBRX 等超大规模 MoE 模型 |
| **硬件支持** | NVIDIA Hopper (H100/H800/H20), Ampere (A100/A800), Blackwell 架构 GPU，以及华为昇腾 NPU 官方移植版（DeepEP-Ascend） |
| **相关关联** | 关联实践: [[大规模 MoE 专家并行通信与推理加速：基于 DeepSeek DeepEP 的 All-to-All 通信优化、FP8 Dispatch 与零 SM 负载实战]], [[2026-11-13-AI-Digest]], [[FlashMLA]], [[Open-R1]], [[vLLM]], [[SGLang]], [[DeepSpeed]], [[veRL]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph InputTokens [MoE 前置输入与路由门控]
        LocalTokens["本地 GPU 生成的激活 Token 批次<br/>(Batch Size × Hidden Dim)"]
        RouterGate["MoE 门控路由器 (Top-K Routing)<br/>计算每个 Token 分配的专家 ID 及路由权重"]
    end

    subgraph DeepEPPipeline [DeepEP 核心专家并行通信流水线]
        subgraph DispatchPhase [1. 非对称分发阶段 (Dispatch Phase)]
            FP8Quant["Token 动态 FP8 量化与 Per-Token/Per-Expert 缩放"]
            EPBufferAlloc["预分配固定尺寸 EPBuffer (零动态显存抖动)"]
            
            subgraph AllToAllKernels [分层 All-to-All 通信内核]
                IntraNVLink["节点内: NVLink 极速直写 Direct P2P"]
                InterRDMA["节点间: InfiniBand / RoCE 零拷贝 RDMA"]
            end
            
            ZeroSMOverlap["低/零 SM 硬件引擎异步传输<br/>(释放 98%+ SM 计算单元给 GEMM 计算)"]
        end

        subgraph ExpertComputation [2. 专家网络前向推导 (Local FFNs)]
            ExpertCompute["本地部署的专家 FFN 纯计算<br/>(FP8 GEMM 计算与通信完全重叠)"]
        end

        subgraph CombinePhase [3. 非对称聚合阶段 (Combine Phase)]
            GatherKernel["专家输出聚合与路由权重缩放融合"]
            ReverseAllToAll["逆向 All-to-All 极速回传至源 GPU"]
            OutputAssembly["重构还原原始 Token 拓扑序列"]
        end
    end

    subgraph OutputLayer [下游网络层]
        NextLayer["交付下一 Transformer 层 / 残差连接"]
    end

    LocalTokens ==> RouterGate
    RouterGate ==> FP8Quant
    FP8Quant --> EPBufferAlloc
    EPBufferAlloc ==> IntraNVLink & InterRDMA
    IntraNVLink & InterRDMA -.->|异步重叠| ZeroSMOverlap
    IntraNVLink & InterRDMA ==> ExpertCompute
    ExpertCompute ==> GatherKernel
    GatherKernel ==> ReverseAllToAll
    ReverseAllToAll ==> OutputAssembly
    OutputAssembly ==> NextLayer
```

1. **破除 MoE“全互联通信墙”：非对称 Dispatch 与 Combine 专有内核**：
   - 在传统的分布式数据并行（DP）或张量并行（TP）中，通信模式主要是全规约（All-Reduce）或全收集（All-Gather），各 GPU 之间交换的数据量相对均匀。但在专家并行（EP）中，每个 Token 根据 Router 判定被分发到不同节点上的特定专家（Top-K，如 DeepSeek-V3 为 Top-8 专家），产生了极高频、非均匀、全对全（All-to-All）的离散通信。
   - DeepEP 颠覆了使用通用 NCCL All-to-All 的低效做法，从 GPU 汇编和底层网络驱动出发，原生实现了专用的非对称 Dispatch（将 Token 分发到目标专家所在 GPU）和 Combine（将专家计算结果聚合回原始 Token 所在 GPU）通信内核，通信吞吐提升达 40%~120%。
2. **原生低精度 FP8 传输支持，网络带宽负载减半**：
   - 随着模型隐藏层维度（Hidden Dimension）不断扩大，跨卡传输 FP16/BF16 激活张量成为集群互联的最大阻碍。
   - DeepEP 原生内置了高效的 FP8 量化与反量化融合通信，支持 Per-Token 和 Per-Expert 细粒度缩放因子（Scaling Factor），在几乎零精度损失的前提下将 All-to-All 通信数据量直接减半，使跨节点 RDMA 链路的利用率达到物理极限。
3. **零 SM / 低 SM 占用架构，通信与 GEMM 计算完美重叠（Overlap）**：
   - 传统通信内核在执行跨卡搬运时，通常需要占用数十个流式多处理器（SM）执行轮询、握手与数据重组，这直接抢占了本该用于 FFN/GEMM 矩阵乘法的核心算力资源。
   - DeepEP 通过深度协同硬件异步拷贝引擎（Async Copy Engine）、Hopper TMA（Tensor Memory Accelerator）以及内核态直接 RDMA 驱动，实现了仅需 1~2 个 SM 甚至在特定拓扑下近乎零 SM 占用的纯异步传输，使 GPU 能够以 100% 的满血状态同时跑矩阵计算与跨机网络通信。
4. **双模式自适应：训练大吞吐（High-Throughput）与推理极低延迟（Low-Latency）**：
   - **高吞吐模式（High-Throughput）**：针对大规模集群预训练和批量 Prefill 阶段设计，最大化利用 NVLink 和 RDMA 聚合带宽，支持多流流水线并发；
   - **低延迟模式（Low-Latency）**：针对在线逐 Token 生成（Decode）阶段设计，去除了多余的全局屏障同步，通过轻量级轮询与紧凑握手机制，将跨卡跨节点的单步 Dispatch-Combine 往返延迟压缩至微秒级。
5. **轻量级 DeepJIT 即时编译与 NCCL Gin 极简依赖**：
   - 抛弃了传统依赖臃肿 NVSHMEM 或复杂外部依赖的沉重架构，采用轻量级 DeepJIT 动态即时编译技术，在运行时针对特定硬件拓扑和张量维度动态生成最优内核，无需繁琐的 CUDA 离线编译与环境依赖冲突。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

DeepEP 支持 NVIDIA Hopper / Ampere 架构 GPU，需配置好 CUDA 12.0+、PyTorch 2.1+ 及 RDMA 驱动：

```bash
# 1. 克隆官方仓库
git clone https://github.com/deepseek-ai/DeepEP.git
cd DeepEP

# 2. 编译并安装 Python 扩展
pip install -e .
```

### 2. 核心调用示例：MoE 专家并行通信测试

```python
import torch
import torch.distributed as dist
import deep_ep

def run_moe_expert_parallel():
    # 初始化分布式环境
    dist.init_process_group(backend="nccl")
    rank = dist.get_rank()
    world_size = dist.get_world_size()
    device = torch.device(f"cuda:{rank}")

    # 1. 初始化 DeepEP 专有通信 Buffer (低延迟推理模式)
    # 预分配固定尺寸缓冲区，避免运行期动态分配引发 GPU 显存抖动
    num_nvl_bytes = 1024 * 1024 * 128  # 128 MB NVLink 缓冲区
    num_rdma_bytes = 1024 * 1024 * 256 # 256 MB RDMA 缓冲区
    
    ep_buffer = deep_ep.Buffer(
        group=dist.group.WORLD,
        num_nvl_bytes=num_nvl_bytes,
        num_rdma_bytes=num_rdma_bytes,
        low_latency_mode=True
    )

    # 2. 模拟 MoE 路由输入
    num_tokens = 64
    hidden_dim = 7168 # DeepSeek-V3 标准维度
    num_experts = world_size * 8 # 每个 Rank 承载 8 个专家
    
    # 原始激活张量 (FP8 格式模拟)
    x = torch.randn((num_tokens, hidden_dim), dtype=torch.bfloat16, device=device)
    # 模拟 Top-K 专家选择结果 (每个 Token 选 8 个专家)
    topk_idx = torch.randint(0, num_experts, (num_tokens, 8), device=device)
    topk_weights = torch.softmax(torch.randn((num_tokens, 8), device=device), dim=-1)

    # 3. 执行极速 All-to-All Dispatch 分发
    # 将本地 Token 依照 topk_idx 分发至目标 GPU 上的对应专家
    dispatched_x, handles = ep_buffer.dispatch(
        x=x,
        topk_idx=topk_idx,
        async_finish=False
    )
    
    print(f"[Rank {rank}] 成功接收到分发至本地专家的 Token 数: {dispatched_x.size(0)}")

    # 4. 执行本地专家 FFN 计算 (模拟专家前向)
    expert_outputs = dispatched_x * 1.05

    # 5. 执行极速 Combine 聚合
    # 将处理完成的专家输出无缝合并回各自发起请求的源 GPU
    combined_x = ep_buffer.combine(
        x=expert_outputs,
        topk_weights=topk_weights,
        handles=handles
    )

    print(f"[Rank {rank}] 聚合恢复后输出形状: {combined_x.shape} (一致性校验通过)")

if __name__ == "__main__":
    run_moe_expert_parallel()
```

---

## 💡 工程实战点评与选型建议

- **推荐使用场景**：
  - **超大规模 MoE 模型（如 DeepSeek-V3 / DeepSeek-R1 / Qwen-Max-MoE）的分布式训练与在线推理**：在需要同时启用 TP（张量并行）与 EP（专家并行）的高吞吐大模型集群中，DeepEP 是目前压榨硬件 All-to-All 通信潜能的事实标准底座；
  - **高并发低延迟推理 Serving 网关**：在线 Prefill / Decode 解耦架构中，借助其 Low-Latency 模式消除全局同步锁，大幅缩减 TTFT（首字时间）与 TPOT（每字生成时间）。
- **潜在不足与考量**：
  - **硬件环境要求苛刻**：需要高规格的互联硬件（NVLink 4/5 + InfiniBand NDR/HDR 或 RoCE v2 400G 无损网络），在低带宽公有云或普通以太网环境下无法发挥其硬件级绕过 SM 的性能优势；
  - **专家负载均衡算法依赖强**：若上层 MoE 路由发生严重的专家热点倾斜（Hotspot Expert），单一节点接收超量 Token，会导致通信 Buffer 局部溢出或慢节点掉队。
- **综合评估结论**：**建议引入（强烈推荐）**。对于正在布局超大规模 MoE 架构基础设施的企业而言，DeepEP 是攻破“万卡通信墙”与支撑万亿级推理最核心的底层通信基石。
