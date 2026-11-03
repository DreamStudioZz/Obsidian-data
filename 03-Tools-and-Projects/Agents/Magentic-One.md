---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/microsoft/autogen
stars: "35k+"
license: MIT
date_added: 2026-11-03
---

# 📦 Magentic-One

> **一句话简介**：由微软研究院（Microsoft Research）重磅开源的高性能通用多模态多智能体系统（Generalist Multi-Agent System）。专为解决长链路、跨模态、需要复杂软件与网页交互的开放式任务而设计，首创由中央编排器驱动的“双账本（Task Ledger & Progress Ledger）”容错状态机，并装配 WebSurfer（视觉网页冲浪）、FileSurfer（文件深度导航）、Coder（代码编写）与 ComputerTerminal（环境执行）四大专有子智能体。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: microsoft/autogen (autogen-magentic-one)](https://github.com/microsoft/autogen) |
| **研发机构** | 微软研究院（Microsoft Research & AutoGen Team） |
| **技术栈** | Python, AutoGen Core, Playwright, Docker, AsyncIO |
| **许可证** | MIT License |
| **核心特性** | 集中式中央编排器（Orchestrator）、双账本容错状态机（事实/假设/计划/进度量化）、四大异构专职子智能体解耦协作、多模态屏幕感知与可交互元素锚定、容器化安全终端隔离、自适应死循环熔断与重规划 |
| **相关关联** | 关联实践: [[企业级复杂任务自主解决架构：基于 Microsoft Magentic-One 的多模态多智能体编排与双账本（Task & Progress Ledger）容错实战]], [[2026-11-03-AI-Digest]], [[AG2]], [[MetaGPT]], [[CrewAI]], [[SWE-agent]], [[Browser-Use]], [[OpenHands]] |

---

## 🚀 核心特性与架构亮点

### 1. 中央编排器与双账本状态机（Orchestrator & Dual Ledgers）
在传统多智能体设计（如群体广播对话或线型链式调用）中，随着执行步骤超过 10 步，智能体极易发生上下文丢失、幻觉自我强化、重复执行相同错误指令等崩溃现象。Magentic-One 提出了突破性的双账本治理范式：
- **任务账本（Task Ledger）**：维护宏观任务事实（Facts）、合理假设（Hypotheses）与当前主计划（Plan），确保在漫长的工作流中核心目标不发生任何漂移；
- **进度账本（Progress Ledger）**：记录当前执行步骤（Current Step）、阶段性检查点验收结果（Step Verification）以及死锁计数器（Stall Counter）。当子智能体在某一环节连续失败或产生循环动作时，编排器能依据进度账本在 3 次停滞内强制触发上下文回滚与重规划。

### 2. 四大专职异构子智能体（Specialized Agents）
Magentic-One 坚决避免使用单一庞大的 Prompt 驱动通用智能体，而是针对不同物理环境拆分专职子代理：
1. **WebSurfer（视觉网页冲浪代理）**：基于 Playwright 构建无头/有头浏览器交互环境。具备多模态屏幕解析能力，可将复杂的 DOM 树精简为只包含可交互元素（按钮、输入框、链接）的状态列表，支持点击、滚动、表单提交与页面截图语义理解；
2. **FileSurfer（本地文件解析代理）**：专为本地目录探索与长文档审阅设计。支持目录树遍历、大文件分块预览、文本快速定位与关键元数据提炼，避免将全量文件内容直接喂入主上下文；
3. **Coder（专业代码生成代理）**：专注于算法编写、数据清洗脚本生成、Shell 命令合成与潜在 Bug 的逻辑修复；
4. **ComputerTerminal（沙箱环境执行代理）**：提供状态持久化的安全控制台（本地或 Docker 容器），负责执行 Coder 交付的代码与环境命令，捕获标准输出（stdout）、标准错误（stderr）及返回码，并实时回传给编排器。

### 3. 外环规划与内环修正的双层闭环（Outer-Inner Loop Paradigm）
- **外环循环（Outer Loop）**：编排器根据全局任务账本更新计划状态，评估当前所处阶段，并将下一个细分目标派发给最匹配的子智能体；
- **内环流转（Inner Loop）**：被选中的子智能体在受限的环境中执行微观动作（例如 WebSurfer 打开网页并检索关键信息，或 Coder 编写代码交由 Terminal 运行），将执行轨迹压缩为结构化事实摘要后交还编排器；
- 这种双层解耦设计使得系统既具备处理长达数十步复杂任务的宏观规划力，又具备在具体工具调用时的微观敏捷性。

---

## 🛠️ 快速上手与典型代码示例

### 1. 依赖安装与浏览器环境准备

```bash
# 安装 Magentic-One 核心依赖 (AutoGen v0.4+ 体系)
pip install autogen-magentic-one autogen-ext[openai]

# 安装 Playwright 浏览器驱动
playwright install --with-deps chromium
```

### 2. 启动 Magentic-One 多智能体团队解决综合任务

```python
import asyncio
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_magentic_one.agents import (
    MagenticOneOrchestrator,
    WebSurfer,
    FileSurfer,
    Coder,
    ComputerTerminal,
)
from autogen_magentic_one.core import RoundRobinGroupChat

async def main():
    # 1. 初始化高性能大模型底座客户端（推荐具备多模态与代码能力的模型）
    model_client = OpenAIChatCompletionClient(
        model="gpt-4o",
        # 支持兼容 OpenAI 协议的本地模型 (如 vLLM / LiteLLM / DeepSeek)
        # api_key="sk-...",
        # base_url="http://localhost:8000/v1"
    )

    # 2. 实例化专职子智能体
    web_surfer = WebSurfer(name="WebSurfer", model_client=model_client)
    file_surfer = FileSurfer(name="FileSurfer", model_client=model_client)
    coder = Coder(name="Coder", model_client=model_client)
    terminal = ComputerTerminal(name="ComputerTerminal")

    # 3. 实例化具有双账本治理能力的中央编排器
    orchestrator = MagenticOneOrchestrator(
        model_client=model_client,
        agents=[web_surfer, file_surfer, coder, terminal],
        max_turns=30,           # 最大交互轮数上限
        max_stalls_before_replan=3 # 连续 3 次无有效进展时强制触发重规划
    )

    # 4. 下发一个需要联网检索、文件处理与本地脚本验证的复合任务
    complex_prompt = (
        "请访问 Hugging Face 官网，查找当前 Trending 前 3 的开源视觉语言模型（VLM）；"
        "分析它们的技术报告链接与参数量差异，并在本地生成一份 Markdown 对比报告 benchmark_vlm.md；"
        "最后在终端执行 Python 脚本验证本地目录中确实已存在该报告，且文件非空。"
    )

    print("🚀 启动 Magentic-One 多智能体任务流...")
    # 5. 执行编排器主循环
    await orchestrator.run(task=complex_prompt)
    print("✅ 任务完成，各子智能体协作链路结束。")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **跨系统深度自主调研与综合交付**：需要同时结合外网信息检索（WebSurfer）、内部历史文档分析（FileSurfer）和本地数据运算（Coder & Terminal）的自动化复杂研究报告生成；
  - **自动化端到端测试与真实环境操作**：模拟真实用户在 Web 页面中操作、并结合后端数据库与代码进行集成验证的场景；
  - **长链路自主软件研发辅助**：面对未知依赖报错，子智能体能通过搜索引擎查阅最新文档、修改本地代码并在沙箱中反复编译测试直至通过。
- **与同类工具对比（vs SWE-agent / MetaGPT / CrewAI）**：
  - 相比于 **MetaGPT**：MetaGPT 依赖较为固化的预设 SOP（需求->架构->代码），而 Magentic-One 的双账本编排器支持高弹性的动态重规划与自主纠错；
  - 相比于 **CrewAI**：CrewAI 在长文本与长链路交互中容易产生代理自言自语与死循环，Magentic-One 的进度账本具备数学化的停滞判定（Stall Detection）与强制熔断机制；
  - 相比于 **SWE-agent**：SWE-agent 专门针对代码仓库 Issue 修复进行了极度特化，而 Magentic-One 是覆盖 Web、本地文件与终端运行的通用全模态系统。
- **个人评估结论**：**强烈推荐采纳 (status/adopted)**。代表了 2026 年通用长链路多智能体编排的最前沿工业级设计，双账本容错状态机是企业自研 Agent 网关的重要参考标准。
