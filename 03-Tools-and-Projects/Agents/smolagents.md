---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/huggingface/smolagents
stars: "18k+"
license: Apache-2.0
date_added: 2026-09-22
---

# 📦 smolagents

> **一句话简介**：Hugging Face 官方出品的轻量级 Agent 框架，主打“代码驱动智能体（CodeAgent）”哲学，让大模型直接编写可执行 Python 代码来组合调用工具，彻底解决传统 JSON 工具调用的 Token 膨胀与逻辑表达受限问题。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: huggingface/smolagents](https://github.com/huggingface/smolagents) |
| **出品方** | Hugging Face 官方团队 |
| **核心特点** | 极简架构（约 1000 行核心代码）、CodeAgent 原生、沙箱安全隔离 |
| **生态集成** | Hugging Face Hub, LiteLLM, Ollama, OpenAI, Anthropic, MCP |
| **关联实践** | [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]], [[FastMCP]] |

---

## 🚀 核心架构与创新亮点

传统 Agent（如 LangChain / OpenAI Assistants）在调用工具时普遍采用 **JSON Schema Function Calling** 模式。但在面对循环、数据流管道和条件判断时，JSON 模式会产生严重的弊端：
- 需要多次往返调用 LLM（每执行一个步骤就要回传一次 JSON），网络延迟和 Token 开销巨大；
- 无法直接表达 `for item in results: if condition(item): ...` 这样最朴素的编程控制流。

**smolagents 的核心突破**：
1. **“以代码思考”（Agents that think in code）**：
   - 核心组件 `CodeAgent` 引导大模型将操作意图直接写成一段 Python 代码块。
   - 大模型可以直接定义变量、编写循环、调用自定义函数并把中间结果传递给下一个工具，在单次生成中完成多步复杂编排。
2. **零抽象膨胀（Minimal Abstraction）**：
   - 抛弃繁琐的链式封装与复杂的内部状态机，核心逻辑仅千行代码，极易审计、调试与魔改。
3. **安全执行环境（Secure Execution Sandbox）**：
   - 支持本地受限解释器，并原生集成 Docker、E2B、Modal 等沙箱容器，确保模型生成的执行代码完全隔离，防范越权风险。
4. **全生态工具兼容**：
   - 原生支持将 LangChain Tools、Hub 社区工具、Model Context Protocol (MCP) 服务作为底层算子无缝挂载。

---

## 🛠️ 快速上手示例

### 1. 安装
```bash
pip install smolagents
```

### 2. 构建一个具备循环与数据处理的 CodeAgent
```python
from smolagents import CodeAgent, HfApiModel, tool

# 1. 使用极简装饰器声明工具
@tool
def fetch_user_orders(user_id: str) -> list:
    """获取指定用户的历史订单列表。
    
    Args:
        user_id: 用户的唯一标识符
    """
    # 模拟返回订单数据
    return [
        {"order_id": "A101", "amount": 120.5, "status": "completed"},
        {"order_id": "A102", "amount": 450.0, "status": "refunded"},
        {"order_id": "A103", "amount": 89.9, "status": "completed"},
    ]

# 2. 初始化模型与 CodeAgent
model = HfApiModel(model_id="Qwen/Qwen2.5-Coder-32B-Instruct")
agent = CodeAgent(tools=[fetch_user_orders], model=model)

# 3. 运行任务：模型会自动编写 Python 代码进行汇总过滤
agent.run("帮我计算用户 'U_8892' 所有已完成（completed）订单的总金额。")
```

大模型内部生成的执行代码片段（单次交互即可计算完成）：
```python
# 模型自主生成的紧凑代码
orders = fetch_user_orders(user_id="U_8892")
completed_total = sum(order["amount"] for order in orders if order["status"] == "completed")
print(f"总金额为: {completed_total}")
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **数据分析与批处理 Agent**：需要对获取的一组列表数据进行排序、聚合、清洗、格式转换；
  - **研发效能与运维 Agent**：需要遍历目录、正则匹配日志、批量调用 API 接口；
  - **追求极速与低成本的轻量级场景**：大幅削减中间轮次的 LLM 调用次数。
- **与传统 JSON Tool Calling 对比**：
  - 代码执行方式在复杂编排场景下能减少 **60% 以上的 API 调用开销**，同时大幅提升多步逻辑的成功率。
- **综合评估结论**：面向复杂任务与批量处理，强烈推荐采纳（Adopted）。
