---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/xorbitsai/inference
stars: "6.5k+"
license: Apache-2.0
date_added: 2026-10-21
---

# 📦 Xinference

> **一句话简介**：Xorbits 团队开源的**工业级分布式异构模型统一推理与多模态服务底座**。专为企业私有化算力集群设计，打破了单一模型推理引擎的割裂孤岛，一站式将大语言模型（LLM）、高维稠密/稀疏向量嵌入（Embedding）、语义重排（Reranker）、语音转录（Whisper）以及图像生成（Diffusers）全栈矩阵纳入统一管理。底层原生集成 vLLM、SGLang、llama.cpp、Transformers 等多种极致加速后端，向上提供 100% 兼容 OpenAI 标准的分布式高可用 API 网关。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: xorbitsai/inference](https://github.com/xorbitsai/inference) |
| **技术栈** | Python 3.9+ / Pydantic / PyTorch / CUDA / vLLM / llama.cpp / SGLang / FastAPI |
| **许可证** | Apache-2.0 |
| **核心特性** | 异构多引擎自适应后端（vLLM/llama.cpp/Transformers）、全栈模型矩阵（LLM+Embedding+Rerank+Audio+Image）、分布式主从集群架构（Supervisor-Worker）、动态显存调度与权重多级缓存、双源权重直连（HuggingFace / ModelScope） |
| **相关关联** | 关联实践: [[分布式异构模型统一服务化架构：基于 Xinference 的多后端协同调度与一站式多模态推理网关实战]], [[2026-10-21-AI-Digest]], [[vLLM]], [[SGLang]], [[llama.cpp]], [[Text-Embeddings-Inference]], [[LiteLLM]], [[Ollama]] |

---

## 🚀 核心特性与架构亮点

### 1. 异构多推理后端自适应编排（Multi-Backend Orchestration）
在传统的私有化模型部署中，运维团队通常陷入严重的碎片化灾难：部署大语言模型需要起一套 vLLM 或 SGLang 容器；部署 Embedding 和 Reranker 需要起一组 Triton 或 TEI 进程；在边缘设备或消费级显卡上又需要维护 llama.cpp。
- **动态后端绑定**：Xinference 将业界顶尖的底层推理库抽象为标准化插件。用户启动模型时，框架可根据硬件条件（NVIDIA GPU / Apple Silicon / CPU）自适应挑选最快引擎；
- **免除底层胶水代码**：无论底层采用 vLLM 的 PagedAttention 还是 llama.cpp 的 GGUF 极限量化，上层业务均享受一致的流式接口与参数规格。

### 2. 全栈全模态模型矩阵（Full-Stack AI Serving Matrix）
绝大多数开源推理网关仅专注纯文本 LLM，而在真实的企业级 RAG 与 Agent 系统中，完整的 AI 基础设施必须包含多种辅助模型：
- **生成式 LLM**：全系列支持 Qwen2.5、DeepSeek-V3/R1、Llama-3.3、Mistral、GLM-4 等；
- **向量与稀疏嵌入（Embedding）**：支持 BAAI/bge-m3、bge-large-zh、text2vec 等，满足密集与词汇稀疏检索；
- **重排序模型（Reranker）**：原生提供 `/v1/rerank` 接口，支持 BGE-Reranker-Large、bge-reranker-v2-m3 等多语言高精度重排器；
- **多模态与语音图像**：原生支持 Whisper 语音转录（`/v1/audio/transcriptions`）与 Stable Diffusion / Flux 生图管道。

### 3. 分布式高可用主从拓扑（Supervisor-Worker Architecture）
Xinference 采用工业级的分布式控制面与数据面分离设计：
- **管理中枢（Supervisor）**：负责集群元数据维护、节点心跳检测、用户鉴权与全局路由；向上暴露标准 RESTful API 与 WebUI；
- **工作节点（Worker）**：部署在具体的物理 GPU 机器上，负责具体的显卡分配、显存池化隔离、模型动态拉起与执行；
- **自动负载均衡与高可用容灾**：当多节点部署同名模型副本时，Supervisor 自动进行加权轮询分发；若某个 Worker 宕机，调度器秒级将流量平滑漂移至健康节点。

### 4. 国内外主流权重源开箱直通（ModelScope / HuggingFace 无缝集成）
- 针对国内网络环境下 HuggingFace 访问受限的痛点，Xinference 原生内置对 **ModelScope（魔搭社区）** 的一键权重直连拉取；
- 支持本地多级缓存（Local Cache），避免重复下载数百 GB 的模型权重，并支持挂载企业内网 NAS 共享存储。

---

## 🛠️ 快速上手与典型代码示例

### 1. 依赖安装与集群启动

```bash
# 安装全功能版本（包含 vLLM 与 GPU 加速依赖）
pip install "xinference[all]"

# 启动单机一站式服务（开发调试模式，默认端口 9997）
xinference-local --host 0.0.0.0 --port 9997
```

若在生产分布式多节点集群中运行：
```bash
# 节点 A (控制节点): 启动 Supervisor
xinference-supervisor -H 192.168.1.10 -p 9997

# 节点 B (GPU 节点 1): 启动 Worker 挂载显卡
xinference-worker -e "http://192.168.1.10:9997" -H 192.168.1.11 -p 9998 --gpu-idx 0,1,2,3

# 节点 C (GPU 节点 2): 启动 Worker 挂载显卡
xinference-worker -e "http://192.168.1.10:9997" -H 192.168.1.12 -p 9998 --gpu-idx 0,1,2,3
```

### 2. 通过 CLI 极速拉起 LLM、Embedding 与 Reranker 模型全家桶

```bash
# 1. 启动 Qwen2.5-7B 大语言模型（使用 vLLM 高并发加速引擎）
xinference launch --model-name qwen2.5-instruct --size-in-billions 7 --model-format pytorch --model-engine vLLM

# 2. 启动 BGE-M3 全能多语言向量模型
xinference launch --model-name bge-m3 --model-type embedding

# 3. 启动 BGE-Reranker-Large 重排模型
xinference launch --model-name bge-reranker-large --model-type rerank
```

### 3. 基于 Python 官方客户端与 OpenAI SDK 对接调用

```python
from xinference.client import Client
from openai import OpenAI

# 1. 使用 Xinference 原生客户端管理模型生命周期
client = Client("http://localhost:9997")

# 列出当前集群运行的所有模型实例
running_models = client.list_models()
print("当前集群运行模型:", running_models)

# 2. 获取具体模型的 OpenAI 兼容调用端点
openai_client = OpenAI(
    base_url="http://localhost:9997/v1",
    api_key="empty"  # 私有集群默认无需认证或填入自定义 Token
)

# 3. 统一调用大模型生成
response = openai_client.chat.completions.create(
    model="qwen2.5-instruct",
    messages=[
        {"role": "system", "content": "你是一个严谨的微服务架构专家。"},
        {"role": "user", "content": "请简述高并发场景下的降级与熔断机制。"}
    ],
    temperature=0.3,
    stream=False
)
print("模型输出:", response.choices[0].message.content)

# 4. 统一调用向量嵌入（开箱即用兼容 text-embedding 协议）
emb_resp = openai_client.embeddings.create(
    model="bge-m3",
    input=["微服务架构设计与高可用", "分布式数据库事务解析"]
)
print("生成向量维度:", len(emb_resp.data[0].embedding))

# 5. 调用原生语义重排接口
rerank_model = client.get_model("bge-reranker-large")
rerank_result = rerank_model.rerank(
    corpus=[
        "熔断机制在下游服务异常时快速切断流量保护自身",
        "数据库分库分表是处理海量数据的核心方案",
        "限流可以通过令牌桶或漏桶算法实现流量整形"
    ],
    query="服务不可用时如何防止级联雪崩？",
    top_n=2
)
print("重排结果:", rerank_result)
```

---

## 💡 工程实战点评与适用场景

### 1. 推荐使用场景
- **企业私有算力中台与多模型资产统一纳管**：当企业拥有数十张 A800/H800/4090 显卡，需要同时对外提供 LLM、Embedding、Reranker、Whisper 全套 AI 能力时，Xinference 是目前整合度最高的一站式解决方案；
- **生产级混合检索 RAG 底座构建**：在企业本地知识库中，通常需要“向量检索 + BM25 稀疏检索 + 语义重排”三段式流水线，Xinference 可单集群无缝提供 Embedding 与 Rerank，彻底免除运维维护多套异构镜像的痛苦；
- **异构算力环境与模型按需热切换**：在算力紧缺或需兼顾开发测试环境时，利用 Xinference 的动态模型启动与下线（Eviction）机制，实现白天提供 LLM 推理、夜间自动释放显存运行批量 Embedding 与微调。

### 2. 潜在不足与避坑考量
- **超大规模集群调度能力仍有提升空间**：与 Kubernetes 原生的 KServe 或 Ray Cluster 相比，Xinference 的内置调度器在超百节点弹性扩缩容、Pod 自动迁移等复杂网格场景中相对轻量，超大规模数据中心建议通过 Kubernetes Operator 结合部署；
- **多模型显存争抢风险**：单卡部署多个小模型（如同时拉起 Embedding 与 Rerank）时，需精细化配置 `gpu_memory_utilization` 参数，防止 CUDA Out Of Memory 引发 Worker 异常崩溃。

### 3. 个人评估结论
- **建议引入（Adopted）**。Xinference 完美填补了“单一内核引擎（如纯 vLLM）”与“企业业务网关”之间的巨大鸿沟。其对多模型类型（生成/向量/重排/语音）的完备覆盖、对国内生态（ModelScope）的友好支持以及优雅的分布式架构，使其成为当之无愧的私有化 AI 算力服务化基石。
