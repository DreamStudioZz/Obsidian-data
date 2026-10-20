---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Evaluation-and-Ops
github: https://github.com/guardrails-ai/guardrails
stars: "6.5k+"
license: Apache-2.0
date_added: 2026-10-20
---

# 📦 Guardrails

> **一句话简介**：开源的大模型**运行时可靠性保障、结构化规范输出校验与自愈重试引擎**。基于 RAIL（Reliable AI Markup Language）规范和 Pydantic Schema，通过丰富的 Guardrails Hub 验证器生态，对大模型生成的文本与 JSON 输出实施毫秒级内容安全审计（PII 敏感信息脱敏、SQL 注入拦截、毒性言论过滤、有害代码屏蔽、幻觉溯源比对），并在出现校验失败时支持自动纠正（fix）、静默裁剪（filter）或触发智能自愈重问（reask）回路，构建从不可预测生成到企业级生产安全可信的坚固防火墙。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: guardrails-ai/guardrails](https://github.com/guardrails-ai/guardrails) |
| **技术栈** | Python 3.10+ / Pydantic v2 / FastAPI / OpenTelemetry |
| **许可证** | Apache-2.0 |
| **核心特性** | 规范化 RAIL 与 Pydantic 校验、Guardrails Hub 开源验证器生态、自愈重问重试机制（Self-Healing Reask）、五级失败处理动作（reask/fix/filter/refrain/exception）、流式输出分块实时校验 |
| **相关关联** | 关联实践: [[大模型端到端输出安全防护：基于 Guardrails 的规范化校验与自愈重试实战]], [[2026-10-20-AI-Digest]], [[Instructor]], [[Outlines]], [[NeMo-Guardrails]], [[Promptfoo]], [[DeepEval]] |

---

## 🚀 核心特性与架构亮点

### 1. 结构化契约与语义规则双重约束（Contract + Semantic Validation）
企业在将大模型接入生产核心系统时，面临双重失控风险：一是“格式破损”（JSON 缺少必填字段、数值溢出或枚举越界导致下游微服务崩溃），二是“语义有害”（输出包含竞品推荐、敏感个人信息 PII 泄露、SQL 注入语法或毫无事实根据的幻觉）。
- **统一规约声明**：Guardrails 支持在标准 Pydantic 模型字段上以声明式注解方式挂载各类语义验证器；
- **分层校验管道**：先执行确定性格式校验（语法树解析、类型转换），再并行执行语义模型校验（语义相似度、毒性探测、实体溯源）。

### 2. 丰富的 Guardrails Hub 生态（社区共享预置验证器）
Guardrails 推出了类似于 Hugging Face Hub 的验证器生态仓库，用户可通过一键 CLI 命令按需安装生产级验证器：
- **安全与合规**：`detect_pii`（敏感实体脱敏）、`sql_column_presence`（SQL 字段存在性校验）、`toxic_language`（有害言论阻断）；
- **真实性与防幻觉**：`provenance_v1`（根据参考文档严格验证生成内容的依据事实）、`hallucination_eval`；
- **代码与格式安全**：`valid_json`（保证闭合无语法错误）、`regex_match`、`competitor_check`。

### 3. 五级失败处理策略（Granular On-Fail Actions）
当验证器拦截到违规输出时，支持配置细粒度的降级或修复行为，避免直接抛出异常导致整个调用崩溃：
1. **`reask`（自愈重问）**：将具体的校验失败原因和当前模型生成物拼接成反馈上下文，自动向大模型发起修正重问；
2. **`fix`（确定性自动纠偏）**：对于微小格式偏差或轻微违规词，由程序进行确定性修复或掩码替换；
3. **`filter`（静默过滤）**：将违规段落或字段从最终字典中剔除；
4. **`refrain`（安全弃权）**：将该字段置为预设默认值或空响应，提示“该信息涉及合规限制无法提供”；
5. **`exception`（即时熔断）**：立即抛出 `ValidationError` 供上层防御模块捕获。

### 4. 流式传输（Streaming）实时拦截
在现代 AI 交互中，流式输出（Server-Sent Events）是提升首字响应体验的刚需。Guardrails 具备流式分块状态机（Chunk-level State Machine），能够在 Token 逐步生成的过程中并行组装并触发前缀校验，一旦检测到高危提示注入或敏感信息泄漏，可在首个违规片段输出前即时切断流式连接。

---

## 🛠️ 快速上手与典型代码示例

### 1. 安装核心库与所需 Hub 验证器

```bash
pip install guardrails-ai
# 安装社区预置验证器
guardrails hub install hub://guardrails/regex_match
guardrails hub install hub://guardrails/detect_pii
guardrails hub install hub://guardrails/provenance_v1
```

### 2. 结合 Pydantic 声明强契约安全防护对象

```python
from pydantic import BaseModel, Field
from typing import List
from guardrails import Guard
from guardrails.hub import DetectPII, ProvenanceV1

# 1. 定义具备运行时内容安全守卫的数据契约
class FinancialAnalysisReport(BaseModel):
    company_name: str = Field(description="企业名称")
    core_metrics: List[str] = Field(description="核心财务指标列表")
    
    # 挂载 PII 敏感信息检测验证器：一旦发现电话/身份证/邮箱，自动在客户端进行脱敏打码
    analyst_comment: str = Field(
        description="分析师核心结论",
        validators=[
            DetectPII(
                pii_entities=["EMAIL_ADDRESS", "PHONE_NUMBER", "CREDIT_CARD"],
                on_fail="fix" # 自动替换为掩码 <REDACTED>
            )
        ]
    )
    
    # 挂载溯源验证器：确保结论严格源于上下文参考资料，杜绝概率幻觉；如失败则自动携带错误反馈重试 2 次
    factual_summary: str = Field(
        description="基于输入研报总结的事实陈述",
        validators=[
            ProvenanceV1(
                threshold=0.85,
                validation_method="sentence",
                on_fail="reask" # 触发自愈重试回路
            )
        ]
    )

# 2. 从 Pydantic 模型实例化 Guard 守卫
guard = Guard.from_pydantic(output_class=FinancialAnalysisReport)

# 3. 执行受保护的 LLM 生成与双向安全拦截
source_document = """
【2026年Q3某科技财报摘录】
公司实现总营收 145.2 亿元，同比增长 28.4%；净利润 32.1 亿元。
分析师张三（联系方式：zhangsan@enterprise.com / 13800138000）指出：
云基础设施业务是核心增长飞轮，增速超 45%。
"""

prompt = """
请基于以下财报摘录输出结构化财务分析报告：
{context}
"""

validated_output = guard(
    model="gpt-4o",
    messages=[{"role": "user", "content": prompt.format(context=source_document)}],
    metadata={"passages": [source_document]},
    max_tokens=1024,
    temperature=0.1
)

# 4. 查看净化且经过强校验的结构化数据
print("=== 结构化合规输出 ===")
print(validated_output.validated_output)
```

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **金融与医疗高合规领域**：需要杜绝任何敏感隐私泄露、必须确保输出结果 100% 具备文献或数据源溯源事实依据的业务系统；
  - **智能客服与自动化运维**：防止大模型生成包含竞品侮辱、恶意攻击性言论或带有破坏性 Bash/SQL 指令的安全底座；
  - **微服务间类型契约消费**：下游业务代码直接反序列化大模型生成的 JSON，要求字段类型零容忍、枚举严格受控的场景。
- **与同类方案对比与选型建议**：
  - **对比 Outlines**：Outlines 侧重于在推理采样阶段通过 Logits 掩码进行强类型正则拦截（底层控制，性能极致但难以处理长语义与外置知识库比对）；
  - **对比 NeMo-Guardrails**：NeMo 侧重于宏观多轮对话的流转控制（Colang 对话策略）；
  - **Guardrails**：聚焦于应用层与模型输出层之间的端到端审计，具备最为丰富的开箱即用规则库与全自动 Reask 自愈协议。
- **综合评估结论**：**生产级 AI 应用必备的合规与输出可靠性防护利器（Adopt）**。
