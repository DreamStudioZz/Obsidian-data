---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/deepseek-ai/3FS
stars: "10k+"
license: MIT
date_added: 2026-11-19
---

# 📦 3FS (Fire-Flyer File System)

> **一句话简介**：DeepSeek 官方开源的 AI 原生超高性能分布式文件系统（Fire-Flyer File System）。彻底打破传统分布式存储（Ceph、Lustre、NFS）在 AI 万卡集群下遭遇的元数据瓶颈与随机读吞吐墙，专为大规模 LLM 训练数据加载（Locality-oblivious Dataloader）、高并发秒级 Checkpointing 以及大模型长文本推理中的海量分布式 KV Cache 卸载（KV Cache Offload）量身打造。首创基于 FoundationDB 的无状态元数据服务、CRAQ 强一致性分摊查询存储链、USRBIO 用户态环形无锁 IO 以及 RoCEv2/InfiniBand RDMA 全链路零拷贝，在 180 节点标准配置下激发出高达 6.6+ TiB/s 的聚合读取带宽。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: deepseek-ai/3FS](https://github.com/deepseek-ai/3FS) |
| **研发机构** | DeepSeek (深度求索) 核心基础设施团队 |
| **技术栈** | C++20, Rust, Python, FoundationDB (FDB), libibverbs (RDMA), USRBIO (User Space Ring Based IO), FUSE / Native Client |
| **许可证** | MIT License |
| **核心机制** | 存算彻底分离（Disaggregated Storage-Compute）、CRAQ 强一致性链式复制与分摊查询（Chain Replication with Apportioned Queries）、基于 FoundationDB 的无状态事务元数据服务（Stateless Metadata Service）、USRBIO 用户态环形 IO 队列（绕过 Linux VFS 与 Page Cache 开销）、400Gbps RDMA 零拷贝直存、POSIX FUSE 兼容与 Native 高吞吐客户端 |
| **典型吞吐** | 单个存储节点读带宽打满 40~50 GB/s；180 节点集群聚合读取吞吐突破 **6.6 TiB/s**；单卡随机访问延迟低至微秒级 |
| **相关关联** | 关联实践: [[AI 原生超高吞吐分布式存储架构：基于 DeepSeek 3FS 的 CRAQ 强一致性、RDMA 零拷贝与多级 KV Cache 卸载实战]], [[2026-11-19-AI-Digest]], [[Mooncake]], [[LMCache]], [[FlashMLA]], [[DeepEP]], [[vLLM]], [[KTransformers]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph ComputeCluster [GPU 计算集群 (Training / Inference Nodes)]
        Worker1["GPU 01~08 (Dataloader / FlashMLA / vLLM)"]
        Worker2["GPU 09~16 (LMCache / Checkpointer)"]
        ClientRuntime["3FS Native Client / USRBIO Engine<br/>(零拷贝内存注册 / 直连 RDMA 网卡)"]
    end

    subgraph MetadataPlane [无状态元数据服务集群 (Metadata Cluster)]
        MDS1["Stateless MDS Node 1"]
        MDS2["Stateless MDS Node 2"]
        FDB["Apple FoundationDB 强一致分布式事务 KV 数据库"]
    end

    subgraph StorageCluster [3FS 存储节点集群 (Storage Nodes with NVMe Pools)]
        subgraph ChainReplication [CRAQ 链式存储通道 (Chain Replication)]
            Head["Head 存储节点 (处理写入)"]
            Middle["Mid 存储节点 (链路复制)"]
            Tail["Tail 存储节点 (版本确认 / 读分摊)"]
        end
        NVMePool["全 NVMe SSD 阵列 (PCIe 5.0 x4)"]
    end

    Worker1 & Worker2 <--> ClientRuntime
    ClientRuntime -- "1. 快速元数据操作 (open, stat, lookup)" --> MDS1 & MDS2
    MDS1 & MDS2 <--> FDB
    ClientRuntime == "2. 400G RoCEv2/IB RDMA 极速零拷贝数据读写 (绕过内核 VFS)" ==> ChainReplication
    ChainReplication <--> NVMePool
```

### 1. 突破传统分布式文件系统的 AI 原生设计哲学
传统 HPC 分布式文件系统（如 Lustre、GPFS、Ceph）大多建立在面向局部性缓存（Locality Caching）和预读取（Prefetching）的基础假设上。然而，在大模型生产环境中：
- **万卡并行随机读灾难**：在数万亿 Token 的预训练中，全球数千张 GPU 随机抽取不同批次样本，预读命中率几乎为 0，反而因无意义预读打爆存储网络带宽；
- **3FS 消除缓存假设（Cache-Oblivious Architecture）**：3FS 彻底放弃庞大的多层客户端文件缓存逻辑，将所有计算节点的数据请求直接、并发地平摊至后端成千上万块 NVMe SSD 上，充分榨干底层硬件的随机小块（Random 4K~64K）读写极限。

### 2. 基于 FoundationDB 的无状态元数据体系
元数据瓶颈（MDS Bottleneck）是导致大规模存储集群在大规模创建、读取海量小文件时性能雪崩的根源：
- 3FS 将元数据服务拆分为**纯无状态计算代理（Stateless MDS）**与底层**高可靠分布式事务引擎（FoundationDB）**；
- 无论客户端并发发起数百万次文件的打开、定位与目录检索，MDS 均可水平无限扩展，所有的元数据原子性、隔离性与版本控制完全由经过严格数学验证的 FoundationDB 事务处理，彻底根除了传统分布式文件系统单 MDS 宕机或锁竞争导致的卡死现象。

### 3. CRAQ 强一致性与线性读分摊（Chain Replication with Apportioned Queries）
针对分布式存储在追求高吞吐时容易牺牲一致性的难题，3FS 实现了 CRAQ 算法：
- **写入链路保障（Write-Through Chain）**：写操作从链头（Head）注入，依次流经链中节点，直到链尾（Tail）完成最终落盘确认；
- **读请求全节点平摊（Apportioned Read）**：不同于传统的主备模式中所有读请求集中在主节点，CRAQ 允许客户端从链中的任意存储节点读取稳定版本的数据，将全集群 NVMe 吞吐利用率最大化拉满，实现近乎线性的读性能扩展。

### 4. USRBIO 与全链路 RDMA 零拷贝引擎
- **绕过操作系统内核与 VFS**：传统文件读取在 Linux 内部需要经历 VFS 调用、Page Cache 拷贝、用户态-内核态切换上下文开销。3FS 提供 **USRBIO（User Space Ring Based IO）**，在用户空间基于环形无锁队列驱动存储与网卡交互；
- **直接直达 GPU / DRAM**：结合 InfiniBand / RoCEv2 的 RDMA Read/Write 语义，存储端 NVMe 控制器的数据可以直接穿透 PCI 拓扑直达计算节点的 GPU 显存或 Host DRAM，消除任何 CPU 中间拷贝开销，端到端延迟低至 10 微秒以内。

---

## 🛠️ 快速上手与配置

### 1. 核心架构依赖与构建

```bash
# 克隆 3FS 官方仓库与子模块
git clone --recurse-submodules https://github.com/deepseek-ai/3FS.git
cd 3FS

# 使用 Docker / 标准工具链编译 C++20 内核组件
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DENABLE_RDMA=ON -DUSE_FDB=ON ..
make -j$(nproc)
```

### 2. 存储服务节点配置 (`storage_node.toml`)

```toml
[server]
node_id = 101
listen_addr = "10.100.1.101:9000"
rdma_device = "mlx5_0"
rdma_port = 1

[storage]
backend = "nvme_direct"
nvme_devices = [
  "/dev/nvme0n1",
  "/dev/nvme1n1",
  "/dev/nvme2n1",
  "/dev/nvme3n1"
]
block_size_kb = 64
ring_buffer_depth = 1024

[replication]
chain_id = "chain-group-01"
role = "head" # 可选: head, middle, tail
next_node = "10.100.1.102:9000"
fdb_cluster_file = "/etc/foundationdb/fdb.cluster"
```

### 3. 高吞吐客户端挂载与 Python USRBIO 直连

```bash
# 启动 POSIX 兼容 FUSE 挂载点（用于常规运维工具与脚本读取）
./bin/3fs_fuse_client \
  --mountpoint /mnt/3fs-data \
  --fdb_cluster /etc/foundationdb/fdb.cluster \
  --rdma_device mlx5_0 \
  --threads 16
```

```python
# 基于 3FS USRBIO 接口进行高性能分布式 KV Cache 与 Dataloader 极速加载
import threefs_io as tio
import torch

# 初始化零拷贝 RDMA 客户端会话
session = tio.create_rdma_session(
    cluster_conf="/etc/foundationdb/fdb.cluster",
    device="mlx5_0"
)

# 直接将大模型长文本推理中的 KV Cache 块从 3FS 异步读取至 GPU 显存
def load_kv_cache_to_gpu(chunk_id: str, gpu_tensor: torch.Tensor):
    """
    通过 3FS RDMA 零拷贝直存引擎，将持久化存储中的 KV Cache 块直接灌入 GPU 内存
    """
    buffer_ptr = gpu_tensor.data_ptr()
    bytes_read = session.read_direct(
        file_path=f"/mnt/3fs-data/kv_cache/{chunk_id}.bin",
        target_memory_ptr=buffer_ptr,
        size_bytes=gpu_tensor.numel() * gpu_tensor.element_size(),
        use_cuda_ipc=True
    )
    return bytes_read
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **万卡大规模预训练数据加载器（Locality-Oblivious Dataloader）**：彻底摆脱传统训练必须预先将巨型数据集切片复制到本地本地 SSD 的烦琐流程，万卡跨机房直接远程拉取海量语料样本；
  2. **万亿参数模型超高速 Checkpoint 保存与恢复**：数十 TB 的权重状态（Weight & Optimizer States）通过百卡并发打满 3FS 聚合带宽，将容灾存档时间从数十分钟压入 10 秒以内，大幅提升算力集群有效训练时间比率（MFU）；
  3. **大模型分布式多级 KV Cache 异构卸载池（L3 存储基座）**：与 [[Mooncake]]、[[LMCache]]、[[vLLM]] 深度集成，将冷数据以毫秒级延迟溢出卸载到便宜的 3FS NVMe 存储池中，彻底打破长文本并发显存限制。
- **潜在不足 / 局限性**：
  1. **高度依赖高规格网络硬件**：3FS 的极限性能基于 RoCEv2 或 InfiniBand RDMA 无损网络环境，在标准千兆或万兆以太网 TCP 环境下无法发挥其零拷贝与用户态微秒级延迟优势；
  2. **基础设施运维复杂度高**：需额外部署与维护 Apple FoundationDB 分布式事务集群，对存储运维工程师的分布式理论与排障能力要求较高；
  3. **不适合常规轻量级开发测试**：如果只是几十个并发或几台机器的小规模应用，标准 NAS 或本地 SSD + MinIO 已足够，使用 3FS 属于“大炮打蚊子”。
- **个人评估结论**：**极度推荐采用（Adopt - 适用于算力平台与中大型 AI 基础设施）**。DeepSeek 3FS 的开源打破了传统高性能分布式存储被商业闭关系统垄断的局面，是目前最纯粹、性能最强悍的 AI 原生数据底座开源实现。