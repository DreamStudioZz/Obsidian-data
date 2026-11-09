---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/LMCache/LMCache
stars: "2.5k+"
license: Apache-2.0
date_added: 2026-11-09
---

# 📦 LMCache

> **一句话简介**：面向超长上下文大模型推理的开源分布式多级 KV Cache 共享与持久化卸载中枢（Open-Source Distributed Multi-Tier KV Cache Sharing & Offloading Engine）。由 UC Berkeley、芝加哥大学联合团队主导研发，彻底打破传统推理引擎（vLLM、SGLang 等）中 KV Cache 受限于单卡 GPU 显存容量与单机隔离的物理屏障。通过实现 GPU 显存、主机内存（CPU DRAM）、本地 NVMe 固态硬盘与远程分布式存储（Redis / MinIO / S3）的无缝分层卸载，并支持跨请求、跨实例、跨节点的 KV Cache 毫秒级复用，将复杂多轮 Agent、长文档 RAG 与巨型 Prompt 的首字延迟（TTFT）降低 3~10 倍，节约 80% 以上的前缀计算算力。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: LMCache/LMCache](https://github.com/LMCache/LMCache) |
| **研发机构** | UC Berkeley、University of Chicago 联合研究团队与开源社区 |
| **技术栈** | Python, C++, PyTorch, CUDA, Redis, vLLM / SGLang 生态扩展 |
| **许可证** | Apache-2.0 License |
| **核心机制** | 多级分层存储（GPU VRAM -> CPU DRAM -> NVMe Disk -> Remote S3/Redis）、Chunked Token-Level KV 序列化与哈希寻址、跨实例 KV 广播与点对点拉取、非前缀 KV Cache 融合复用（KV Blending） |
| **推理引擎集成** | 原生深度集成 vLLM（通过 `KVTransferConfig`、`LMCacheConnectorV1` 及 `LMCacheMPConnector`）、兼容 SGLang、支持 Prefill-Decode 分离部署架构 |
| **典型性能收益** | 128k 超长上下文场景下 Prefill 计算降低 85%+，跨 Pod 副本首字延迟（TTFT）从数十秒压缩至几百毫秒 |
| **相关关联** | 关联实践: [[企业级大模型长文本推理加速：基于 LMCache 的分布式多级 KV Cache 共享与跨节点复用实战]], [[2026-11-09-AI-Digest]], [[vLLM]], [[SGLang]], [[FlashInfer]], [[KTransformers]], [[Prompt Caching 架构设计与最佳实践]], [[Text-Embeddings-Inference]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph ClientLayer [客户端请求层 (Multi-Tenant & Agent Requests)]
        Req1["请求 A (用户 1: 包含 64k 企业文档 RAG)"]
        Req2["请求 B (用户 2: 相同文档 + 追问新指令)"]
    end

    subgraph ServingCluster [分布式推理集群 (vLLM / SGLang Cluster)]
        subgraph Pod1 [Worker Pod 1]
            Engine1[vLLM Serving Engine 1]
            GPU1[GPU VRAM: Active KV Cache]
            LMCacheClient1[LMCache In-Process Connector]
        end

        subgraph Pod2 [Worker Pod 2]
            Engine2[vLLM Serving Engine 2]
            GPU2[GPU VRAM: Zero-Compute Hit]
            LMCacheClient2[LMCache In-Process Connector]
        end
    end

    subgraph LMCacheStorageTiers [LMCache 分层存储抽象层 (Hierarchical Storage Engine)]
        Tier1["Level 1: Local Host Memory (CPU DRAM / Zero-Copy DMA)<br/>延迟: < 1ms | 吞吐: 几十 GB/s"]
        Tier2["Level 2: Local NVMe SSD (High-Speed Direct I/O)<br/>延迟: 5~15ms | 容量: 数 TB 级单机持久化"]
        Tier3["Level 3: Distributed Remote Store (Redis / S3 / MinIO / Mooncake)<br/>延迟: 20~50ms | 共享: 全局集群跨 Pod 共享"]
    end

    Req1 ==> Engine1
    Engine1 --> GPU1
    Engine1 -.->|Prefill 完成异步保存 KV| LMCacheClient1
    LMCacheClient1 ==> Tier1
    Tier1 -.->|驱逐降级| Tier2
    Tier1 -.->|集群广播/异步上浮| Tier3

    Req2 ==> Engine2
    Engine2 -.->|前缀哈希命中探测| LMCacheClient2
    LMCacheClient2 ==>|远程极速拉取并直接加载至显存| Tier3
    Engine2 --> GPU2
```

1. **从“计算换显存”向“分层外存换延迟”的架构跃迁**：
   - 传统大模型推理中，单张 A100/H800 的显存极大被动态增长的 KV Cache 蚕食。当并发爆发或处理 128k~1M 超长文本时，GPU 显存瞬间见顶，导致系统不得不频繁驱逐旧 Cache，使得后续追问重新触发长耗时的 Prefill（二次计算）。
   - LMCache 将 KV Cache 从紧密的 GPU 显存依赖中解耦，抽象为独立的分布式知识缓存层。当显存紧张时，自动将已就绪的 KV Cache 无损写出到主机内存（CPU RAM）甚至 NVMe 磁盘；后续请求到来时，通过高带宽 PCIe DMA 与零拷贝技术秒级还原回 GPU，以极小 I/O 代价规避庞大的 GPU 矩阵计算。
2. **跨实例、跨副本的分布式共享机制（Cross-Instance Cache Sharing）**：
   - 在主流 Kubernetes 集群中，为了应对高并发，通常配置多个 vLLM 副本并由负载均衡器分发流量。传统 vLLM 的 RadixAttention / APC（Automatic Prefix Caching）只能在单个进程的本卡显存内生效；一旦请求被轮询调度到 Pod 2，Pod 1 算过的几万 Token 前缀依然要在 Pod 2 重新算一遍。
   - LMCache 实现了基于全局哈希寻址的分布式缓存同步层。Pod 1 计算完长文档前缀后，异步存入共享存储（如 Redis / Ceph / S3 或专有 RDMA 内存池）；Pod 2 接到相关请求时，直接拉取现成的 KV 张量，实现真正的**集群级零计算冷启动**。
3. **Chunk-Level 动态切片与哈希索引**：
   - 将超长 Token 序列按照固定 Chunk 大小（如 256 或 512 Token）进行分片并计算哈希值（Prefix Hash Chain）。这一机制确保了即使输入文档发生微小改动（如修改了末尾几行指令），前面绝大部分匹配的 Chunk 仍能精准命中并复用，避免整段缓存雪崩。
4. **突破性的 KV Blending（非连续前缀融合技术）**：
   - 传统 Prefix Caching 严格要求新请求的 Prompt 必须拥有“完全一致的前缀”。但在高级 RAG 场景中，文档片段的排列顺序可能发生变化（例如多篇召回文档由 Reranker 重新排序）。LMCache 支持创新的 KV 融合机制，允许对非连续匹配的上下文片段进行位置重映射与 KV 拼接，大幅拓宽缓存复用的边界。

---

## 🛠️ 快速上手与配置

### 1. 环境安装

LMCache 提供了与 vLLM 深度集成的适配器。可以通过 pip 直接安装核心库：

```bash
# 建议在包含 CUDA 12.1+ 的 PyTorch 2.3+ 环境中安装
pip install lmcache lmcache-vllm
```

### 2. 配置文件声明 (`lmcache.yaml`)

创建 LMCache 的存储分层配置文件，指定本地 CPU 内存、NVMe 固态硬盘与远程共享后端的配额：

```yaml
# lmcache.yaml: 分布式多级 KV Cache 策略配置
chunk_size: 256                      # 缓存切片粒度 (Tokens)
local_cpu: true                      # 启用宿主机 CPU DRAM 缓存
local_cpu_memory_gb: 64              # 分配 64GB CPU 内存充当 L1 缓存
local_disk: "/data/lmcache_nvme"     # 启用 NVMe SSD 本地磁盘充当 L2 缓存
local_disk_size_gb: 500              # 分配 500GB 磁盘持久化空间
remote_url: "redis://10.0.0.88:6379" # 启用分布式共享后端 (可选: redis://, minio://)
remote_serde: "torch"                # 序列化格式: torch / safetensors
save_decode_cache: false             # 默认仅缓存 Prefill 阶段的高价值上下文
```

### 3. 与 vLLM 配合启动服务

通过设置环境变量与指定 vLLM 的 `kv_transfer_config`，无缝挂接 LMCache 引擎：

```bash
export LMCACHE_CONFIG_FILE="./lmcache.yaml"

# 启动单机 vLLM 实例（内置接入 LMCache 进程内连接器）
vllm serve Qwen/Qwen2.5-72B-Instruct \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --gpu-memory-utilization 0.90 \
    --kv-transfer-config '{"kv_connector":"LMCacheConnectorV1","kv_role":"kv_both"}'
```

```python
# 客户端调用验证：首字延迟 (TTFT) 极速测试
import time
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

# 模拟 32k 长度的共享知识库前缀
shared_prefix_prompt = "【企业核心知识库规章制度 V2026.11】\n" + ("数据治理与网络安全规范细则条款...\n" * 1500)
user_query = "\n请问针对三级安全事件的响应时限是多久？"

full_prompt = shared_prefix_prompt + user_query

# 第一次请求：冷启动，执行 Prefill 并自动写出 KV Cache 至 LMCache
t0 = time.time()
resp1 = client.chat.completions.create(
    model="Qwen/Qwen2.5-72B-Instruct",
    messages=[{"role": "user", "content": full_prompt}],
    stream=True
)
first_token_time1 = None
for chunk in resp1:
    if chunk.choices[0].delta.content and first_token_time1 is None:
        first_token_time1 = time.time() - t0
        break
print(f"👉 首次请求冷启动 TTFT: {first_token_time1:.2f} 秒")

# 第二次请求（换一个追问指令）：命中 LMCache，从 CPU 内存/SSD 极速加载 KV，免 Prefill
t0 = time.time()
resp2 = client.chat.completions.create(
    model="Qwen/Qwen2.5-72B-Instruct",
    messages=[{"role": "user", "content": shared_prefix_prompt + "\n请问五级合规违规如何处罚？"}],
    stream=True
)
first_token_time2 = None
for chunk in resp2:
    if chunk.choices[0].delta.content and first_token_time2 is None:
        first_token_time2 = time.time() - t0
        break
print(f"⚡ 二次请求命中 LMCache TTFT: {first_token_time2:.2f} 秒 (提速超 {(first_token_time1/first_token_time2):.1f} 倍!)")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **企业私有化长文档 RAG 与多轮法律/研报问答**：同一篇长达数万字的年报或技术规范，被企业内多个员工反复咨询，LMCache 可将除第一次之外的全部 Prefill 耗时彻底抹平；
  2. **多智能体协作与复杂 Codebase 分析**：在多 Agent 协同循环中，庞大的系统提示词（System Prompt）、MCP 工具描述定义和完整代码工程上下文每轮都需要带入，LMCache 可实现跨 Agent 节点极速复用；
  3. **大规模多副本弹性 Kubernetes 推理集群**：解决客户端请求被随机路由到不同 Pod 时导致的重复计算浪费，实现全集群全局共享的“统一神经记忆层”。
- **潜在不足与局限性**：
  - **网络与 PCIe 传输带宽瓶颈**：在极大规模模型（如 70B+ FP16）下，KV Cache 自身体积较大，若节点间网络带宽不足（如低于 25Gbps），从远程 Redis/S3 传输 KV 张量的耗时可能会稀释计算节省的红利；推荐搭配本地 NVMe SSD 与 100G RDMA 网络部署；
  - **序列长度对齐与位置编码依赖**：部分基于相对位置编码（RoPE）的模型在对非前缀 Chunk 拼接时可能存在数值敏感性，需确保模型与 LMCache 切片算法完全兼容。
- **个人评估结论**：**大模型推理进入长文本时代的必备底座，强烈建议在多副本 vLLM 生产集群中全量引入评估**。
