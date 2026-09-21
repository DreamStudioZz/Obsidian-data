---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/vllm-project/vllm
stars: "40k+"
license: Apache-2.0
date_added: 2026-09-21
---

# 📦 vLLM

> **一句话简介**：基于 PagedAttention 算法设计的高吞吐、低延迟开源大语言模型推理与 Serving 框架。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: vllm-project/vllm](https://github.com/vllm-project/vllm) |
| **技术栈** | Python, C++/CUDA, Triton |
| **许可证** | Apache-2.0 |
| **相关关联** | [[000-AI-Index]], [[2026-09-21-AI-Digest]] |

---

## 🚀 核心架构与工程优势

### 1. PagedAttention 机制
- **传统痛点**：传统 Transformer 生成过程中，KV Cache 随着生成序列变长而线性增长。为了预留空间，通常必须预先在 GPU 显存中开辟连续的显存块，这带来了高达 **60%~80% 的显存碎片浪费（Internal & External Fragmentation）**。
- **创新实现**：借鉴操作系统的虚拟内存和分页（Paging）概念，将 KV 缓存划分为固定大小的“块（Blocks）”。每个块可以存放在物理显存的任意非连续位置。显存碎片降低到 **4% 以下**。

### 2. Continuous Batching（连续批处理）
- 传统的批处理必须等待一个 Batch 内所有请求都生成完毕才能调度下一个 Batch（导致短请求被长请求拖垮）。
- vLLM 采用迭代级（Iteration-level）动态调度，每个 Token 步都可以动态剔除已完成请求并塞入新到达请求，GPU 利用率几乎拉满。

### 3. 多 LoRA 并发支持与投机采样
- 支持在单张 GPU 显存内同时挂载数十个不同的 LoRA 微调权重，不同请求路由到同一个基础模型 + 对应 LoRA，实现真正的多业务复用。
- 原生支持投机采样（Speculative Decoding），小模型预测 + 大模型并行验证，推理延迟显著降低。

---

## 🛠️ 快速启动与 API 兼容

### 1. 启动兼容 OpenAI 规范的 HTTP 服务
```bash
# 安装
pip install vllm

# 单卡启动例如 Qwen2.5-7B 或 DeepSeek
python3 -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen2.5-7B-Instruct \
    --port 8000 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192
```

### 2. 客户端调用
由于完全兼容 OpenAI 接口规范，已有项目无需修改业务代码，仅需变更 `base_url`：

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="token-not-needed",
)

chat_completion = client.chat.completions.create(
    model="Qwen/Qwen2.5-7B-Instruct",
    messages=[{"role": "user", "content": "你好，请介绍一下你自己。"}],
    temperature=0.7,
)
print(chat_completion.choices[0].message.content)
```

---

## 💡 生产落地踩坑经验与选型建议

> [!TIP] 生产参数调优建议
> 1. `--gpu-memory-utilization`：默认 0.90。如果显存充裕且追求超高并发，可适当调至 0.95；但若机器上有其他辅助进程（如 Embedding 模型），请调低至 0.75~0.80 避免 OOM。
> 2. `--tensor-parallel-size`：多卡部署时设置（例如 4 卡设为 4）。注意单机跨卡走 NVLink 性能最佳，PCIe 跨卡通信开销会降低高吞吐优势。
> 3. 量化选择：生产环境强烈推荐 **AWQ (Activation-aware Weight Quantization)** 或 **FP8**，在几乎无损精度下可将显存占用砍半，推理速度提升 1.5~2 倍。
