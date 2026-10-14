---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/jxnl/instructor
stars: "11k+"
license: MIT
date_added: 2026-10-14
---

# 📦 Instructor

> **一句话简介**：基于 **Pydantic** 的企业级大模型结构化抽取与强类型接口库，首创**验证报错模型自愈重试（Self-Correction with Validation Feedback）**与**流式部分对象还原（Streaming Partial JSON Objects）**，全兼容几乎所有商业云端与本地开源模型 API。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: jxnl/instructor](https://github.com/jxnl/instructor) |
| **技术栈** | Python / TypeScript |
| **许可证** | MIT |
| **核心特性** | Pydantic V2 强类型映射, Auto-Retries with LLM Feedback, `Partial[T]` 流式解析, Multi-provider 统一抽象 |
| **相关关联** | 关联实践: [[大模型端到端强类型输出：基于 Instructor 与 Pydantic 的流式结构化抽取与自愈重试实战]], [[2026-10-14-AI-Digest]], [[Outlines]], [[PydanticAI]] |

---

## 🚀 核心特性与架构亮点

### 1. 无侵入客户端补丁（Patched Client）
- **单行代码增强**：通过 `instructor.from_openai(client)` 或 `from_anthropic / from_gemini` 包装原生客户端；
- **保留原生能力**：保留所有原生 SDK 的连接池、代理、重试与流式能力，同时新增 `response_model` 参数，将不可靠的文本输出强制约束转换为强类型 Pydantic 对象。

### 2. 带验证上下文的自愈重试（Self-Healing Retries）
- **传统困境**：传统重试只是盲目向模型再次发送原始 Prompt，模型往往因为不清楚错误细节而在同一个地方再次犯错；
- **自愈机制**：当输出违反 Pydantic 校验规则（字段缺失、类型错误、自定义 `@field_validator` 业务校验失败）时，Instructor 自动捕获异常追踪信息（Traceback & Error Message），将其动态拼接为上下文反馈回大模型，引导模型针对性纠错，直至校验通过或达到 `max_retries` 上限。

### 3. 实时流式部分对象解析（Streaming Partial Objects）
- **告别整块等待**：针对大体积嵌套 JSON，传统方案需要等待数十秒全部 Token 生成结束才能解析；
- **增量解析引擎**：结合高性能增量 JSON 解析器，在 Token 流式生成的每一毫秒，实时反序列化出已完成部分字段的 `Partial[Model]`，前端可即时渲染渐进式表格与生成式 UI，彻底消除用户界面卡顿。

### 4. 智能多模式回退（Smart Modes）
- 自动适配底层供应商：支持 `Mode.TOOLS`（函数调用）、`Mode.JSON_SCHEMA`（官方结构化输出）以及 `Mode.JSON`（原生 Prompt 指令引导），在云端与本地开源模型之间提供统一的数据契约体验。

---

## 🛠️ 快速上手与典型代码

```bash
# 安装
pip install instructor pydantic openai
```

### 带有字段业务校验与自动纠错的提取示例：

```python
from typing import List
from pydantic import BaseModel, Field, field_validator
from openai import OpenAI
import instructor

# 1. 用 instructor 打补丁增强原生客户端
client = instructor.from_openai(OpenAI())

# 2. 定义带业务校验规则的 Pydantic 模型
class InvoiceItem(BaseModel):
    name: str = Field(description="商品或服务名称")
    quantity: int = Field(gt=0, description="数量，必须大于0")
    unit_price: float = Field(gt=0, description="单价，必须大于0")

class InvoiceData(BaseModel):
    invoice_id: str
    items: List[InvoiceItem]
    total_amount: float

    # 自定义校验：总金额必须与明细项合计一致
    @field_validator("total_amount")
    @classmethod
    def validate_total(cls, v: float, info):
        items = info.data.get("items", [])
        calculated_total = sum(item.quantity * item.unit_price for item in items)
        if abs(v - calculated_total) > 0.01:
            raise ValueError(f"总金额 {v} 与明细计算总和 {calculated_total} 不一致！请重新核对。")
        return v

# 3. 执行提取（若模型计算出错，将自动将 ValueError 反馈给模型重试）
resp: InvoiceData = client.chat.completions.create(
    model="gpt-4o-mini",
    response_model=InvoiceData,
    max_retries=3,
    messages=[
        {"role": "system", "content": "提取发票结构化信息，确保计算精确。"},
        {"role": "user", "content": "发票单号 INV-2026-998，购买了 2 台显示器单价 1200 元，以及 1 个键盘单价 300 元，总金额为 2700 元。"}
    ]
)

print(f"提取结果: 单号 {resp.invoice_id}, 包含明细 {len(resp.items)} 项, 总价 {resp.total_amount}")
```

---

## 💡 方案对比与选型评估

| 选型维度 | Instructor (Client-Side) | Outlines (Server-Side) | 原生 JSON Mode |
| :--- | :--- | :--- | :--- |
| **部署与适用场景** | **全通用**（云端 OpenAI/Claude/Gemini/本地 API） | 本地私有化推理集群（vLLM, llama.cpp） | 依赖特定模型厂商 API |
| **保证级别** | 语义级校验 + 自愈重试保证 | 底层数学级 Logits 掩码 100% 保证 | 仅语法级保证，无语义校验 |
| **流式部分对象** | ⭐⭐⭐⭐⭐（原生支持 `Partial[T]`） | ⭐⭐⭐（需定制） | ⭐⭐（需自行解析不完整 JSON） |
| **侵入性与依赖** | 极低（仅依赖 Pydantic，单行 patch） | 较高（需介入模型采样层） | 无外部依赖 |

### 生产评估结论：
- **必备基础库**：在涉及云端商业模型 API 接入、复杂跨字段业务校验、流式生成式 UI 交互及批量结构化抽取场景下，Instructor 是最成熟、最符合工程直觉的首选工具。
