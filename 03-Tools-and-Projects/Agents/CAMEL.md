---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/camel-ai/camel
stars: "17.8k+"
license: Apache-2.0
date_added: 2026-11-13
---

# 📦 CAMEL

> **一句话简介**：全球首创“自主角色扮演与沟通式智能体社会探索”（Communicative Agents for "Mind" Exploration of Large Scale Society）开源多智能体通用架构。通过首创的“启发式提示（Inception Prompting）”与角色自主博弈、任务规范器（Task Specifier）、层级化工作力编排体系（`Workforce`）以及“代码即提示（Code-as-Prompt）”范式，彻底摆脱人工硬编码交互流程的束缚，实现多智能体的大规模社会化协作、长链任务自治攻坚与高质量合成数据自动生成。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: camel-ai/camel](https://github.com/camel-ai/camel) |
| **研发机构** | CAMEL-AI 开源社区与学术联合团队（由 Guohao Li 等学者于 KAUST/牛津等机构发起） |
| **技术栈** | Python 3.10+, PyTorch, Pydantic, OpenAI/Anthropic/DeepSeek/Qwen SDKs, LiteLLM, vLLM, FastMCP |
| **许可证** | Apache-2.0 License |
| **核心机制** | 角色扮演博弈（Role-Playing）、启发式提示技术（Inception Prompting）、任务规范与解耦器（Task Specifier）、层级化工作团队编排（Workforce）、单智能体与会话节点统一接入（SingleAgentWorker / RolePlayingWorker）、裁判与仲裁员审查（Critic / Moderator）、代码即提示（Code-as-Prompt） |
| **支持模型** | GPT-4o, Claude 3.5 Sonnet, DeepSeek-R1 / V3, Qwen-2.5, LLaMA-3.1，以及通过 Ollama / vLLM 驱动的本地离线模型 |
| **生态定位** | 覆盖学术研究、万级规模智能体社会仿真（Scalable Multi-Agent Societies）、企业级自动化研发流水线与智能体训练合成数据飞轮 |
| **相关关联** | 关联实践: [[多智能体社会化自主协同架构：基于 CAMEL 的 Role-Playing 启发式博弈、动态任务解耦与死循环熔断实战]], [[2026-11-13-AI-Digest]], [[AG2]], [[CrewAI]], [[MetaGPT]], [[LangGraph]], [[PydanticAI]], [[OpenManus]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph HumanInput [人类意图与初始任务]
        RawTask["原始任务/粗粒度构想<br/>(例如: 研发一套基于 FastAPI 的高并发实时风控引擎)"]
    end

    subgraph TaskRefinement [任务规范化与角色启动]
        TaskSpecifier["任务规范器智能体 (Task Specifier)<br/>将模糊需求细化为具备明确边界与交付物的完整任务方案"]
        InceptionPrompting["启发式提示引擎 (Inception Prompting)<br/>注入双方角色互补系统指令、禁制规则与任务终止条件"]
    end

    subgraph WorkforceOrchestrator [CAMEL Workforce 层级化多智能体团队中枢]
        Coordinator["工作流协调中枢 (Coordinator Agent)<br/>全局任务拆解、动态依赖追踪与工单下发调度"]
        
        subgraph WorkerPool [异构智能体工作池 (Worker Pool)]
            subgraph SingleWorkerNode [单体执行单元]
                SingleWorker1["架构师智能体 (SingleAgentWorker)"]
                SingleWorker2["测试审计智能体 (SingleAgentWorker)"]
            end
            
            subgraph RolePlayingWorkerNode [多角色博弈单元]
                AIAssistant["AI 助手 (AI Assistant)<br/>提出具体技术方案与代码实现"]
                AIUser["AI 用户 (AI User)<br/>提出质疑、需求反问与验收评估"]
                AIAssistant <==>|双向自主博弈循环| AIUser
            end
        end

        CriticModerator["裁判与审查员 (Critic / Moderator)<br/>客观度量方案完成度、杜绝幻觉扩散与死循环熔断"]
    end

    subgraph ExecutionAndEnv [工具调用与环境执行层]
        MCPTools["统一工具集 (Web 搜索 / Git / Docker 沙箱 / DB-SQL)"]
        Environment["真实运行沙箱 (代码编译 / 接口测试)"]
    end

    subgraph FinalDeliverable [最终高信噪比成果交付]
        StructuredArtifacts["可验证工程成果 (代码库 / 架构文档 / 测试报告)"]
    end

    RawTask ==> TaskSpecifier
    TaskSpecifier ==> InceptionPrompting
    InceptionPrompting ==> Coordinator
    Coordinator ==> SingleWorker1 & SingleWorker2 & RolePlayingWorkerNode
    RolePlayingWorkerNode ==> MCPTools
    SingleWorker1 & SingleWorker2 ==> MCPTools
    MCPTools ==> Environment
    Environment -.->|执行结果回传| CriticModerator
    CriticModerator ==>|验证通过信号| Coordinator
    Coordinator ==> StructuredArtifacts
```

1. **开创性的“角色扮演与启发式提示（Role-Playing & Inception Prompting）”范式**：
   - 传统的单智能体或流水线式 Agent 容易在多轮对话中逐渐迷失目标，产生角色漂移（Role Drifting）甚至陷入无意义的客套废话循环。
   - CAMEL 首次提出了严格数学形式化的 **Inception Prompting** 框架：通过一个中立的启动提示，自动实例化出具备鲜明职责界限的 **AI User**（代表业务需求提出者、负责提问与验收）与 **AI Assistant**（代表技术执行者、负责拆解方案与编写代码）。两个智能体基于环境反馈和自主博弈互相推进，在极少或零人工介入的情况下，自发收敛至高质量解决方案。
2. **任务规范器（Task Specifier）实现从“模糊构思”到“严密规格”的自动跃迁**：
   - 用户往往只能提供一句宽泛的设想（如“写一个爬虫”）。CAMEL 内置的 Task Specifier 能够自发对原始需求展开多维推演，补全技术选型、异常处理要求、并发指标与接口协议，生成详尽的技术规格说明书（Task Specification），从而在源头上杜绝了由于初始指令模糊导致的多智能体协同偏差。
3. **层级化团队编排引擎：`Workforce` 与混合工作节点**：
   - CAMEL 在最新的架构演进中推出了工业级任务协同中枢 **`Workforce`**。它突破了以往扁平对话的局限，引入了 `Coordinator`（统筹协调者）与动态工单流体系。
   - 在工作池中，支持灵活混编 **`SingleAgentWorker`**（单人单职、专注高效工具调用）与 **`RolePlayingWorker`**（将两个角色的博弈对谈打包为一个独立的黑盒节点处理复杂脑暴）。复杂工程任务被自动切分为有向子任务流，按需分发给最适合的节点攻坚。
4. **客观仲裁与死循环熔断状态机（Critic & Moderator）**：
   - 针对智能体网络常见的“客套死锁（Politeness Deadlock）”与“幻觉共振”，CAMEL 设计了专职的 Critic / Moderator 监督机制。审查员独立于执行节点，基于绝对的目标达成度判定指标（如代码测试通过率、Schema 结构化验证、特定语义终止符）实时监控交互轨迹。一旦监测到对话无实质信息增益，立即强行介入触发熔断，保障系统计算预算的可控性。
5. **大规模智能体社会模拟与训练数据飞轮**：
   - CAMEL 不仅是一个自动化开发框架，更是构建千万级规模数字孪生社会仿真（Multi-Agent Society Simulation）与大模型合成强化学习数据（Agentic Data Synthesis）的事实标准基座。全行业大量前沿模型基于 CAMEL 的角色对谈轨迹进行自我蒸馏与工具使用微调。

---

## 🛠️ 快速上手与配置

### 1. 安装方式

```bash
# 安装 CAMEL 核心多智能体框架
pip install camel-ai

# 若需全套多平台模型集成与高级工具链支持
pip install "camel-ai[all]"
```

### 2. 核心调用示例：基于 Workforce 的高自主多智能体研发团队

```python
import os
from camel.agents import ChatAgent
from camel.models import ModelFactory
from camel.types import ModelPlatformType, ModelType
from camel.societies.workforce import Workforce
from camel.tasks import Task

# 配置环境变量
os.environ["OPENAI_API_KEY"] = "your-api-key"

def run_camel_collaborative_team():
    # 1. 统一创建模型后端基座 (以 GPT-4o 或 DeepSeek-V3 为例)
    model = ModelFactory.create(
        model_platform=ModelPlatformType.OPENAI,
        model_type=ModelType.GPT_4O,
        model_config_dict={"temperature": 0.2}
    )

    # 2. 初始化 Workforce 协同中枢
    # Coordinator 智能体负责解构任务与动态工单分发
    workforce = Workforce(
        description="专业软件架构与代码安全审查团队",
        coordinator_agent_kwargs={"model": model},
        task_agent_kwargs={"model": model}
    )

    # 3. 构造各具系统指令与专长的执行智能体
    architect_agent = ChatAgent(
        system_message="你是一名资深分布式系统架构师。负责系统总体拓扑设计、API 接口定义与高可用方案输出。",
        model=model
    )

    security_agent = ChatAgent(
        system_message="你是一名代码安全审计专家。负责检查系统代码中的 OWASP Top 10 漏洞、SQL 注入风险并提出加固方案。",
        model=model
    )

    # 4. 将智能体作为独立 Worker 挂载至 Workforce
    workforce.add_single_agent_worker(
        description="负责系统底层架构设计、技术选型与数据库 Schema 规划",
        worker=architect_agent
    ).add_single_agent_worker(
        description="负责安全合规性审查、代码边界异常防御与加密安全评估",
        worker=security_agent
    )

    # 5. 提交高复杂度研发任务
    project_task = Task(
        content=(
            "设计一套基于 FastAPI + Redis 的企业级限流中间件。"
            "要求包含滑动窗口算法（Sliding Window Counter）核心实现，"
            "并提供完整的架构设计说明与防并发竞争安全审计方案。"
        ),
        id="task-rate-limiter-001"
    )

    print("🚀 CAMEL Workforce 启动任务协同推进...")
    result_task = workforce.process_task(project_task)

    print("\n✅ 团队协同成果交付：")
    print(result_task.result)

if __name__ == "__main__":
    run_camel_collaborative_team()
```

---

## 💡 工程实战点评与选型建议

- **推荐使用场景**：
  - **复杂开放型软件研发与方案脑暴**：当单一 Prompt 无法穷尽需求边界时，利用 CAMEL 的 Role-Playing 机制让 AI Assistant 与 AI User 进行对立式交叉推演，能够挖掘出常规方法难以察觉的极端边界条件与架构缺陷；
  - **大规模模拟仿真与合成数据生成**：用于生成多轮高对抗性、高质量的 Agentic 对话轨迹，反哺企业专有大模型的后训练与对齐；
  - **层级化企业流程自动化（BPA）**：通过 Workforce 组建结构清晰的虚拟专家组，自主拆解执行多步骤研究报告撰写与竞品技术调研。
- **潜在不足与考量**：
  - **Token 消耗速率较高**：在多角色深入交锋博弈的场景下，上下文轮数呈指数级上升，需严格配置轮数上限（`max_turns`）与总结压缩机制；
  - **依赖高智商基座模型**：Role-Playing 与 Inception Prompting 对基座模型的指令遵循（Instruction Following）与角色代入感要求极高，若在弱小模型（如 7B 以下未对齐模型）上运行，容易发生逻辑退化与死循环。
- **综合评估结论**：**建议引入（首选采用）**。CAMEL 是目前多智能体学术前沿与开源工业落地结合最紧密的框架之一，其层级化 Workforce 与角色博弈理念代表了下一代认知自治系统的核心演进方向。
