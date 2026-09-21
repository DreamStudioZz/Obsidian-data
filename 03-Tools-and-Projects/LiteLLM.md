---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Gateway
github: https://github.com/BerriAI/litellm
stars: "20k+"
license: MIT
date_added: 2026-09-21
---

# 📦 LiteLLM

> **一句话简介**：使用 1 行统一规范调用 100+ 商业与开源大模型，提供开箱即用的负载均衡、故障转移降级与成本限额代理网关。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: BerriAI/litellm](https://github.com/BerriAI/litellm) |
| **技术栈** | Python, FastAPI |
| **许可证** | MIT |
| **相关关联** | [[vLLM]], [[2026-09-21-AI-Digest]] |

---

## 🚀 核心价值与架构解决的问题

### 1. 消除供应商锁定与接口适配成本
在真实业务中，通常需要混用 OpenAI (GPT-4o)、Anthropic (Claude 3.5 Sonnet)、Google (Gemini 1.5/2.0)、DeepSeek 以及自建的本地 [[vLLM]]。各家 SDK、入参格式、流式响应（Streaming chunks）规范差异巨大。
LiteLLM 将所有调用**统一收敛为标准 OpenAI 格式**。

### 2. 生产级代理代理层（Proxy Gateway）
LiteLLM 不仅是一个 Python 库，还自带可独立 Docker 部署的 Proxy Server：
- **负载均衡与故障转移（Load Balancing & Fallbacks）**：当主模型接口（如 Claude）触发 429 限流或 500 报错时，毫秒级自动重试并优雅降级切换至备用模型（如 GPT-4o 或 DeepSeek-V3）。
- **成本与配额审计**：自动统计每个团队/用户的 Token 消耗与计费账单，支持设定预算熔断限额。
- **动态路由**：可根据请求 Prompt 长度或任务类型自动路由到性价比最高的模型。

---

## 🛠️ 快速上手

### 方式 A：Python 代码中统一调用
```python
from litellm import completion

# 调用 Claude
response_claude = completion(
    model="claude-3-5-sonnet-20240620",
    messages=[{"role": "user", "content": "你好"}],
)

# 无缝切换调用 DeepSeek
response_deepseek = completion(
    model="deepseek/deepseek-chat",
    messages=[{"role": "user", "content": "你好"}],
)

# 统一配置自动降级（Fallback）
response_safe = completion(
    model="claude-3-5-sonnet-20240620",
    messages=[{"role": "user", "content": "你好"}],
    fallbacks=["gpt-4o", "deepseek/deepseek-chat"],
)
```

### 方式 B：作为集中网关启动
```bash
# 启动 LiteLLM Proxy，对外提供统一的 OpenAI 协议端点
pip install 'litellm[proxy]'
litellm --config config.yaml --port 4000
```

---

## 💡 选型与落地建议
- 对于中大型团队搭建内部统一“AI 中台 / AI Gateway”，LiteLLM 是目前最轻量、维护成本最低的首选方案。
- 相比于从头手写封装各类 SDK，LiteLLM 保证了官方 SDK 迭代后第一时间跟进新参数。
