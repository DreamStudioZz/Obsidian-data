---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Inference
github: https://github.com/lm-sys/RouteLLM
stars: "5.2k+"
license: Apache-2.0
date_added: 2026-09-30
---

# 📦 RouteLLM

> **一句话简介**：LMSYS 开源的大模型动态路由与请求分流框架，能够在低成本小模型（如 Llama-3-8B、Haiku）与顶级大模型（如 GPT-4o、Claude 3.5 Sonnet）之间智能分流，以极小的质量折损换取高达 70%~85% 的推理成本节省。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: lm-sys/RouteLLM](https://github.com/lm-sys/RouteLLM) |
| **核心特点** | 强弱模型动态路由、自适应成本-质量帕累托前沿调控、兼容 OpenAI API 代理、内置四大开源路由算法 |
| **技术栈** | Python / PyTorch / HuggingFace Transformers / FastEval |
| **关联实践** | [[大模型工程降本利器：基于 RouteLLM 的动态强弱模型路由与成本节省实战]], [[私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构]], [[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]] |

---

## 🚀 核心特性与技术亮点

1. **工业级动态模型路由算法矩阵**：
   - 内置四大核心路由器，兼顾延迟、显存与准确度：
     - **Matrix Factorization (MF)**：利用用户-偏好协同过滤与 Elo 竞技场历史对决矩阵，极速预测弱模型成功率（微秒级开销）；
     - **BERT Classifier**：基于轻量化 Transformer 判别输入复杂度；
     - **Causal LLM**：使用紧凑因果语言模型生成自适应路由决策；
     - **Random / Similarity Router**：作为基准测试与基于向量距离的领域路由。
2. **直观的成本-质量可控阈值（Threshold Calibration）**：
   - 提供标准化的阈值调节系数（`0.0 ~ 1.0`）。开发者可按业务容忍度自由调整：
     - 阈值设为 `0.115` 时：路由至弱模型的请求占比达 50% 以上，平均成本直降 50%，整体评测指标保持在强模型的 95% 以上；
     - 阈值设为 `0` 时：全量路由至弱模型，成本最低；
     - 阈值设为 `1` 时：全量路由至强模型，质量最高。
3. **即插即用的 OpenAI 兼容反向代理服务**：
   - 提供开箱即用的 HTTP 代理服务，完全兼容 OpenAI `/v1/chat/completions` 接口规范；
   - 业务端只需修改 `base_url` 与 `model: "router-mf-gpt-4o-haiku"`，无需更改任何业务代码即可享受全自动降本分流。
4. **与 LMSYS Chatbot Arena 数据集直接对齐**：
   - 训练数据深度融合了全球数百万开发者在 Chatbot Arena 上的真实盲测交互，精准识别不同任务（如简单常识问答 vs 复杂逻辑推演）对模型能力边界的差异需求。

---

## 🛠️ 快速上手与集成

### 1. 安装 RouteLLM

```bash
pip install "routellm[serve,eval]"
```

### 2. Python SDK 核心调用示例

```python
import os
from routellm.controller import Controller

# 配置强弱模型 API Key
os.environ["OPENAI_API_KEY"] = "sk-..."
os.environ["ANTHROPIC_API_KEY"] = "sk-..."

# 初始化控制器，配置强模型与弱模型对
client = Controller(
    routers=["mf"],  # 选择 Matrix Factorization 高速矩阵分解路由器
    strong_model="claude-3-5-sonnet-20241022",
    weak_model="claude-3-haiku-20240307",
    config={
        "mf": {
            "checkpoint": "routellm/mf_gpt4_augmented"
        }
    }
)

# 发起带有路由判断的对话生成
response = client.chat.completions.create(
    model="router-mf-0.115",  # 设定阈值 0.115，平衡成本与质量
    messages=[
        {"role": "user", "content": "帮我写一个 Python 正则表达式，匹配国内大陆手机号。"}
    ]
)

# 查看最终由哪个模型实际生成了结果
print(f"实际执行模型: {response.model}")
print(response.choices[0].message.content)
```

### 3. 作为独立网关服务启动

```bash
# 启动兼容 OpenAI 规范的本地/线上路由网关
python -m routellm.server \
    --routers mf \
    --strong-model gpt-4o \
    --weak-model meta-llama/Llama-3-8b-chat \
    --port 8000
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 日请求量数百万级别的大规模企业级在线服务（如智能客服、知识库基础问答、内容初筛）；
  - 任务复杂度方差极大的场景：大部分是简单格式转换、分类、摘要，小部分是复杂多步推理与代码生成；
  - 配合私有化轻量部署架构，将简单请求本地 Ollama/vLLM 消化，疑难复杂请求转发云端顶级商业 API。
- **潜在不足 / 局限性**：
  - 路由决策需要前置增加 2~10ms 的轻微网络或计算延迟，对亚毫秒级严苛时延场景需评估开销；
  - 极高安全或法律审计场景下，若任务要求 100% 必须由特定型号模型输出，动态路由可能会引入合规审计差异。
- **评估结论**：**极力推荐引入（Adopted）**。在大模型工程落地从“跑得通”走向“跑得省”的精细化运营阶段，RouteLLM 是实现 ROI 翻倍的标杆基础设施。
