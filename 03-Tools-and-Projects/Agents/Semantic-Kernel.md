---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/microsoft/semantic-kernel
stars: "28k+"
license: MIT
date_added: 2026-10-15
---

# 📦 Semantic Kernel

> **一句话简介**：微软官方企业级轻量化大型语言模型（LLM）编排 SDK，支持 C#/.NET、Python 与 Java 多语言异构技术栈，以 **Kernel 核心容器**、**强类型插件体系（Plugins）**、**多轮规划器（Planners）** 与 **三层拦截过滤器（Filters）** 为基石，专为工业级安全治理与企业业务系统集成打造。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel) |
| **技术栈** | C# / .NET, Python, Java |
| **许可证** | MIT |
| **核心特性** | 跨语言原生支持, 强类型插件与 OpenAPI 自动绑定, 三层过滤器治理（审计/脱敏/熔断）, 步骤规划器与 Agent 体系 |
| **相关关联** | 关联实践: [[企业级多语言智能体编排：基于 Microsoft Semantic Kernel 的插件化架构与过滤器拦截实战]], [[2026-10-15-AI-Digest]], [[PydanticAI]], [[LangGraph]], [[FastMCP]] |

---

## 🚀 核心特性与架构亮点

### 1. 统一内核与服务解耦（Kernel & Service Architecture）
- **核心依赖注入容器**：`Kernel` 作为中心调度枢纽，聚合模型服务（ChatCompletion、TextEmbedding 等）、插件（Plugins）以及拦截器（Filters）；
- **多模型服务混编**：单 Kernel 实例可注册多个提供商（Azure OpenAI、原生 OpenAI、HuggingFace、Ollama、LiteLLM 等），并通过 `service_id` 按业务场景动态分发调用，天然支持故障转移与成本路由。

### 2. 强类型插件与 OpenAPI 零样板自动映射（Kernel Plugins）
- **原生函数声明**：使用 `@kernel_function` 装饰器，直接将企业现有业务代码封装为 Agent 可调用的工具，支持全类型注解（Type Annotations）与详细语义描述；
- **OpenAPI / Swagger 即插即用**：无需手动编写 Prompt 或 JSON Schema，直接解析企业现有的 RESTful API Swagger 规范，自动将其编译为合法的 Kernel 工具集。

### 3. 企业级三层拦截过滤器（Filter Pipelines for Governance）
企业落地 Agent 最核心的门槛是**安全、审计与流控**。Semantic Kernel 原生提供了三层细粒度拦截能力：
1. **PromptRenderFilter**：在 Prompt 模板填充变量前后执行，用于敏感实体识别、动态注入公司通用安全合规免责声明，以及防注入检测；
2. **FunctionInvocationFilter**：在每一次工具函数执行前后触发，进行精细化 RBAC 权限鉴权、入参安全白名单清洗、耗时统计与执行审计日志记录；
3. **AutoFunctionInvocationFilter**：在模型连续多轮触发 Tool Calling 的全局循环中拦截，可实时读取当前调用轮次、Token 消耗、或业务终止信号，强制中断恶性循环或防止无限 Token 账单膨胀。

### 4. 智能规划器与 Agent 体系（Planners & Process Framework）
- **函数调用规划器（FunctionCallingStepwisePlanner）**：利用模型原生的 Tool Calling 能力完成多步骤思考-行动-观察（ReAct）循环，比传统基于纯文本 Prompt 的 ReAct 更加稳定；
- **多智能体对话协议（AgentGroupChat）**：内置类似 AutoGen 的多角色协作机制，支持定义多智能体选择策略与终止条件（TerminationStrategy）。

---

## 🛠️ 快速上手与典型代码

```bash
# 安装 Python 官方 SDK
pip install semantic-kernel
```

### 带有业务插件与自动工具调用的完整 Python 示例：

```python
import asyncio
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion, OpenAIChatCompletion
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior
from semantic_kernel.functions import kernel_function
from semantic_kernel.filters import FilterTypes, FunctionInvocationContext

# 1. 定义企业业务插件
class InventoryPlugin:
    """企业库存与订单查询插件"""
    
    @kernel_function(
        name="check_stock",
        description="根据商品 SKU ID 查询当前仓库的可用库存数量"
    )
    def check_stock(self, sku_id: str) -> int:
        mock_db = {"SKU-A100": 42, "SKU-H800": 0}
        return mock_db.get(sku_id, 0)

# 2. 定义企业级函数拦截过滤器（安全审计与权限校验）
async def audit_filter(context: FunctionInvocationContext, next):
    print(f"🔒 [安全审计] 正在调用工具: {context.function.plugin_name}.{context.function.name}")
    print(f"📥 [入参校验] 参数列表: {context.arguments}")
    
    # 执行下一个环节或真实函数
    await next(context)
    
    print(f"📤 [执行完成] 输出结果: {context.result.value}")

# 3. 初始化 Kernel 并组装服务与插件
async def main():
    kernel = Kernel()
    
    # 注入模型服务（兼容 OpenAI / Azure OpenAI / 本地网关）
    kernel.add_service(
        OpenAIChatCompletion(
            service_id="default_chat",
            ai_model_id="gpt-4o-mini",
            api_key="sk-mock-key"
        )
    )
    
    # 注册插件与过滤器
    kernel.add_plugin(InventoryPlugin(), plugin_name="Inventory")
    kernel.add_filter(FilterTypes.FUNCTION_INVOCATION, audit_filter)
    
    # 配置自动函数调用策略（启用模型自主决策执行插件）
    settings = kernel.get_prompt_execution_settings_from_service_id("default_chat")
    settings.function_choice_behavior = FunctionChoiceBehavior.Auto(auto_invoke=True)
    
    # 执行复杂多步骤问答
    prompt = "请帮我查询 SKU-A100 的库存数量，并告知是否满足 10 台的发货需求。"
    result = await kernel.invoke_prompt(
        prompt=prompt,
        settings=settings
    )
    print(f"\n🤖 Agent 最终回复:\n{result}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 💡 方案对比与选型评估

| 选型维度 | Microsoft Semantic Kernel | LangGraph / LangChain | PydanticAI |
| :--- | :--- | :--- | :--- |
| **多语言生态** | ⭐⭐⭐⭐⭐（.NET、Python、Java 官方一线支持） | ⭐⭐（主要集中在 Python 与 JS） | ⭐（专注于 Python 生态） |
| **企业集成与治理** | ⭐⭐⭐⭐⭐（原生三层 Filter，天然契合企业安全与审计） | ⭐⭐⭐（需依赖 Callback 机制，规范较重） | ⭐⭐⭐⭐（基于依赖注入与强类型校验） |
| **工具映射体验** | 强类型 `@kernel_function` + 原生 OpenAPI 自动解析 | `@tool` 装饰器或 StructuredTool | 基于 Pydantic 的 `@agent.tool` |
| **架构稳定性** | 极高（微软企业级软件工程规范，API 演进严谨） | 变化较频繁，版本迁移成本高 | 极高（极简设计，基于 Pydantic V2） |
| **多模型与服务混编**| 原生支持多 Provider 注册与按 ID 路由 | 需手动组织 Chain 或 Graph 路由节点 | 支持配置多模型客户端 |

### 生产评估结论：
- **企业异构系统首选**：对于拥有 .NET/C# 传统企业服务、Java 后台系统或 Python AI 中台混合技术栈的团队，Semantic Kernel 是打破技术栈壁垒、构建统一企业 Agent 中枢的最佳选择；
- **重度合规与金融场景首选**：其原生 Filters 机制为金融、政企等强监管行业提供了近乎完美的拦截、风控、脱敏与计费接入点。
