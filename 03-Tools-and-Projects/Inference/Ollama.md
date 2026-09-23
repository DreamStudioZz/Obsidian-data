---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/ollama/ollama
stars: "135k+"
license: MIT
date_added: 2026-09-23
---

# 📦 Ollama

> **一句话简介**：本地大模型运行与打包部署的“Docker 级”事实标准工具，将底层繁琐的 llama.cpp、CUDA 驱动、量化权重与模型管理封装为极致优雅的 CLI 与标准 OpenAI 兼容 REST API。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: ollama/ollama](https://github.com/ollama/ollama) |
| **底层引擎** | llama.cpp (GGUF 格式), 原生跨平台加速 (macOS Metal / Linux CUDA / Windows ROCm) |
| **核心特点** | 零配置启动、Modelfile 声明式定制、CPU/GPU 动态卸载、标准 OpenAI 接口 |
| **关联实践** | [[私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构]], [[LiteLLM]], [[vLLM]], [[SGLang]] |

---

## 🚀 核心架构与创新亮点

1. **“Docker 级”简易体验**：
   - 传统本地运行 LLM 需要编译 C++ 代码、下载 HuggingFace 繁杂的分卷权重、手动指定量化参数；
   - Ollama 将全流程极简为一条命令：`ollama run llama3.2` 或 `ollama run qwen2.5-coder`，自动拉取、自动分配显存与 CPU 内存并即刻进入对话。
2. **声明式模型打包配置（Modelfile）**：
   - 类似于 Dockerfile，开发者可通过 `Modelfile` 自由定制基座模型、系统提示词（SYSTEM）、温度（temperature）、停止词（stop words）及上下文长度（num_ctx），并导出为独立可分发模型镜像：
     ```dockerfile
     FROM qwen2.5-coder:7b
     PARAMETER temperature 0.2
     PARAMETER num_ctx 32768
     SYSTEM """你是一名资深代码审查专家，必须用严格的规范输出安全漏洞与重构建议。"""
     ```
3. **原生兼容 OpenAI API 规范**：
   - 本地自动开启 `http://localhost:11434/v1` 服务，无缝兼容所有基于 OpenAI SDK、LangChain、OpenHands 或 Cursor 构建的应用，做到零代码改动平替商用 API。
4. **多模型共存与动态热加载/卸载**：
   - 显存不足时自动将冷模型置换出内存，支持多个模型按需热拉起与自动并发排队。

---

## 🛠️ 快速上手示例

### 1. 命令行拉取并运行
```bash
# 运行极速代码模型
ollama run qwen2.5-coder:7b

# 查看本地模型列表
ollama list
```

### 2. 通过标准 OpenAI Python SDK 调用本地 Ollama
```python
from openai import OpenAI

# 将 base_url 指向本地 Ollama 实例
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama" # 任意非空字符串
)

response = client.chat.completions.create(
    model="qwen2.5-coder:7b",
    messages=[
        {"role": "system", "content": "你是一名 Python 性能优化专家。"},
        {"role": "user", "content": "写一个线程安全的单例模式实现。"}
    ],
    temperature=0.2
)

print(response.choices[0].message.content)
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **开发环境与本地测试**：开发者笔记本或离线工作站快速体验开源模型；
  - **边缘计算与数据高度敏感场景**：医疗、金融、政企内网等不允许数据上云的场景；
  - **轻量级单机微服务**：搭配 LiteLLM 作为网关调度器构建团队私有推理集群。
- **与 vLLM / SGLang 对比**：
  - **Ollama**：适合个人与小微团队本地跑，资源门槛极低（支持 CPU/集显，量化压缩友好），开箱即用；
  - **vLLM / SGLang**：适合生产级高并发多卡 GPU 集群部署，支持 PagedAttention 和 RadixAttention 极端吞吐。
- **综合评估结论**：本地与边缘 LLM 运行时无可争议的第一基石，强烈推荐采纳（Adopted）。
