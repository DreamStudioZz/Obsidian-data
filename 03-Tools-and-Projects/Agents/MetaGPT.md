---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/geekan/MetaGPT
stars: "45k+"
license: MIT
date_added: 2026-10-21
---

# 📦 MetaGPT

> **一句话简介**：基于**标准操作规程（SOP, Standard Operating Procedures）**的多智能体协同软件研发框架。以“Code = SOP(Team)”为核心设计哲学，将人类成熟的软件工程工作流编码进多智能体系统中，通过模拟全功能软件开发团队（产品经理、系统架构师、项目经理、研发工程师、测试工程师），以严谨的**结构化文档（PRD、系统设计 UML、任务依赖图）**与**发布-订阅环境**取代无序的自然语言闲聊，实现从“一句自然语言需求”到“完整可运行代码仓库”的全自动端到端交付。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: geekan/MetaGPT](https://github.com/geekan/MetaGPT) |
| **技术栈** | Python 3.9+ / Pydantic v2 / AsyncIO / Mermaid / Git / Pytest |
| **许可证** | MIT License |
| **核心特性** | 软件工程 SOP 驱动工作流、结构化产出（PRD/UML/Tasks）、基于发布-订阅的消息总线环境、全角色软件公司模拟、增量代码演进与自动化测试修复 |
| **相关关联** | 关联实践: [[企业级 SOP 驱动多智能体协同研发：基于 MetaGPT 的标准化角色装配与结构化产出交付实战]], [[2026-10-21-AI-Digest]], [[CrewAI]], [[LangGraph]], [[OpenHands]], [[AG2]], [[Aider]] |

---

## 🚀 核心特性与架构亮点

### 1. “Code = SOP(Team)” 哲学：以流程制度规范智能体协同
在开放域多智能体系统（如 AutoGen、传统 Chat 团队）中，多个智能体直接通过自由对话进行沟通，极易出现三大致命顽疾：**上下文语义漂移（Drift）**、**职责推诿与死循环闲聊（Infinite Chatter）**以及**幻觉级联放大（Hallucination Cascading）**。
- **SOP 约束行为空间**：MetaGPT 借鉴人类现代软件工程的 SOP 思想，为每个角色定义输入条件、思维过程、输出规约与交付标准。
- **阶段性质量门禁**：前一个角色的输出构成了下一个角色的严格输入约束（例如 Architect 必须依据 Product Manager 的 PRD 进行数据结构设计，禁止自行脑补需求）。

### 2. 结构化产出物（Structured Artifacts）取代非受控文本
MetaGPT 拒绝以冗长、模糊的纯自然语言作为中间交付件，而是引入强类型 Pydantic Schema 与标准工程中间件：
- **产品经理（Product Manager）**：输出符合行业标准的结构化 PRD，包含竞品分析、用户痛点、功能用例（User Stories）与优先级；
- **系统架构师（Architect）**：输出 Mermaid 类图（Class Diagram）、系统时序图（Sequence Diagram）与目录文件结构设计；
- **项目经理（Project Manager）**：依据架构设计输出模块依赖图（Task Dependency Graph）与甘特分解；
- **工程师（Engineer）**：针对具体的单个源文件，严格依据接口规范编写高质量、带类型注解的 Python 代码；
- **QA 测试员（Quality Engineer）**：自动编写 pytest 单测用例，并在沙箱中真实执行，捕获报错 Traceback 驱动工程师自愈反思。

### 3. 基于“发布-订阅（Pub-Sub）”的共享协作环境（Environment）
MetaGPT 构建了一个统一的消息总线环境（`Environment`），智能体之间并不直接互相持有引用或发起点对点轮询：
- **按需监听订阅**：每个 `Role` 声明自己感兴趣的消息类型（如 `Action`、`Message.cause_by`）。例如架构师仅监听 `WritePRD` 事件；
- **解耦防膨胀**：避免了将全量团队对话一股脑塞入每个智能体的 Prompt 上下文中，使得上下文窗口利用率提升 80% 以上，极大降低了 Token 消耗并保障了长流程稳定收敛。

### 4. 增量代码演进与软件全生命周期覆盖
MetaGPT 不仅支持从零生成项目（Greenfield），更支持在已有代码库上执行**需求迭代与增量开发（Brownfield / Incremental Development）**：
- 通过分析 Git 历史与 AST 语法树，理解项目现状并定位需要修改的文件切片；
- 支持代码审查（Code Review）双智能体结对模式，由 Senior Engineer 审查 Junior Engineer 生成的代码逻辑与漏洞。

---

## 🛠️ 快速上手与典型代码示例

### 1. 依赖安装与环境准备

```bash
# 安装 MetaGPT 核心包
pip install metagpt

# 初始化配置文件并填入大模型 API Key
metagpt --init-config
# 编辑 ~/.metagpt/config2.yaml 填入 LLM 配置（OpenAI, Claude, 或本地私有化端点）
```

### 2. 终端一键启动端到端全自动软件研发团队

仅需一行 CLI 命令，MetaGPT 即可协同 5 个角色自动输出完整的 PRD、设计图、任务清单及代码：

```bash
# 启动全自动团队，端到端编写一个基于 Pygame 的 2048 游戏
metagpt "编写一个包含精美 UI、平滑动画过渡与最高分记录的 2048 桌面游戏" --project-name "game_2048" --run-tests
```

执行后，MetaGPT 会在 `workspace/game_2048` 目录下生成：
- `docs/prd.md`：完整的产品需求规格说明书
- `docs/system_design.md`：含 Mermaid 类图与接口时序的系统设计方案
- `docs/tasks.md`：任务分解依赖列表与执行顺序
- `game_2048/`：可直接运行的工程源代码与单测代码

### 3. Python SDK 自定义 SOP 角色流水线（需求分析 -> 架构设计）

```python
import asyncio
from metagpt.actions import Action
from metagpt.roles import Role
from metagpt.schema import Message
from metagpt.environment import Environment

# 1. 定义产品经理角色
class ProductAnalyst(Role):
    name: str = "Alice"
    profile: str = "Product Manager"

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        # 绑定核心行为：输出标准结构化 PRD
        self.set_actions([WritePRDAction])
        self._watch([UserRequirement])

# 2. 定义系统架构师角色
class SystemArchitect(Role):
    name: str = "Bob"
    profile: str = "Architect"

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        # 绑定核心行为：输出系统设计图与数据模型
        self.set_actions([WriteDesignAction])
        # 架构师仅订阅 Alice 产出的 PRD 消息
        self._watch([WritePRDAction])

# 3. 组建 SOP 流水线环境
async def run_pipeline():
    env = Environment()
    analyst = ProductAnalyst()
    architect = SystemArchitect()
    
    env.add_roles([analyst, architect])
    # 注入用户需求
    env.publish_message(
        Message(content="为企业设计一套支持多租户与 RBAC 权限的知识库问答后端服务", send_to="Alice")
    )
    
    # 按照 SOP 驱动多轮反应
    while not env.is_idle:
        await env.run()

if __name__ == "__main__":
    asyncio.run(run_pipeline())
```

---

## 💡 工程实战点评与适用场景

### 1. 推荐使用场景
- **端到端中小型工程自动化脚手架生成**：从自然语言需求一键输出包含标准设计文档、类图、工程结构与基准代码的完整项目；
- **企业复杂业务 SOP 流程自动化**：金融风控审计流程、合规审查流水线、研报多角色撰写等强规则、强分工的多人协同场景；
- **代码重构与单测补全**：利用 QA Role 与 Reviewer Role 对现有工程代码进行扫描，自动补充覆盖率 80%+ 的单测套件。

### 2. 潜在不足与避坑考量
- **超长大型项目（>10,000 行代码）的上下文衰减**：在超大规模项目中，单靠文件拆解可能会丢失全局依赖，需结合 GraphRAG 或代码知识图谱（如 [[Graphify]]）进行上下文注水；
- **Token 消耗相对集中**：由于模拟全套软件开发生命周期，每个阶段均会生成大篇幅 Markdown / UML 规范，建议搭配高性价比推理模型（如 DeepSeek、Claude-3.5-Sonnet）或启用模型路由；
- **执行环境沙箱隔离必须严守**：QA 工程师在测试执行生成的代码时，务必置于 Docker 容器或 MicroVM 沙箱（如 [[E2B]]、[[OpenShell]]）内运行，防范恶意脚本逃逸。

### 3. 个人评估结论
- **建议引入（Adopted）**。MetaGPT 是业内将“软件工程标准”与“多智能体协作”结合得最严密、成熟度最高的开源项目之一。其提出的 SOP 范式和结构化中间产出思想，彻底终结了纯聊天式 Agent 的不稳定性，为企业构建复杂自主工程团队提供了教科书级范本。
