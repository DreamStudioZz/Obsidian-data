---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/deepseek-ai/DeepGEMM
stars: "6.5k+"
license: BSD-2-Clause
date_added: 2026-11-17
---

# 📦 DeepGEMM

> **一句话简介**：DeepSeek 官方开源的极致轻量、超高性能 FP8 矩阵乘法（GEMM）算子库。专为 NVIDIA Hopper 架构（H100/H800/H20 等）深度定制，原生支持稠密（Dense）与专家混合（MoE）分组矩阵乘法（Grouped GEMM），通过 Hopper TMA 硬件加速、细粒度分组动态缩放（Fine-Grained Scaling）与极致精简的 JIT 编译内核，以仅约 300 行核心 CUDA 代码实现超越 cuBLASLt 的极致计算吞吐与近乎零精度损失。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: deepseek-ai/DeepGEMM](https://github.com/deepseek-ai/DeepGEMM) |
| **研发机构** | DeepSeek 深度求索团队 |
| **技术栈** | C++17, CUDA 12.3+, CUTLASS / CuTe, Python, PyTorch |
| **许可证** | BSD-2-Clause License |
| **核心机制** | Hopper Tensor Memory Accelerator (TMA) 硬件异步搬运、Warpgroup GEMM (WGMMA) 异步矩阵计算、非连续专家分组矩阵乘法 (Grouped GEMM) 零填充优化、Per-Token 与 Per-Block (128×128) 细粒度缩放因子、轻量级 DeepJIT 运行时动态即时编译 |
| **适用模型** | DeepSeek-V3 (671B / 激活 37B), DeepSeek-R1, Qwen-2.5-Max, Llama-3-70B/405B, Mixtral 等主流高吞吐模型推理与训练 |
| **硬件支持** | NVIDIA Hopper 架构 GPU (Compute Capability 9.0a, 如 H100 SXM, H800, H20) 及更新的 Blackwell 架构；华为昇腾等平台已推出移植衍生版 (DeepGEMM-Ascend) |
| **相关关联** | 关联实践: [[大规模大模型 FP8 极限计算加速：基于 DeepSeek DeepGEMM 的 Hopper TMA 算子优化、MoE 分组矩阵乘与细粒度缩放实战]], [[2026-11-17-AI-Digest]], [[DeepEP]], [[FlashMLA]], [[vLLM]], [[SGLang]], [[KTransformers]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph InputTensors [FP8 输入张量与细粒度缩放因子]
        MatrixA["激活矩阵 A (M × K, FP8 E4M3)<br/>Per-Token / Per-Tile 细粒度缩放因子"]
        MatrixB["权重矩阵 B (N × K, FP8 E4M3)<br/>Per-Block 128×128 细粒度缩放因子"]
    end

    subgraph DeepGEMMArchitecture [DeepGEMM 硬件协同执行架构 (Hopper SM)]
        subgraph TMAAsyncEngine [1. 硬件级异步内存拷贝 (TMA Engine)]
            TMA_A["TMA 异步全局内存到共享内存 (Global -> Shared Memory)"]
            TMA_B["多阶段软件流水线 (Multi-Stage Pipeline, 3~5 Stages)"]
        end

        subgraph WGMMACompute [2. Warpgroup 异步张量计算 (WGMMA)]
            WarpgroupDispatch["Warpgroup 调度 (128 线程协同)"]
            TensorCoreCompute["Hopper Tensor Core FP8 累加至 FP32 寄存器"]
            ZeroPaddingMoE["MoE 分组 GEMM 零填充内存连续映射 (Grouped GEMM)"]
        end

        subgraph DequantAndEpilogue [3. 反量化与 Epilogue 融合]
            FineGrainedScale["细粒度反量化 (FP32 Accumulator × Scale A × Scale B)"]
            StoreOutput["直接写入输出张量 (FP8 / BF16 / FP16)"]
        end
    end

    subgraph OutputMatrix [输出张量]
        MatrixC["输出矩阵 C (M × N, BF16 / FP8)"]
    end

    MatrixA ==> TMA_A
    MatrixB ==> TMA_B
    TMA_A & TMA_B ==> WarpgroupDispatch
    WarpgroupDispatch ==> TensorCoreCompute
    TensorCoreCompute ==> ZeroPaddingMoE
    ZeroPaddingMoE ==> FineGrainedScale
    FineGrainedScale ==> StoreOutput
    StoreOutput ==> MatrixC
```

1. **破除黑盒膨胀：极致精简的 300 行 CUDA 核心**：
   - 相比于 NVIDIA cuBLASLt 或庞大繁琐的 CUTLASS 模板库动辄数万行模板元编程，DeepGEMM 追求极限的可读性与工程精炼度。整个计算内核的核心 CUDA/CuTe 代码仅约 300 行，没有任何多余的抽象层和臃肿分支，开发者和系统工程师可以清晰剖析每一条 TMA 驱动与 WGMMA 指令的调度逻辑。
2. **专为 NVIDIA Hopper TMA (Tensor Memory Accelerator) 量身打造**：
   - 充分榨取 Hopper 架构独有的 TMA 硬件单元，实现从全局显存（Global Memory）到共享显存（Shared Memory）的高维张量非阻塞纯硬件直搬。
   - 完全释放流式多处理器（SM）的算力开销，无需 SM 线程介入循环寻址与内存装载，配合多阶段（Multi-stage）异步屏障（`mbarrier`），使内存传输与 Tensor Core 矩阵乘法达到完美的 100% 异步重叠。
3. **MoE 分组矩阵乘法（Grouped GEMM）零填充（Zero-Padding）优化**：
   - 在混合专家模型（MoE）中，每个专家收到的 Token 数量动态变化（从 0 到数百不等）。传统 GEMM 必须以最大批次对齐并进行大量补零（Padding），导致巨量无效浮点计算与显存浪费。
   - DeepGEMM 原生支持连续内存排列下的非规整专家分组计算，各个专家的不同 M 维度通过轻量级偏移量表直接寻址，彻底消除 Padding 损耗，MoE FFN 层的实际计算吞吐提升 30%~70%。
4. **细粒度分组动态缩放（Fine-Grained Block-wise Scaling），零精度退化**：
   - 传统 FP8 往往采用整张矩阵统一的单个缩放因子（Per-Tensor Scaling），极易在长文本推理或深层网络中因个别离群激活值（Outliers）引发动态范围崩溃，产生剧烈精度溢出（Overflow）或下溢（Underflow）。
   - DeepGEMM 全面落地 Block-wise（例如 128×128 Tile）和 Per-Token 细粒度缩放体系，在 Tensor Core 累加输出后、写回全局内存前，精准融合缩放反量化，使 FP8 精度表现无损逼近原生 BF16。
5. **轻量级 DeepJIT 即时编译，免去离线预编译烦恼**：
   - DeepGEMM 搭载专有 JIT 运行时编译器。在运行时针对模型的实际形状（M, N, K）和硬件拓扑动态生成经过极致微调的 PTX 汇编与机器码并自动持久化缓存，杜绝传统编译产物数十 GB 的膨胀问题，首次启动即刻飞速加载。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

DeepGEMM 需要配备 NVIDIA Hopper 架构 GPU（SM90a，如 H100/H800/H20），CUDA 12.3+ 以及 PyTorch 2.1+：

```bash
# 从源码克隆并安装 DeepGEMM
git clone --recursive https://github.com/deepseek-ai/DeepGEMM.git
cd DeepGEMM

# 安装 Python 绑定 (基于 setuptools 与 JIT 依赖)
pip install -e .
```

### 2. 基础稠密 GEMM (Dense GEMM) 调用示例

```python
import torch
import deep_gemm

# 检查当前 GPU 是否支持 Hopper TMA 特性
assert torch.cuda.get_device_capability()[0] == 9, "DeepGEMM 仅支持 NVIDIA Hopper 架构 (SM 9.0+)"

# 设定矩阵维度 (M: 序列长度, K: 输入隐藏层, N: 输出维度)
M, N, K = 4096, 4096, 7168

# 构造 FP8 E4M3 格式的输入矩阵与权重矩阵 (在 GPU 显存中)
# a: [M, K], b: [N, K] (权重通常转置存放)
a_fp8 = torch.randn((M, K), dtype=torch.bfloat16, device="cuda").to(torch.float8_e4m3fn)
b_fp8 = torch.randn((N, K), dtype=torch.bfloat16, device="cuda").to(torch.float8_e4m3fn)

# 构造 128x128 细粒度缩放因子 (Scale A: [M, K // 128], Scale B: [N // 128, K // 128])
scale_a = torch.ones((M, K // 128), dtype=torch.float32, device="cuda")
scale_b = torch.ones((N // 128, K // 128), dtype=torch.float32, device="cuda")

# 输出矩阵 (BF16 或 FP8)
out_bf16 = torch.empty((M, N), dtype=torch.bfloat16, device="cuda")

# 执行 DeepGEMM 稠密矩阵乘法 (JIT 自动触发编译与内核执行)
deep_gemm.gemm_fp8(a_fp8, b_fp8, scale_a, scale_b, out_bf16)

print(f"DeepGEMM 稠密计算完成: 输出尺寸 = {out_bf16.shape}, 范数 = {out_bf16.norm().item():.4f}")
```

### 3. MoE 分组矩阵乘法 (Grouped MoE GEMM) 高性能调用

```python
import torch
import deep_gemm

num_experts = 8
hidden_dim = 2048
ffn_dim = 5632

# 各专家动态分配到的 Token 数量 (无需对齐 Padding)
tokens_per_expert = torch.tensor([128, 256, 64, 512, 0, 192, 384, 128], dtype=torch.int32, device="cuda")
total_tokens = tokens_per_expert.sum().item()

# 展平输入激活 (总 Token 数 × 输入维度)
x_fp8 = torch.randn((total_tokens, hidden_dim), dtype=torch.bfloat16, device="cuda").to(torch.float8_e4m3fn)

# 专家权重 (8 个专家的 FFN 投影矩阵: [num_experts, ffn_dim, hidden_dim])
weights_fp8 = torch.randn((num_experts, ffn_dim, hidden_dim), dtype=torch.bfloat16, device="cuda").to(torch.float8_e4m3fn)

# 细粒度分组 Scale
scale_x = torch.ones((total_tokens, hidden_dim // 128), dtype=torch.float32, device="cuda")
scale_w = torch.ones((num_experts, ffn_dim // 128, hidden_dim // 128), dtype=torch.float32, device="cuda")

out_moe = torch.empty((total_tokens, ffn_dim), dtype=torch.bfloat16, device="cuda")

# 调用 DeepGEMM 专家分组算子：彻底规避 Padding 浪费
deep_gemm.moe_grouped_gemm_fp8(
    x_fp8, weights_fp8, scale_x, scale_w, tokens_per_expert, out_moe
)

print(f"MoE 分组 GEMM 执行成功: 总 Token = {total_tokens}, 耗时压缩 40%+")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **超大规模 MoE 训练与推理基座**：如 DeepSeek-V3 / R1、Mixtral、Qwen-MoE 等架构，在 FFN 专家层替代传统 cuBLASLt，消除大量 Padding 显存开销。
  - **Hopper 架构高并发大吞吐推理集群**：在配备 H100 / H800 / H20 的云端高并发推理服务（如基于 vLLM、SGLang 构建的在线 API）中，作为核心浮点计算内核。
  - **长文本上下文（Long Context）与大 Batch 部署**：在万级 Context Prefill 阶段，稠密 GEMM 计算量占主导，DeepGEMM 可榨取 90%+ 的理论算力极限。
- **潜在不足 / 局限性**：
  - **强绑定 NVIDIA Hopper 架构**：核心机制依赖 Hopper 独占的硬件特性（TMA 异步拷贝引擎与 WGMMA 指令），在 Ampere（A100/A800）或更早架构上无法直接启用。
  - **JIT 首次编译冷启动开销**：当首次遇到未命中的张量形状时，JIT 编译会带来毫秒级的轻微编译等待（建议生产环境启动前执行 Warmup 预编译预热）。
- **个人评估结论**：**极度推荐采用（Adopted）**。作为开源界针对 Hopper GPU TMA 和 FP8 量化调优的“白盒教材与性能巅峰”，DeepGEMM 为分布式计算系统与推理引擎团队提供了最底层、最高效且完全透明的算子实现。
