---
tags:
  - ai-tool
  - open-source
  - agent-platform
  - workflow
  - status/adopted
category: Agents
github: https://github.com/langgenius/dify
stars: "60k+"
license: Apache-2.0
date_added: 2026-09-23
---

# 📦 Dify

> **一句话简介**：开源的顶级大模型应用开发与工作流编排平台（LLM Application Development Platform），融合可视化 Agent 工作流、高质量 RAG 知识库与全生命周期 LLMOps 能力。

---

## 📌 基本信息

| 属性       | 内容                                                                        |
| :------- | :------------------------------------------------------------------------ |
| **仓库地址** | [GitHub: langgenius/dify](https://github.com/langgenius/dify)             |
| **技术栈**  | Python (Flask/Celery), Next.js, PostgreSQL, Redis, Weaviate/Qdrant/Milvus |
| **许可证**  | Apache-2.0 (注: 遵循开源商业化协议约束)                                               |
| **核心定位** | 企业级 AI Agent 敏捷编排与应用交付中台                                                  |
| **关联卡片** | [[LangGraph]], [[LiteLLM]], [[000-AI-Index]]                              |

---

## 🚀 核心架构与杀手级特性

### 1. 可视化工作流与编排引擎 (Workflows & Chatflows)
- **节点化拖拽编排**：提供 LLM、代码执行（Python/NodeJS 沙箱）、知识库检索、HTTP 请求、条件分支（IF/ELSE）、变量聚合器、迭代循环（Iteration）等节点。
- **对话流（Chatflow）与任务流（Workflow）分离**：
  - **Chatflow**：专为多轮交互式会话设计，自带记忆机制（Memory）；
  - **Workflow**：面向自动化批处理或自动化流水线，单次触发并返回结构化输出。
- **DSL 导出与版本受控**：每个工作流均可完整导出为 `.yml` 格式的 DSL 文件，支持纳入 Git 版本控制与 CI/CD 自动化发布。

### 2. 企业级 RAG 知识库系统
- 内置开箱即用的大文件分块（Chunking）、文本清洗、元数据过滤机制。
- 支持**混合检索（Hybrid Search）**与多种向量数据库（Qdrant, Milvus, PGVector 等）无缝切换。
- 内置主流 Rerank 模型接入支持，直接在界面调优召回准确率。

### 3. 全新模块化插件生态 (Dify Plugins 1.0+)
- 支持通过独立插件扩展工具（Tools）、模型提供商（Models）、Agent 推理策略与外部触发器（Triggers）。
- 插件采用独立沙箱运行，与核心系统进程解耦，支持从官方市场或私有 Git 仓库一键安装。

### 4. 开箱即用的 API 与 WebApp
- 编排好的 Agent 可以 1 秒生成独立访问的 WebApp 网页，也可以一键发布为遵循标准 RESTful 规范的后端 API，极大加速业务集成。

---

## 🛠️ 私有化 Docker Compose 快速部署

```bash
# 1. 克隆代码仓库
git clone https://github.com/langgenius/dify.git
cd dify/docker

# 2. 复制环境变量配置文件
cp .env.example .env

# 3. 一键启动全套微服务栈（含 API、Worker、Web、PostgreSQL、Redis、沙箱等）
docker compose up -d
```

> 启动成功后，浏览器访问 `http://localhost/install` 即可初始化管理员账号并进入后台。

---

## 💡 Dify vs LangGraph / 纯代码框架选型矩阵

| 评估维度 | Dify (应用平台) | LangGraph / PydanticAI (纯代码框架) |
| :--- | :--- | :--- |
| **上手门槛** | 极低（产品经理/业务人员均可编排） | 中高（需扎实的 Python/TS 工程功底） |
| **开发与迭代效率** | 极高（界面所见即所得、提示词在线调试） | 依赖代码编写、热重载与本地调试环境 |
| **业务逻辑灵活性** | 受限于现有节点与沙箱能力 | 绝对自由（图灵完备，可任意写死循环与黑盒逻辑） |
| **前端与运营后台** | 开箱自带 WebApp、多租户、日志追踪 | 需自建前端页面与后台管理系统 |
| **最适业务场景** | 企业通用客服、内部知识库、流程审批自动化 | 核心高复杂度代码编写、深度算法自驱 Agent |
