---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/pydantic/pydantic-ai
stars: "15k+"
license: MIT
date_added: 2026-09-21
---

# 📦 PydanticAI

> **一句话简介**：由 Pydantic 官方团队打造的 Python 类型安全 Agent 框架，将 FastAPI 式的类型注解、依赖注入与严格输出校验带入大模型智能体开发。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) |
| **主创团队** | Pydantic 团队（Samuel Colvin 等） |
| **技术栈** | Python 3.10+ / Pydantic V2 / AsyncIO |
| **生态集成** | OpenAI, Anthropic, Gemini, Groq, Ollama, Pydantic Logfire, Langfuse |
| **关联实践** | [[基于 PydanticAI 的类型安全 Agent 架构设计与结构化输出实践]], [[FastMCP]] |

---

## 🚀 为什么选择 PydanticAI？核心特性与亮点

在传统 Agent 框架中，大模型的输出往往是松散的自然语言或弱类型字典，生产环境中频繁发生**JSON Schema 结构缺失、类型转换崩溃、工具入参幻觉**等问题。PydanticAI 正是为解决此痛点而生：

1. **强类型智能体闭环（Typed Agent Loop）**：
   - 智能体的输入、输出、依赖、工具调用全部基于 Python 类型注解与 Pydantic 验证器。
   - 定义输出格式只需声明 `result_type=MyDataModel`，框架会自动将 Schema 注入模型，并在模型返回不合规格式时自动触发反思重试（Reflection Loop），直至校验通过。
2. **依赖注入机制（Dependency Injection via `RunContext`）**：
   - 借鉴 FastAPI 的依赖注入设计，可在 Agent 执行时将数据库连接、HTTP 会话、用户鉴权身份直接注入到工具函数中，极大简化了单元测试与 Mock。
3. **模型中立与极简切换（Model Agnostic）**：
   - 一行代码即可在 OpenAI、Anthropic、Gemini 或本地 Ollama 之间无缝切换，无需改动任何工具逻辑或输出模型。
4. **原生流式与结构化校验（Streaming Structured Validation）**：
   - 支持一边从大模型接收 Token 流，一边逐步解析并验证部分 Pydantic 模型，兼顾极致的交互响应速度与数据完整性。

---

## 🛠️ 快速上手与示例

### 1. 安装
```bash
pip install pydantic-ai
```

### 2. 构建带严格输出与依赖注入的智能体
```python
from dataclasses import dataclass
import httpx
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext

# 1. 声明结构化产物 Schema
class RepositoryHealth(BaseModel):
    repo_name: str = Field(description="仓库全名，如 owner/repo")
    star_count: int = Field(description="Stars 数量")
    is_active: bool = Field(description="近 30 天是否有活跃提交")
    health_score: float = Field(description="健康评分 0.0 - 10.0")
    summary: str = Field(description="核心诊断评价")

# 2. 声明运行时依赖
@dataclass
class AgentDeps:
    http_client: httpx.AsyncClient
    github_token: str

# 3. 初始化类型安全 Agent
agent = Agent(
    model="openai:gpt-4o",
    result_type=RepositoryHealth,
    deps_type=AgentDeps,
    system_prompt="你是一名资深开源项目架构评估师，请基于工具数据评估仓库健康度。"
)

# 4. 注册带类型注入的工具
@agent.tool
async def fetch_repo_stats(ctx: RunContext[AgentDeps], repo: str) -> dict:
    """获取指定 GitHub 仓库的实时统计信息"""
    headers = {"Authorization": f"Bearer {ctx.deps.github_token}"}
    resp = await ctx.deps.http_client.get(f"https://api.github.com/repos/{repo}", headers=headers)
    return resp.json()

# 5. 执行调用
async def main():
    async with httpx.AsyncClient() as client:
        deps = AgentDeps(http_client=client, github_token="ghp_xxxx")
        result = await agent.run("评估 pydantic/pydantic-ai 的项目状态", deps=deps)
        
        # 结果已被严格校验为 RepositoryHealth 实例
        data: RepositoryHealth = result.data
        print(f"[{data.repo_name}] 评分: {data.health_score} | 评语: {data.summary}")

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

## 💡 工程实战点评与选型建议

- **最推荐场景**：
  - **核心业务管线中的 Agent**：下游系统强依赖确定性 JSON 结构（如报表生成、入库落盘、触发工作流）。
  - **需要严密单元测试的代码库**：利用 `RunContext` 轻松完成对数据库、网络调用的依赖替换与 Mock。
  - **Python 后端开发者首选**：如果你熟悉 FastAPI / Pydantic，学习成本几乎为零。
- **与 LangChain / CrewAI 对比**：
  - 没有繁杂冗余的抽象层，代码透明直观，报错栈清晰，极具工程掌控力。
- **综合评估结论**：生产级 Python AI 智能体开发推荐优先采纳（Adopted）。
