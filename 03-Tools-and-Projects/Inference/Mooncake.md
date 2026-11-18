---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/kvcache-ai/Mooncake
stars: "3.5k+"
license: Apache-2.0
date_added: 2026-11-18
---

# 📦 Mooncake

> **一句话简介**：Moonshot AI (月之暗面 / Kimi) 联合 KVCache.AI 开源的高性能、以 KV Cache 为中心的解耦式（Disaggregated）大模型推理与服务架构底座。专为长上下文（Long-Context）场景量身打造，首创 Prefill-Decode 物理节点异构解耦、跨节点多级异构存储（GPU HBM / Host DRAM / NVMe SSD）分层池化调度，以及基于 RDMA/GPUDirect 的超高吞吐传输引擎（Transfer Engine），从根本上打破传统推理的“显存墙”与计算通信资源错配瓶颈。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake) |
| **研发机构** | Moonshot AI (月之暗面) & KVCache.AI 研发团队 |
| **技术栈** | C++20, CUDA 12.0+, Python, RDMA / RoCEv2, TCP, libverbs |
| **许可证** | Apache-2.0 License |
| **核心机制** | Prefill-Decode 实例物理分离（Disaggregated Prefill-Decoding）、分布式 KV Cache 池化中枢（Mooncake Store）、多级存储智能置换调度（GPU HBM -> Host DRAM -> NVMe）、超高速跨节点数据传输引擎（Transfer Engine，支持 GPUDirect RDMA 零拷贝）、前缀缓存全局感知与内容寻址哈希（Content-Addressed Prefix Caching） |
| **适用生态** | 深度集成并原生赋能主流推理引擎：[[vLLM]], [[SGLang]], [[LMDeploy]], [[TensorRT-LLM]], TokenSpeed 等 |
| **硬件支持** | NVIDIA GPU (Ampere/Hopper/Blackwell 架构，如 A100/A800/H100/H800/H20 等)，支持 RoCEv2 / InfiniBand 高速 RDMA 网卡集群，亦支持通用以太网 TCP 兜底 |
| **相关关联** | 关联实践: [[大规模长文本大模型解耦推理架构：基于 Mooncake 的 Prefill-Decode 异构分离、多级 KV Cache 池化与 RDMA 零拷贝传输实战]], [[2026-11-18-AI-Digest]], [[LMCache]], [[vLLM]], [[SGLang]], [[KTransformers]], [[FlashInfer]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph ClientLayer [客户端请求入口]
        Request["用户长文本请求<br/>(128k ~ 1M Context Prompt)"]
    end

    subgraph MooncakeArchitecture [Mooncake 总体解耦推理中枢架构]
        subgraph Gateway [智能全局调度网关 (Mooncake Gateway)]
            PrefixRouter["全局 Prefix Cache 命中感知与负载均衡路由"]
        end

        subgraph PrefillCluster [Prefill 计算密集型实例集群 (算力优先)]
            P_Node1["Prefill GPU Worker 1<br/>(专注 Prompt 首次吞吐计算)"]
            P_Node2["Prefill GPU Worker 2<br/>(Chunked Prefill 并行加速)"]
        end

        subgraph TransferEngineMesh [Transfer Engine: 高速数据传输网络]
            RDMA_Mesh["GPUDirect RDMA / RoCEv2 高速零拷贝通信拓扑 (400Gbps)"]
        end

        subgraph StoragePool [分布式跨节点多级 KV Cache 资源池 (Mooncake Store)]
            HBM_Pool["L1: 局部 GPU HBM 显存"]
            DRAM_Pool["L2: 分布式 Host DRAM 内存池"]
            SSD_Pool["L3: 本地 NVMe 高速固态阵列"]
        end

        subgraph DecodeCluster [Decode 访存带宽密集型实例集群 (吞吐优先)]
            D_Node1["Decode GPU Worker 1<br/>(微批次自回归生成，无计算扰动)"]
            D_Node2["Decode GPU Worker 2<br/>(极高并发，TTFT 稳定)"]
        end
    end

    subgraph FinalOutput [流式生成输出]
        StreamingResponse["低延迟流式响应 (Time-to-First-Token < 500ms)"]
    end

    Request ==> PrefixRouter
    PrefixRouter -. 命中历史缓存 .-> StoragePool
    PrefixRouter ==> PrefillCluster
    PrefillCluster ==>|生成新 KV Cache| RDMA_Mesh
    RDMA_Mesh ==>|零拷贝直存| StoragePool
    StoragePool ==>|预取就绪 KV Cache| D_Node1 & D_Node2
    D_Node1 & D_Node2 ==> StreamingResponse
```

1. **破除传统单体推理（Monolithic Serving）死穴，首创生产级 Prefill-Decode 分离**：
   - 在传统的合设部署（Monolithic）中，Prefill（首字计算）是**计算密集型**（Compute-bound），需要大 Batch、高张量并行榨干算力；而 Decode（逐字自回归）是**访存密集型**（Memory-bandwidth bound），极易受到 Prefill 任务的计算扰动与显存挤占，导致首 Token 延迟（TTFT）剧烈抖动，并发吞吐断崖下跌。
   - Mooncake 将两者物理拆分：Prefill 节点专注极速计算长 Prompt 生成 KV Cache，随即通过 RDMA 高速流式推送至共享池或 Decode 节点，Decode 节点仅需专心进行极度平滑的自回归推理，双方算力利用率同时提升 50%~100%。
2. **多级异构存储解耦池化（Disaggregated KV Cache Pool）**：
   - 充分榨取集群中原本闲置、廉价的 Host CPU DRAM 内存和 NVMe SSD 资源。Mooncake 将各 GPU 节点的空闲内存汇聚成一张巨大的分布式虚拟 KV 缓存池。
   - 实现 **L1 (GPU HBM) -> L2 (Host DRAM) -> L3 (NVMe SSD)** 三级分级自适应换入换出。即便长达 1M 的上下文 KV Cache，也无需时刻常驻昂贵脆弱的 GPU 显存，显存利用率由被动受限转向完全弹性。
3. **超高性能低延迟传输引擎（Transfer Engine）**：
   - 依托自主研发的高并发传输引擎，原生支持 InfiniBand、RoCEv2、GPUDirect RDMA（GDR）和多网卡带宽聚合（Multi-NIC Striping）。
   - 绕过 CPU 内存中转与上下文切换，实现 GPU 到 GPU、GPU 到远程 Host 内存的百 Gbps 级硬件直传，将百兆字节级长上下文 KV Cache 的跨节点转移延迟压低至数毫秒内，彻底消除 Prefill 与 Decode 间的传输气泡。
4. **统一命名与全局内容寻址（Mooncake Store）**：
   - 引入全局分布式对象存储模型，针对模型层级结构与 Block-based KV Cache 实施统一哈希索引与分片切块（Object Striping）。
   - 全局多实例共享 System Prompt、角色设定、长文档资料等历史公共前缀（Prefix Cache），全局缓存复用率提升 60% 以上，显著减少海量重复 Prefill 算力浪费。
5. **主流生态无缝插拔支持**：
   - 不仅是独立的底层系统，Mooncake 还向开源生态提供极度干净的抽象接口与集成中间件，开箱即用适配 [[vLLM]]、[[SGLang]]、[[LMDeploy]] 等各大主流推理加速框架，使企业无需推翻现有推理栈即可平滑升级为万卡级长文本解耦集群。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

Mooncake 需要 Linux 环境、CUDA 12.0+、支持 C++20 的编译环境（GCC 11+ / Clang 14+），推荐配备 Mellanox RoCEv2/IB 网卡：

```bash
# 从源码克隆并初始化子模块
git clone --recursive https://github.com/kvcache-ai/Mooncake.git
cd Mooncake

# 构建 C++ 核心库与 Transfer Engine
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DUSE_CUDA=ON -DUSE_RDMA=ON
make -j$(nproc)

# 安装 Python 绑定接口
cd ../python
pip install -e .
```

### 2. 启动分布式 Mooncake Store 与 Transfer Engine 节点

配置简单的 JSON/YAML 集群拓扑配置文件 `mooncake_cluster.json`：

```json
{
  "topology": {
    "prefill_nodes": [
      {"id": "prefill-01", "ip": "192.168.10.11", "rdma_device": "mlx5_0", "port": 18888},
      {"id": "prefill-02", "ip": "192.168.10.12", "rdma_device": "mlx5_0", "port": 18888}
    ],
    "decode_nodes": [
      {"id": "decode-01", "ip": "192.168.10.21", "rdma_device": "mlx5_0", "port": 18889},
      {"id": "decode-02", "ip": "192.168.10.22", "rdma_device": "mlx5_0", "port": 18889}
    ],
    "storage_pool": {
      "dram_cache_per_node_gb": 256,
      "nvme_cache_per_node_gb": 1024,
      "enable_gdr": true
    }
  }
}
```

### 3. 与 vLLM / SGLang 解耦集成调用示例

```python
import torch
from mooncake.engine import TransferEngine
from mooncake.store import MooncakeStore

# 1. 初始化 Mooncake 传输引擎与存储池
engine = TransferEngine(device_id=0, rdma_nic="mlx5_0")
store = MooncakeStore(engine=engine, cluster_config="mooncake_cluster.json")

# 2. Prefill 节点：完成长文本计算并零拷贝推送至分布式池
def process_prefill_and_publish(prompt_tokens: list[int], kv_cache: torch.Tensor):
    cache_key = store.compute_hash_key(prompt_tokens)
    print(f"[*] Prefill 完成，推送 KV Cache (大小: {kv_cache.nbytes / 1024**2:.2f} MB) 至全局池...")
    # 异步通过 GPUDirect RDMA 写入全局共享池
    future = store.put_async(key=cache_key, tensor=kv_cache)
    future.wait()
    return cache_key

# 3. Decode 节点：拉取就绪的 KV Cache 并直接启动解码
def fetch_and_decode(cache_key: str, decode_stream):
    print(f"[*] Decode 节点预取 KV Cache (Key: {cache_key[:12]}...)...")
    remote_tensor = store.get(key=cache_key, device=torch.device("cuda:0"))
    # 直接挂载至本地 PagedAttention 显存池，进入低延迟自回归解码
    return remote_tensor
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **超长上下文业务场景**：长篇文档总结、全库代码研读、长视频多模态理解（128k~1M Token），单机显存无法承受并发 KV Cache 膨胀的企业级生产集群；
  - **高并发企业级对话中枢**：需要严格保障首 Token 延迟（TTFT < 800ms）同时维持极致高并发（Concurrency > 1000 requests/s）的云服务与网关；
  - **异构算力混合集群**：希望将价格高昂的先进 GPU（如 H100/H800）集中用于 Prefill 计算，而将成本较低的通用 GPU（如 L40S/A10/A800）编为 Decode 节点降本增效的场景。
- **潜在不足 / 局限性**：
  - **网络硬件门槛较高**：要获得最佳性能（亚毫秒级传输与零 CPU 负载），强依赖 InfiniBand 或无损 RoCEv2 交换机配置及 GPUDirect RDMA 环境，普通千兆/万兆以太网 TCP 模式下吞吐优势会打折；
  - **系统运维与调度复杂度上升**：相比传统单体部署，Prefill 与 Decode 解耦需要更精密的前置全局调度网关处理负载动态平衡与连接容错。
- **个人评估结论**：**极度推荐落地（status/adopted）**。Mooncake 代表了大模型长文本推理基建从“单机硬件死扛”走向“分布式存算解耦”的必经之路，是解决 Kimi、DeepSeek 等万亿参数与百万上下文落地服务的最核心基础设施之一。
