---
tags:
  - ai-practice
  - engineering
  - architecture
domain: 本地部署加速
difficulty: 中等
date_added: 2026-09-23
---

# 💡 私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构

> **核心摘要**：针对企业研发团队本地离线研发、数据安全隔离与多模型统一调度的诉求，通过“底层 Ollama 极简算力容器 + 顶层 LiteLLM Proxy 统一智能路由”的轻量化双层架构，实现本地私有开源模型与云端商业模型的混合部署、高可用故障转移（Failover）与 Token 成本审计。

---

## 🎯 业务/技术背景与痛点

企业在落地大模型应用或小团队日常开发中，直接调用云端商业 API 或在多台机器上裸跑脚本常遭遇以下困境：
1. **源码与敏感数据泄漏风险**：金融、政务或核心专利代码严禁上传至第三方公共 API；
2. **多端本地环境割裂**：不同开发者的 GPU/CPU 架构不同，下载不同格式的权重文件（HuggingFace、Safetensors、GGUF）反复报错，运维极其痛苦；
3. **缺乏统一网关治理**：无法按角色实施 Rate Limit（限流）、多节点负载均衡，且本地显卡偶尔显存溢出（OOM）时没有降级容灾方案。

---

## 🏗️ 架构设计与解决方案

构建 **“Ollama 异构节点集群 + LiteLLM 统一反向代理网关”** 的轻量级高可用拓扑：

```mermaid
flowchart TD
    ClientApp[各业务系统 / Cursor / Agent / 内网知识库] -->|统一 OpenAI 格式 API 请求| LiteLLM[🛡️ LiteLLM Proxy 智能调度网关]

    subgraph Governance [网关治理层]
        LiteLLM --> Auth[统一虚拟 API Key 鉴权]
        LiteLLM --> Quota[用户配额 & 预算熔断]
        LiteLLM --> FallbackRoute{动态健康检查与路由}
    end

    subgraph LocalCluster [本地私有算力池 (Ollama)]
        FallbackRoute -->|主路由: 本地极速调用 (0 成本)| Ollama1[🖥️ Node A: Ollama (Qwen2.5-Coder)]
        FallbackRoute -->|从路由: 本地大显存节点| Ollama2[🖥️ Node B: Ollama (Llama-3.3-70B)]
    end

    subgraph CloudBackup [公有云容灾兜底]
        FallbackRoute -.->|健康检查失败 / 超时自动降级| CloudAPI[☁️ DeepSeek / Claude / OpenAI API]
    end

    Ollama1 & Ollama2 & CloudAPI --> LiteLLM
    LiteLLM --> Langfuse[(Langfuse 统一可观测审计)]
```

### 核心设计原则
1. **统一 OpenAI 协议抽象**：
   - 业务方只对接 `http://gateway.company.internal/v1`，无需关心底层运行的是本地 Ollama 还是云端商用模型，切换底层零改动；
2. **零成本优先 + 自动云端降级（Automatic Fallback）**：
   - 优先路由到内部本地部署的 Ollama 实例（零 API 费用、局域网低延迟）；当本地节点高负载排队超时或宕机时，网关透明降级至云端商业模型，业务无感知；
3. **Modelfile 标准化交付**：
   - 团队内部统一将微调/提示词加固后的模型打包为标准 Modelfile，一键分发至每位工程师的工作站。

---

## 💻 关键配置与代码实现

### 1. 定制标准开发模型 Modelfile
创建 `Modelfile.coder`：
```dockerfile
# 基于 Qwen2.5-Coder 7B
FROM qwen2.5-coder:7b

# 设置温度与上下文长度
PARAMETER temperature 0.1
PARAMETER num_ctx 32768
PARAMETER stop "<|im_end|>"

# 注入团队标准工程提示词
SYSTEM """你是企业内部资深开发助手，严格遵循内部编码规范：
1. 优先使用 Python 3.11+ 类型注解与 Pydantic V2；
2. 所有公开接口必须具备 Google Docstring 说明；
3. 严格禁止泄露内部凭证与明文密码。
"""
```

编译并启动模型：
```bash
ollama create internal-coder -f ./Modelfile.coder
```

### 2. LiteLLM 网关高可用路由配置 (`config.yaml`)
```yaml
model_list:
  # 统一模型名称: internal-coding-agent
  - model_name: internal-coding-agent
    litellm_params:
      model: ollama/internal-coder
      api_base: http://192.168.1.100:11434  # 本地 Ollama 节点 A
      tpm: 100000
  - model_name: internal-coding-agent
    litellm_params:
      model: ollama/internal-coder
      api_base: http://192.168.1.101:11434  # 本地 Ollama 节点 B (负载均衡)
  - model_name: internal-coding-agent
    litellm_params:
      model: deepseek/deepseek-chat         # 云端兜底模型
      api_key: os.environ/DEEPSEEK_API_KEY

router_settings:
  routing_strategy: latency-based-routing   # 按延迟与健康度调度
  num_retries: 2                           # 失败重试次数
  timeout: 30                              # 本地超时 30 秒则触发降级
  fallbacks:
    - internal-coding-agent: ["deepseek/deepseek-chat"] # 故障无缝降级

general_settings:
  master_key: sk-enterprise-master-token
```

启动网关：
```bash
litellm --config ./config.yaml --port 4000
```

---

## ⚠️ 生产环境踩坑与避坑指南

1. **Ollama 默认长上下文截断导致指令失效**：
   - **踩坑表现**：本地运行模型时，长代码直接被从中间腰斩，因为 Ollama 默认 `num_ctx` 仅为 2048 Token。
   - **避坑方案**：在 Modelfile 中显式声明 `PARAMETER num_ctx 32768`，并在客户端启动请求中显式传递 `options={"num_ctx": 32768}`。
2. **多并发请求导致显存排队严重抖动**：
   - **踩坑表现**：Ollama 默认单模型单并发，当多个同事同时调用时，后续请求被硬挂起阻塞。
   - **避坑方案**：配置环境变量 `OLLAMA_NUM_PARALLEL=4` 并适度降低量化位宽（如从 Q8_0 改为 Q4_K_M），允许在单张显卡中并发处理多个请求上下文。
3. **局域网跨网段访问被拒**：
   - **踩坑表现**：部署在服务器上的 Ollama 默认绑定 `127.0.0.1`，LiteLLM 无法跨机访问。
   - **避坑方案**：在启动环境增加 `OLLAMA_HOST=0.0.0.0:11434`。

---

## 🔗 关联项目与引用
- 核心工具：[[Ollama]], [[LiteLLM]], [[vLLM]], [[SGLang]]
- 关联实践：[[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]]
