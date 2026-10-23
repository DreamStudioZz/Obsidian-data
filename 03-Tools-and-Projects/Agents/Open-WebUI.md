---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/open-webui/open-webui
stars: "98k+"
license: MIT
date_added: 2026-10-23
---

# 📦 Open-WebUI

> **一句话简介**：全球最受欢迎的开源、离线优先的企业级多模型交互中枢与自主应用平台。深度集成 Ollama 与兼容 OpenAI 规范的私有/商用推理引擎，内置生产级 RAG 知识库检索、细粒度 RBAC 权限审计，并通过模块化 **Pipelines** 插件机制实现请求拦截、动态安全护栏与中间件扩展。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: open-webui/open-webui](https://github.com/open-webui/open-webui) |
| **技术栈** | Python 3.11+ / FastAPI / SvelteKit / TailwindCSS / Chroma / SQLite & PostgreSQL |
| **许可证** | MIT License |
| **核心特性** | 100% 离线自托管、多模型统一网关纳管（Ollama/vLLM/Xinference/LiteLLM/OpenAI）、内置开箱即用 RAG 知识库、Pipelines 插件流水线拦截、细粒度企业 RBAC 权限与计费配额、多模态图文与语音交互、OpenAI 兼容外部 API 暴露 |
| **相关关联** | 关联实践: [[企业级私有化大模型统一交互中枢：基于 Open-WebUI 的多模型纳管、知识库 RAG 与 Pipelines 自定义中间件实战]], [[2026-10-23-AI-Digest]], [[Chroma]], [[Ollama]], [[LiteLLM]], [[vLLM]], [[Xinference]], [[Langfuse]], [[Guardrails]] |

---

## 🚀 核心特性与架构亮点

### 1. 离线优先与企业私有化多模型统一纳管
Open-WebUI 是专为企业内网与私有算力设计的统一用户交互与调度中枢：
- **无缝对接私有推理基座**：原生支持直连本地或局域网内的 [[Ollama]]、[[vLLM]]、[[Xinference]]、[[SGLang]]，自动发现并拉取模型列表；
- **全生态 OpenAI 协议兼容**：可同时挂载多个云端商用大模型提供商（OpenAI、Anthropic、DeepSeek、Moonshot 等）或企业网关（如 [[LiteLLM]]），支持在一套 UI 中无缝切换与并排盲测（Arena 模式）；
- **零外部网络依赖**：核心代码库与静态资源完全离线打包，支持纯局域网乃至物理隔离网络环境的私有化交付。

### 2. 企业级细粒度 RBAC 权限控制与资产隔离
针对企业多团队、多部门协同场景，Open-WebUI 提供了完善的企业治理能力：
- **多角色权限模型（RBAC）**：内置 Admin、User 等角色，支持限制普通用户的模型调用范围、每日/每月 Token 消费配额、以及外部 Web 搜索开关；
- **Prompt 模板与 Modelfiles 沉淀**：支持用户或管理员将系统提示词、温度（Temperature）、惩罚参数（Top-P）、挂载工具封装为独立的专属“自定义模型”，并在组织内一键共享；
- **审计日志与合规监控**：所有对话历史、模型调用耗时、Token 吞吐量均统一沉淀至数据库，原生支持与 [[Langfuse]] 等企业级可观测底座联动。

### 3. 开箱即用的工业级混合 RAG 知识库
Open-WebUI 内置了完整的端到端文档检索问答闭环：
- **多格式高保真文档解析**：支持 PDF、DOCX、TXT、CSV、Markdown、网页抓取等各类数据源；
- **嵌入式向量存储集成**：默认采用轻量高效的 [[Chroma]] 向量数据库，支持持久化存储并可外挂外部独立向量集群；
- **BM25 + Dense 混合检索与 Rerank**：支持挂载 BGE-Reranker、Cohere Rerank，并在前端提供高置信度引用来源（Citations）溯源弹窗，有效遏制业务幻觉。

### 4. 模块化 Pipelines 架构与中间件扩展机制
通过独立的 **Pipelines**（基于 FastAPI 的插件式流水线引擎），Open-WebUI 具备了无限的二次开发可能：
- **请求/响应双向拦截**：在用户 Query 发送至 LLM 之前或 LLM 输出返回前端之前，通过 Python 脚本实现 PII 敏感信息脱敏、语义意图路由与特定风控检查；
- **函数调用（Function Calling）与外部工具集成**：支持通过 Python 装饰器快速编写自定义 Tool，允许模型在对话中动态调用公司 ERP、SQL 数据库或外部 RESTful 接口；
- **生态隔离与热插拔**：Pipelines 作为独立子服务运行，修改插件代码无需重启前端 UI 核心容器。

---

## 🛠️ 快速上手与典型配置

### 1. 基于 Docker Compose 快速编排启动（集成 Ollama + Open-WebUI）

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ./ollama_data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_SECRET_KEY=your_production_secret_key_change_me
      - ENABLE_SIGNUP=false # 生产环境建议关闭自注册，由管理员受控分配
      - DEFAULT_USER_ROLE=user
      - VECTOR_DB=chroma # 默认采用轻量 Chroma 向量库
      - CHROMA_DATA_PATH=/app/backend/data/vector_db
      - RAG_TOP_K=5
      - RAG_RELEVANCE_THRESHOLD=0.4
    volumes:
      - ./open_webui_data:/app/backend/data
    depends_on:
      - ollama
```

### 2. 核心 Pipelines 自定义中间件编写示例

通过编写继承自 `Pipeline` 的 Python 模块，可轻松实现请求前敏感词审查与动态提示词注入：

```python
"""
title: Enterprise Compliance & Prompt Injection Filter
author: Antigravity AI Team
date: 2026-10-23
version: 1.0.0
license: MIT
description: 拦截用户输入，进行合规校验与统一系统提示词注入的生产级中间件
"""

import re
from typing import List, Optional, Union, Generator, Iterator
from pydantic import BaseModel

class Pipeline:
    class Valves(BaseModel):
        # 可在 Open-WebUI 后台动态调整的阀门配置
        max_prompt_length: int = 4096
        enable_pii_masking: bool = True
        system_prefix: str = "【企业安全模式开启】你是一个严谨的企业 AI 助手，请基于事实回答。"

    def __init__(self):
        self.name = "Enterprise Security Middleware"
        self.valves = self.Valves()
        # 常见敏感信息正则：手机号、身份证号
        self.phone_pattern = re.compile(r'1[3-9]\d{9}')
        self.id_card_pattern = re.compile(r'\d{17}[\dXx]')

    async def on_startup(self):
        print(f"[{self.name}] 中间件已成功加载并启动。")

    async def on_shutdown(self):
        print(f"[{self.name}] 中间件已卸载。")

    def inlet(self, body: dict, user: Optional[dict] = None) -> dict:
        """
        请求进入 LLM 之前的拦截器：执行脱敏与上下文增强
        """
        messages = body.get("messages", [])
        if not messages:
            return body

        # 获取用户最新的 Query
        last_message = messages[-1]
        content = last_message.get("content", "")

        # 1. 长度熔断截断
        if len(content) > self.valves.max_prompt_length:
            raise ValueError(f"Prompt 长度超出企业安全阈值 ({self.valves.max_prompt_length} 字符)")

        # 2. PII 敏感信息掩码替换
        if self.valves.enable_pii_masking:
            content = self.phone_pattern.sub("[已脱敏手机号]", content)
            content = self.id_card_pattern.sub("[已脱敏身份证]", content)
            last_message["content"] = content

        # 3. 动态系统 Prompt 注入
        if messages[0].get("role") == "system":
            messages[0]["content"] = f"{self.valves.system_prefix}\n{messages[0]['content']}"
        else:
            messages.insert(0, {"role": "system", "content": self.valves.system_prefix})

        body["messages"] = messages
        return body

    def outlet(self, body: dict, user: Optional[dict] = None) -> dict:
        """
        LLM 生成完成返回前端之前的拦截器：追加企业版权与溯源标识
        """
        # 可在此处执行输出违规词过滤或企业版权信息追加
        return body
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **企业私有化 AI 办公门户**：整合本地 Ollama / vLLM 算力，为全员提供安全、低延迟、可控的 ChatGPT 替代方案；
  2. **多部门知识库问答中枢**：按业务团队建立独立的 RAG 知识库，各部门上传各自 SOP 与技术手册并设定权限隔离；
  3. **模型评估与 Benchmark 对比**：利用 Arena 双模同屏对比能力，由算法团队直观评估各开源微调模型在具体业务 Query 下的优劣。
- **潜在不足 / 局限性**：
  1. **默认存储限制**：默认使用 SQLite 存储用户会话与元数据，在几百人以上高并发场景下可能触发写入锁争用，生产部署强烈建议配置为外部独立 PostgreSQL；
  2. **Pipelines 性能开销**：如果在 Inlet/Outlet 中执行重度同步阻塞逻辑（如高延迟外部 API 调用），会导致 SSE 流式响应首字延迟（TTFT）显著增加，必须采用异步非阻塞编写。
- **个人评估结论**：**生产级必选底座（status/adopted）**。目前开源界交互最完善、生态最繁荣、二次扩展能力最强的企业级私有化 AI 客户端交互中枢。
