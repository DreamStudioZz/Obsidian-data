---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/kvcache-ai/ktransformers
stars: "7.5k+"
license: Apache-2.0
date_added: 2026-10-27
---

# 📦 KTransformers

> **一句话简介**：由清华大学与 KVCache.AI 联合研发的轻量化 Python 原生异构混合推理框架。利用超大规模 MoE（混合专家模型）的稀疏激活特性，首创 **CPU/GPU 内存分层卸载** 与 **Marlin-AMX/AVX 融合算子加速**，让单张消费级显卡（如 RTX 4090）顺畅运行千亿参数级巨型模型（如 DeepSeek-V3/R1 671B）。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: kvcache-ai/ktransformers](https://github.com/kvcache-ai/ktransformers) |
| **技术栈** | C++20 / CUDA / Python 3.10+ / Intel AMX & AVX-512 / Marlin / Torch |
| **许可证** | Apache-2.0 License |
| **核心能力** | MoE 稀疏专家异构显存卸载（Heterogeneous CPU-GPU Offloading）、Marlin FP4/INT4 极速 GPU 矩阵内核、Intel AMX/AVX-512 高并发 CPU GEMM 算子、异步专家预取流水线（Expert Prefetching）、DeepSeek-V2/V3/R1 原生深度支持、OpenAI 兼容 API 极速部署 |
| **相关关联** | 关联实践: [[消费级单卡运行巨量MoE模型：基于 KTransformers 的异构 CPU-GPU 显存卸载与 Marlin-AMX 算子加速实战]], [[2026-10-27-AI-Digest]], [[LMDeploy]], [[vLLM]], [[llama.cpp]], [[TensorRT-LLM]], [[Unsloth]], [[Xinference]] |

---

## 🚀 核心架构与杀手级特性

### 1. MoE 稀疏激活特性与异构分层内存布局
以 DeepSeek-V3/R1 等新一代万亿/千亿级 MoE 模型为例，模型总参数高达 671B，但每个 Token 经过路由（Router）分发时，仅激活其中的少数专家（如每个 Token 激活 8 个专家，激活参数量仅约 37B，其余 248 个专家处于休眠状态）：
- **注意力与共享专家常驻显存**：将计算密集且每个 Token 必经的 Attention 权重与 Shared Experts 牢牢固定在 GPU VRAM（如 24GB）中；
- **庞大稀疏专家池卸载至主机内存**：将多达几百 GB 的 Routed Experts 矩阵存放在价格廉价、扩展性强的主机物理内存（Host DDR5 RAM）中；
- **分层拓扑解耦**：彻底打破了“运行大模型必须采购 8 卡 H800 / 8 卡 A100”的算力壁垒。

### 2. GPU 端 Marlin 4-bit 算子极速计算
- 集成业内顶尖的 **Marlin FP4/INT4 高性能 GEMM 核心**，针对张量核心（Tensor Core）进行了极端内存对其与算子融合优化；
- 避免传统量化方案在计算时的即时反量化内存额外开销，为显存受限的单卡环境榨干每一点 CUDA 吞吐。

### 3. CPU 端 Intel AMX / AVX-512 算子融合
传统 CPU Offloading 方案最大的痛点在于 CPU 浮点矩阵乘法极其缓慢，导致推理速度暴跌至 < 1 token/s：
- **深度整合 Intel AMX (Advanced Matrix Extensions)**：充分利用至强（Xeon）等现代处理器上的硬件瓦片矩阵乘法引擎（Tile GEMM），提供高达数倍于传统 AVX-512 的整型与浮点计算吞吐；
- **多线程 NUMA 亲和性调度**：针对多插槽 CPU 与多通道 DDR5 内存设计无锁环形队列与线程亲和性（Thread Affinity）机制，消除了跨 NUMA 内存访问的带宽抖动。

### 4. 异步计算与传输隐藏（Prefetching Pipeline Overlap）
为了消除经由 PCIe 总线从主机内存加载专家权重带来的延迟：
- **专家预取流水线（Expert Prefetching）**：在前一层（Layer $N$）进行注意力计算和路由器预测的瞬间，后台异步流水线根据预测出的专家 ID，提前通过 PCIe DMA 引擎将目标专家的稀疏权重预取至 GPU 临时缓冲池；
- **计算传输完全掩盖**：在 Layer $N+1$ 到来时，权重已预载完毕，使得 PCIe 传输时间与 GPU/CPU 的矩阵计算完全重叠（Overlapping）。

### 5. Python 原生的极客级灵活注入（Injectable Engine）
- 不同于编译繁重的 C++ 黑盒框架，KTransformers 采用“Python 外壳 + 核心 C++/CUDA 扩展”的设计；
- 开发者可以通过一份精简的 `optimize_rule.yaml` 声明式规则，精准指定模型的每一层、每一个模块（Linear, Gate, Expert）究竟运行在哪个硬件后端（CPU AMX、GPU Marlin 或 Torch 原生），极易进行定制实验。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖安装
```bash
# 推荐在配备 CUDA 12.1+ 与最新 Torch 的环境中安装
pip install ktransformers

# 验证底层 CPU 指令集支持（检查 AMX / AVX-512）
lscpu | grep -E "amx|avx512"
```

### 2. 编写优化配置规则 `optimize_rule.yaml`
```yaml
# 针对 DeepSeek-V3 671B 的典型单卡混合部署策略
- match:
    name: ".*self_attn.*"
  replace:
    class: "ktransformers.operators.attention.KTransformersAttention"
    kwargs:
      device: "cuda:0"
      backend: "marlin"

- match:
    name: ".*mlp.shared_experts.*"
  replace:
    class: "ktransformers.operators.linear.KTransformersLinear"
    kwargs:
      device: "cuda:0"
      backend: "marlin"

- match:
    name: ".*mlp.experts.*"
  replace:
    class: "ktransformers.operators.experts.KMoEExperts"
    kwargs:
      device: "cpu"
      backend: "amx" # 支持 Intel AMX 加速；非 Intel 平台可设为 avx512
      thread_pool_size: 32
```

### 3. 一键启动本地交互式终端与 OpenAI 兼容服务
```bash
# 单卡 RTX 4090 (24GB) + 192GB 主机内存启动 DeepSeek-V3
ktransformers --model_path /data/models/deepseek-ai/DeepSeek-V3 \
              --optimize_config_path optimize_rule.yaml \
              --port 8000 \
              --max_new_tokens 4096
```

### 4. Python 原生调用示例
```python
from ktransformers.optimize.optimize import optimize_and_load_gguf
from ktransformers.models.modeling_deepseek_v3 import DeepSeekV3ForCausalLM
from transformers import AutoTokenizer
import torch

model_path = "/data/models/deepseek-ai/DeepSeek-V3"
rule_path = "./optimize_rule.yaml"

# 1. 加载 Tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)

# 2. 注入 KTransformers 异构优化规则并极速加载
with torch.device("meta"):
    model = DeepSeekV3ForCausalLM.from_pretrained(model_path, trust_remote_code=True)

optimize_and_load_gguf(model, rule_path, model_path)

# 3. 执行推理生成
prompt = "请详细分析在异构 CPU-GPU 架构下，针对 MoE 模型的显存分层卸载优化原理："
inputs = tokenizer(prompt, return_tensors="pt").to("cuda:0")

outputs = model.generate(
    **inputs,
    max_new_tokens=512,
    temperature=0.7,
    do_sample=True
)

print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

---

## 📊 与主流大模型推理框架横向对比

| 评估维度 | llama.cpp | vLLM / SGLang | LMDeploy | KTransformers |
| :--- | :--- | :--- | :--- | :--- |
| **超大 MoE 单卡单机支持 (DeepSeek-V3/R1)** | 支持（纯 CPU 或常规层卸载，极慢） | 不支持（必须全显存装载，需 8xH100） | 部分支持（需多卡张量并行） | **业界最佳（首创专家级细粒度 CPU 卸载与 AMX 加速）** |
| **千亿 MoE 消费级单卡生成速度** | 1 ~ 3 tokens/s | N/A (显存不足直接 OOM) | N/A (显存不足) | **12 ~ 28 tokens/s（具生产实用价值）** |
| **GPU 算子优化** | cuBLAS / GGML 自研 | FlashAttention-3 / PagedAttention | TurboMind C++ 算子融合 | Marlin 4-bit 极致算子 |
| **CPU 硬件算力挖掘** | AVX-512 基础支持 | 弱（偏向 GPU 集群） | 弱（偏向 GPU 集群） | **深度支持 Intel AMX 瓦片矩阵乘法** |
| **动态规则热插拔** | 否（写死编译逻辑） | 否 | 否 | **是（基于 YAML 声明式规则灵活替换算子）** |

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **中小团队与高校实验室本地运行千亿 MoE 巨型模型**：无需采购动辄上百万元的 8 卡专业级算力服务器，只需配备一台双路 Xeon + 256GB 内存 + 单张 RTX 4090 的工作站，即可零成本私有化跑通 DeepSeek-V3/R1；
  - **严苛数据安全与离线推理场景**：国防、金融核心部门严禁业务数据出内网，且需要超高代码生成或复杂数学推理能力；
  - **新型 MoE 架构算子探索与实验**：研究者可以利用其高度模块化的 Python 接口快速替换自定义 Triton/CUDA/AMX 核函数。
- **潜在不足 / 局限性**：
  - **对主机内存带宽极其敏感**：必须配合双通道或四通道高频 DDR5 内存；单通道 DDR4 会导致 PCIe 和内存吞吐形成瓶颈；
  - **极高并发吞吐受限**：适合低并发、单用户交互或中小型批量推理；在成百上千的高并发公共 API 场景下仍推荐多卡全显存集群。
- **个人评估结论**：**建议全面引入（adopted）**。作为在消费级和低成本单卡硬件上跑通新一代开源超巨型 MoE 模型的“破局者”，填补了工业界在算力门槛与顶尖模型规模之间的巨大鸿沟。