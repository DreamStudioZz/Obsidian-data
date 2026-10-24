---
tags:
  - moc
  - index
  - home
---

# 🧠 个人 AI 知识库仪表盘 (AI Knowledge Dashboard)

> [!TIP] 💡 Dataview 动态渲染说明
> 本页面已内置 **Dataview 动态查询组件**。
> 只要在 Obsidian 处于 **实时预览（Live Preview）** 或 **阅读视图（Reading View）**，下方表格就会自动从各个文件夹中扫描提取最新卡片并实时渲染成表格，无需手动维护！

---

## 🛠️ 开源工具与框架动态库 (Tools & Frameworks)

```dataview
TABLE category AS "分类", github AS "GitHub 仓库", date_added AS "收录日期"
FROM "03-Tools-and-Projects"
WHERE file.name != "Template-Project-Card"
SORT date_added DESC, file.name ASC
```

---

## 💡 工程实践与架构设计库 (Engineering Practices)

```dataview
TABLE domain AS "技术领域", difficulty AS "难度", date_added AS "收录日期"
FROM "04-Engineering-Practices"
WHERE file.name != "Template-Practice-Card"
SORT date_added DESC, file.name ASC
```

---

## 📰 最近 AI 日报与前沿雷达 (Recent Digests)

```dataview
TABLE date AS "发布日期", summary AS "核心导读"
FROM "01-Daily-Digest"
WHERE file.name != "Template-Daily-Digest"
SORT file.name DESC
LIMIT 7
```

---

## ⚡ 快捷专题核心脉络

### 💻 本地部署与高性能推理 (Inference & Deployment)
- 现代推理与微调基座：[[TRL]]（Hugging Face 官方大模型强化学习与后训练对齐底座、GRPO 推理思维链涌现与 RLVR 可验证规则奖励引擎）, [[Xinference]]（工业级分布式异构模型统一推理与多模态服务底座、vLLM/llama.cpp/Transformers 自适应编排与高可用主从集群）, [[Text-Embeddings-Inference]]（Hugging Face 官方 Rust 极速向量化与重排连续批处理引擎、FlashAttention-2 融合核）, [[TensorRT-LLM]]（NVIDIA 官方工业级极限吞吐推理引擎、In-Flight Batching 与 FP8 编译基座）, [[llama.cpp]]（纯 C/C++ 零依赖、GGUF 量化与投机解码基石）, [[Ollama]]（本地私有化引擎）, [[vLLM]]（PagedAttention 高并发）, [[SGLang]]（RadixAttention 前缀缓存）, [[Outlines]]（基于 DFA/FSM 的 Logits 掩码约束与零幻觉结构化输出引擎）, [[Instructor]]（基于 Pydantic 的多模型强类型数据契约、自愈重试与流式增量解析引擎）, [[Unsloth]]（Triton 手写内核单卡极限微调与 GRPO 对齐）, [[RouteLLM]]（LMSYS 动态强弱模型路由网关与自适应降本）, [[LLaMA-Factory]]（工业级全功能微调与 DPO 强化学习对齐中枢）
- 降本加速实战：[[大模型强化学习与推理对齐：基于 TRL 的 GRPO 算法调优与可验证奖励函数 (RLVR) 工程实战]], [[分布式异构模型统一服务化架构：基于 Xinference 的多后端协同调度与一站式多模态推理网关实战]], [[企业级 RAG 向量化与重排加速：基于 Text Embeddings Inference (TEI) 的连续批处理与微秒级服务实战]], [[企业级极限推理加速：基于 NVIDIA TensorRT-LLM 的 In-Flight Batching 与 FP8 量化部署实战]], [[大模型端到端强类型输出：基于 Instructor 与 Pydantic 的流式结构化抽取与自愈重试实战]], [[大模型结构化输出保障：基于 Outlines 的 Logits 掩码约束与零幻觉提取实战]], [[大模型高效微调与强化学习对齐：基于 LLaMA-Factory 的多模型全流程训练实战]], [[大模型工程降本利器：基于 RouteLLM 的动态强弱模型路由与成本节省实战]], [[端侧高吞吐低延迟推理：基于 llama.cpp 的 GGUF 极限混合量化与投机解码工程实战]], [[私有化大模型基础设施：基于 Ollama 与 LiteLLM 的高可用网关架构]], [[大模型工程降本提速：Prompt Caching 架构设计与最佳实践]], [[大模型极致后训练调优：基于 Unsloth 的显存节省与 LoRA 极速微调工程实战]]

### 🤖 智能体与应用编排平台 (Agents & Workflow Platforms)
- 应用中台与工作流平台：[[Open-WebUI]]（全球领先的企业级私有化 AI 交互中枢、离线多模型统管客户端与 Pipelines 自定义插件网关）, [[Langflow]]（面向多智能体与复杂 RAG 的开源可视化低代码流水线编排、Python 原生组件与生产级 FastAPI 导出框架）, [[Semantic-Kernel]]（微软官方多语言企业级智能体编排底座、强类型插件与三层过滤器治理中枢）, [[LlamaIndex]]（事实标准数据连接、属性图混合索引与 Workflows 事件驱动多智能体编排底座）, [[Dify]]（可视化工作流与 Agent 中台）
- 自主工程师与多智能体：[[MetaGPT]]（标准操作规程 SOP 驱动的多智能体协同研发标杆、发布-订阅环境与结构化全生命周期交付）, [[Agno]]（原 Phidata、高性能纯 Python 多模态多智能体团队编排底座、微秒级运行时与原生状态持久化）, [[Graphiti]]（Zep 团队开源的智能体动态时序知识图谱与双时态长效记忆底座）, [[Letta]]（类操作系统分层记忆与有状态 Agent 运行时）, [[AG2]]（生产级分层多智能体协同与对话编排底座）, [[Cline]]（IDE 原生自治编程与 MCP 扩展先锋）, [[Aider]]（终端 Git 原生架构师结对）, [[OpenHands]]（SWE 级自主编程）, [[CrewAI]]（角色协作与事件流）, [[smolagents]]（代码驱动）, [[LangGraph]]（状态图）, [[PydanticAI]]（类型安全）
- 协议、沙箱与终端 Harness：[[OmniParser]]（微软纯视觉屏幕解析与 GUI 智能体定位引擎）, [[CopilotKit]]（应用内置 Copilot、生成式 UI 与 Co-Agent 双向状态同步框架）, [[Composio]]（生产级 1000+ 工具与 OAuth 托管调用基础设施）, [[OpenShell]]（NVIDIA 内核级 eBPF 动态系统调用与网络隔离沙箱）, [[E2B]]（Firecracker MicroVM 极速代码沙箱与安全解释器）, [[FastMCP]], [[Pi-Agent]], [[Browser-Use]]
- 实战沉淀：[[企业级私有化大模型统一交互中枢：基于 Open-WebUI 的多模型纳管、知识库 RAG 与 Pipelines 自定义中间件实战]], [[企业级可视化智能体编排：基于 Langflow 的低代码组件定制与生产级 API 服务化实战]], [[企业级 SOP 驱动多智能体协同研发：基于 MetaGPT 的标准化角色装配与结构化产出交付实战]], [[高性能多模态智能体架构：基于 Agno 的轻量化多智能体团队编排与持久化记忆实战]], [[智能体时序记忆架构：基于 Graphiti 的动态时序知识图谱与状态演化检索实战]], [[企业级多语言智能体编排：基于 Microsoft Semantic Kernel 的插件化架构与过滤器拦截实战]], [[基于 LlamaIndex Workflows 的事件驱动型多智能体编排与异步 RAG 流水线实战]], [[纯视觉 GUI 智能体屏幕解析：基于 Microsoft OmniParser 的可交互元素检测与坐标基准定位实战]], [[端侧智能体与生成式 UI：基于 CopilotKit 与 LangGraph 的双向状态同步与组件流式渲染实战]], [[Agent 生产级工具集成：基于 Composio 的企业级 OAuth 托管与高可靠 Action 执行实战]], [[有状态智能体架构：基于 Letta 的分层记忆管理与持久化运行时实战]], [[多智能体生产级协同：基于 AG2 的分层编排与会话状态持久化实战]], [[Agent 内核级安全隔离：基于 OpenShell 的 Syscall 过滤与网络白名单防御实战]], [[Agent 安全沙箱隔离：基于 E2B MicroVM 的代码解释器与无感运行时环境实战]], [[IDE 智能编程范式：基于 Cline 与 MCP 协议的上下文工程与自主开发闭环实战]], [[终端代码智能体实践：基于 Aider 的 Repo Map 压缩与 Architect-Editor 双模型分工实战]], [[基于 Dify 的企业级 Agent 工作流与 RAG 落地架构实践]], [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[多智能体协同工程：基于 CrewAI 的角色编排与层级流（Flows）实践]], [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]], [[多模态网页自动化：基于 Browser-Use 的视觉驱动 Agent 设计与工程避坑]]

### 🔍 深度文档解析与混合检索 (Advanced RAG & Knowledge Layer)
- 版面解析与向量引擎：[[Haystack]]（deepset 工业级强类型 DAG 检索流水线编排底座、OpenSearch/Qdrant 多路混合召回与 RRF 融合重排引擎）, [[Chroma]]（开源轻量级 AI 原生嵌入式向量数据库、无服务器本地持久化与元数据多维过滤基石）, [[FlagEmbedding]]（北京智源 BAAI 工业级语义表征标杆、BGE-M3 三模态统一检索与 BGE-Reranker-v2 深度重排底座）, [[Vanna]]（针对复杂数仓的开源三维元数据 RAG 增强与自纠错 Text-to-SQL 引擎）, [[ColPali]]（突破性多模态视觉端到端文档检索底座、Patch 多向量与 ColBERT 迟交互引擎）, [[Weaviate]]（云原生高并发动态多租户与 BQ/PQ 极致压缩向量检索底座）, [[LlamaIndex]]（全策略索引与属性图混合检索底座）, [[Crawl4AI]]（开源高吞吐无头网页抓取与大模型友好型结构化提取引擎）, [[Kotaemon]]（生产级可定制 RAG 文档问答与可视化溯源底座）, [[Milvus]]（云原生分布式百亿级向量数据库与混合检索基石）, [[Qdrant]]（纯 Rust 云原生向量数据库与 INT8 标量过滤引擎）, [[MinerU]]（学术论文与复杂研报高保真 LaTeX/表格抽取）, [[AnythingLLM]]（开箱即用桌面/单容器知识库）, [[RAGFlow]]（深度版面理解与零幻觉溯源）, [[Docling]]（复杂版面解析）, [[LightRAG]]（双层知识图谱）, [[Graphify]]（本地代码图谱）, [[Firecrawl]]（网页智能清洗与反爬提取）, [[LanceDB]]（无服务器嵌入式列式向量检索）
- 实战沉淀：[[企业级强类型 DAG 检索流水线：基于 Haystack 2.0 的混合多路召回与自适应路由工程实战]], [[轻量级嵌入式向量检索架构：基于 Chroma 的无服务器本地持久化与多维元数据过滤实战]], [[企业级 RAG 混合检索与重排精排：基于 FlagEmbedding (BGE-M3 & Reranker) 的三模态语义召回与交叉评分实战]], [[企业级 Text-to-SQL 实战：基于 Vanna 的三维元数据 RAG 检索与自纠错执行流水线]], [[多模态文档检索革命：基于 ColPali 的视觉无损分块与 ColBERT 迟交互检索实战]], [[企业级多租户海量向量检索：基于 Weaviate 的动态租户隔离与 PQ-BQ 标量压缩实战]], [[基于 LlamaIndex Workflows 的事件驱动型多智能体编排与异步 RAG 流水线实战]], [[大模型友好型高效网页抽取：基于 Crawl4AI 的异步无头爬取与自适应分块实战]], [[企业级精准文档问答：基于 Kotaemon 的多模态混合检索与可溯源引用架构实战]], [[企业级百亿向量分布式检索：基于 Milvus 的分区键切分与稠密稀疏混合检索实战]], [[生产级高性能向量检索：基于 Qdrant 的 Payload 过滤与标量量化混合检索实战]], [[复杂文档高保真结构化：基于 MinerU 的论文研报公式还原与跨页版面解析实践]], [[企业级 RAG 深度文档解析：基于 RAGFlow 的视觉模板分块与防幻觉实战]], [[双层图谱增强检索：基于 LightRAG 的轻量化 GraphRAG 架构实践]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]], [[企业级 RAG 外部数据源清洗：基于 Firecrawl 的反爬对抗与智能 Markdown 结构化抽取实践]], [[轻量无服务器检索架构：基于 LanceDB 的列式存储与端侧混合检索实战]]

### 🧪 大模型评测、合规与质量安全 (Evaluation, Quality & Safety Guardrails)
- 质量评估与安全合规底座：[[Guardrails]]（Guardrails AI 开源结构化输出校验、运行时内容安全防护与自愈重试引擎）, [[AgentOps]]（专为多智能体打造的开源全链路可观测、会话时间旅行回放与死循环熔断底座）, [[Phoenix]]（OpenTelemetry 原生 Agent/RAG 全链路可观测与嵌入漂移评测中枢）, [[Promptfoo]]（轻量级 LLM 安全红队对抗、越狱扫描与 CI/CD 自动化门禁）, [[Ragas]]（RAG 与 Agent 管道四维量化度量与合成测试集生成）, [[NeMo-Guardrails]]（Colang 2.0 声明式安全护栏与运行时对齐）, [[DeepEval]]（大模型单元测试与 CI-CD 门禁）, [[Langfuse]]（全链路可观测与追溯）, [[DSPy]]（算法化提示词编译）
- 实战沉淀：[[大模型端到端输出安全防护：基于 Guardrails 的规范化校验与自愈重试实战]], [[自主智能体全链路可观测与可复现调试：基于 AgentOps 的会话回放与故障瀑布分析实战]], [[Agent 与 RAG 全链路可观测性：基于 Arize Phoenix 的 OpenTelemetry 追踪与嵌入漂移评测实战]], [[LLM 安全合规与防越狱红队测试：基于 Promptfoo 的自动化脆弱性扫描与 CI 门禁实战]], [[RAG 体系化量化评估：基于 Ragas 的核心四维指标与自动化合成测试集实战]], [[企业级大模型可编程安全底座：基于 NeMo Guardrails 的对话护栏与防越狱拦截实战]], [[大模型应用质量工程：基于 DeepEval 的端到端指标评测与 CI-CD 自动化门禁实战]], [[Agent 生产级可观测性与评估：基于 Langfuse 的全链路追踪实践]], [[告别手工调优：基于 DSPy 的提示词与 Few-Shot 自动编译优化实践]]

---

## 🗺️ 知识库常规目录导航

| 模块 | 目录路径 | 说明 |
| :--- | :--- | :--- |
| 📰 **每日雷达 (Daily Digest)** | `[[01-Daily-Digest]]` | 每日由 AI 自动收集的精选工程实战技巧、新开源工具与行业动向 |
| 🏗️ **工程实践 (Practices)** | `[[04-Engineering-Practices]]` | Agent设计模式、RAG深度调优、Prompt最佳工程、本地部署加速 |
| 📦 **开源工具 (Tools & Projects)** | `[[03-Tools-and-Projects]]` | GitHub / HuggingFace 上经过验证的高质量框架与实用工具库 |
| 🧩 **核心概念 (Core Concepts)** | `[[02-Core-Concepts]]` | LLM原理、注意力机制、量化原理、上下文压缩等核心技术内功 |
| 📚 **精读与资料 (Resources)** | `[[05-Papers-and-Resources]]` | 经典论文笔记、优质博客、官方技术报告整理 |
| 📑 **卡片模板 (Templates)** | `[[99-Templates]]` | 日报模板、开源工具卡片、实践卡片规范 |
| 📘 **插件教程 (Tutorial)** | `[[Dataview插件使用说明与实战]]` | Dataview 语法、核心用法与拓展指令速查 |
