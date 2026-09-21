---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/sgl-project/sglang
stars: "15k+"
license: Apache-2.0
date_added: 2026-09-21
---

# 📦 SGLang

> **一句话简介**：由伯克利 LMSYS 团队主导研发的高性能 LLM & VLM 推理引擎，首创 **RadixAttention（基数树前缀缓存）**，大幅加速多轮对话、RAG 与复杂 Agent 分支推理。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: sgl-project/sglang](https://github.com/sgl-project/sglang) |
| **核心技术** | RadixAttention, FlashInfer, PagedAttention, Continuous Batching |
| **许可证** | Apache-2.0 |
| **关联概念** | [[vLLM]], [[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]], [[2026-09-21-AI-Digest]] |

---

## 🚀 核心架构与杀手级特性

### 1. RadixAttention（自动前缀 KV 缓存复用）
- **传统困境**：在多轮对话（Chat）、Few-shot 示例、Agent 工具调用和长文档 RAG 场景中，大量请求的前半部分 Prompt（System Prompt、历史消息、参考资料）是完全相同或高度重叠的。传统推理框架（如早期 vLLM）对每个请求都要重复计算前缀的 KV Cache。
- **RadixAttention 突破**：
  - 将显存中的 KV Cache 维护为一颗**基数树（Radix Tree）**。
  - 当新请求到来时，自动匹配基数树中最长公共前缀，**直接命中并复用已存在的 KV Cache**，无需任何前向计算。
  - 结合 LRU 缓存淘汰机制，显存不足时自动释放最久未访问的分支。
  - **效果**：首字生成时间（TTFT, Time-To-First-Token）降低 **5x ~ 15x**，吞吐量翻倍。

### 2. SGLang 编程语言前端（Structured Generation Language）
- 允许开发者以类似代码函数的方式编写复杂大模型逻辑（如包含多轮分支、正则表达式强制 JSON 约束提取、循环自检等），并在后端实现自动批处理和编译级优化。

### 3. 多模态与前沿大模型原生优化
- 对 DeepSeek-V3 / R1、Qwen2.5-VL、Llama 3 等主流模型具备高度优化的内核级加速。

---

## 🛠️ 快速启动示例

```bash
# 安装
pip install "sglang[all]"

# 启动兼容 OpenAI 规范的本地高性能推理服务
python3 -m sglang.launch_server \
    --model-path Qwen/Qwen2.5-7B-Instruct \
    --port 30000 \
    --host 0.0.0.0 \
    --context-length 32768
```

---

## 💡 vLLM vs SGLang 选型建议

| 场景维度 | vLLM | SGLang |
| :--- | :--- | :--- |
| **生态成熟度与企业级周边** | ⭐⭐⭐⭐⭐（老牌稳健、周边监控/K8s完善） | ⭐⭐⭐⭐（快速演进、LMSYS背书） |
| **长前缀复用 / RAG / 复杂 Agent** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐（RadixAttention 性能天花板） |
| **多模态视觉大模型 (VLM)** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐（针对多图混合场景优化极佳） |
| **结构化输出 (JSON / Regex)** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐（内置前端编译器） |
