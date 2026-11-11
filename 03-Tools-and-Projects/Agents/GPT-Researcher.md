---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/assafelovic/gpt-researcher
stars: "20.5k+"
license: Apache-2.0
date_added: 2026-11-11
---

# 📦 GPT-Researcher

> **一句话简介**：全球最具影响力的开源自主深度研究智能体（Autonomous Deep Research Agent）框架。专为解决传统 RAG 与单步搜索在应对宏观、多维度、高信噪比复杂调研时的局限而生。通过首创的“分层多智能体协同（Hierarchical Multi-Agent）”架构，实现从宏观任务拆解、多源并行检索清洗、事实交叉验证到万字级长篇学术与工业研报结构化合成的全自动闭环，彻底重新定义了 AI 时代的深度调研生产力。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: assafelovic/gpt-researcher](https://github.com/assafelovic/gpt-researcher) |
| **研发机构** | Assaf Elovic 及全球开源社区 |
| **技术栈** | Python 3.10+, LangChain / LangGraph, FastAPI, Pydantic, Playwright, BeautifulStoneSoup |
| **许可证** | Apache-2.0 License |
| **核心机制** | 分层多智能体协同（Master Planner -> Sub-Topic Researchers -> Fact Verifiers -> Chief Publisher）、多源异构搜索网格（Tavily, SearXNG, DuckDuckGo, Bing, Google, arXiv）、正文无头提取与语义清洗、确定性来源溯源与内联引用、自适应多章节大纲编排 |
| **模型支持** | 支持 OpenAI (GPT-4o/o3-mini)、Anthropic (Claude 3.5/3.7 Sonnet)、DeepSeek (DeepSeek-V3/R1)、Ollama / vLLM 本地大模型等 |
| **导出格式** | 原生支持 Markdown, PDF, DOCX, 结构化 JSON 等多格式排版渲染 |
| **相关关联** | 关联实践: [[自主深度研究智能体架构：基于 GPT-Researcher 的分层多智能体并行检索、事实交叉验证与长篇综述自适应生成实战]], [[2026-11-11-AI-Digest]], [[OpenManus]], [[Browser-Use]], [[Crawl4AI]], [[Haystack]], [[LangGraph]], [[Kotaemon]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph UserLayer [用户输入层]
        UserInput["宏观调研指令<br/>(例如: 深入调研 2026 具身智能人形机器人驱动方案并撰写万字研报)"]
    end

    subgraph DeepResearchCore [GPT-Researcher 分层多智能体认知中枢]
        ChiefEditor["主编调度智能体 (Chief Editor / Planner)<br/>解析宏观意图，动态规划多章节大纲与正交子问题"]
        
        subgraph SubAgentSwarm [并发研究子智能体集群 (Parallel Research Swarm)]
            Agent1["子智能体 1: 谐波减速器 vs 行星减速器方案"]
            Agent2["子智能体 2: 准直驱电机热力学与扭矩密度"]
            Agent3["子智能体 3: 灵巧手腱绳驱动与传感器集成"]
        end

        subgraph SearchAndScrapeGrid [多源搜索与页面无头抓取网格]
            SearchAPIs["搜索引擎聚合<br/>(Tavily / SearXNG / DuckDuckGo / arXiv)"]
            DocScraper["无头页面解析与正文清洗<br/>(剔除广告/脚本，保留语义段落与元数据)"]
        end

        subgraph FactVerificationLayer [事实交叉校验与引用管理]
            ContextStore["结构化证据池 (Verified Context Buffer)<br/>按子话题归类，保留精确 URL 来源锚点"]
            FactChecker["一致性审查器 (Fact-Checker / Critic)<br/>过滤冲突数据、低信噪比噪声与重复段落"]
        end

        subgraph SynthesisEngine [长文编排与渲染引擎]
            DraftWriter["分章节长文起草器 (Section Writer)<br/>严格基于证据池段落注入确定性引用"]
            ReportPublisher["全局研报统筹器 (Publisher)<br/>生成引言、对比表格、结论展望并导出 Markdown/PDF"]
        end
    end

    UserInput ==> ChiefEditor
    ChiefEditor -->|拆解出子问题 DAG| SubAgentSwarm
    SubAgentSwarm --> SearchAPIs
    SearchAPIs --> DocScraper
    DocScraper ==> ContextStore
    ContextStore --> FactChecker
    FactChecker ==> DraftWriter
    DraftWriter --> ReportPublisher
```

1. **突破传统 RAG 局限的分层深度研究范式（Hierarchical Deep Research）**：
   - 传统检索增强生成（RAG）大多依赖单次语义检索匹配 Top-K 片段，在面对跨维度、需要综合对比论证的宏观研究题目（如“全球固态电池量产工艺瓶颈与产业格局对比”）时，极易出现以偏概全、缺乏全局视角的断层。
   - GPT-Researcher 首创主编-子研究员分层体系，先生成结构化研究大纲（Research Outline），再将每个章节拆解为正交的探索子任务，实现多角度无死角的穷尽式调研。
2. **多源异构搜索网格与高质量无头正文清洗**：
   - 内置灵活可配置的搜索后端，不仅支持企业级 AI 搜索专线（Tavily、Perplexity），还支持开源私有化元搜索引擎（SearXNG）、学术知识库（arXiv）以及通用搜索引擎。
   - 具备高效的异步网页正文抽取流水线，自动剥离导航栏、页脚、广告脚本等杂质，仅保留高信息密度的语义段落，并精准记录发布时间与原文 URL 来源。
3. **确定性证据溯源与学术级引用保真（Deterministic Sourcing）**：
   - 针对大模型在长篇生成中极易产生的“虚构引用”与“张冠李戴”幻觉，GPT-Researcher 采用上下文证据绑定机制。在最终研报中的每一个数据点、关键结论均由模型自动附带指向原始出处的超链接与脚注，杜绝编造不存在的来源。
4. **长篇万字研报的模块化自适应合成**：
   - 摒弃了一次性全量生成的做法，采用分章节迭代递进撰写与全局聚合的流水线。支持自适应生成多维度 Markdown 对比表格、技术路线时序图、SWOT 分析等深度内容，单次生成研报长度可稳定突破万字，达到可以直接用于商业决策或学术开题的生产级水准。

---

## 🛠️ 快速上手与配置

### 1. 环境安装与核心依赖

```bash
# 基于 uv 极速安装 GPT-Researcher
uv pip install gpt-researcher

# 或使用标准 pip 安装
pip install gpt-researcher
```

### 2. 环境变量配置 (`.env`)

```ini
# 配置主思考与长文撰写模型（支持 OpenAI, DeepSeek, Anthropic）
FAST_LLM="openai:gpt-4o-mini"
SMART_LLM="deepseek:deepseek-chat"

# 配置主力搜索引擎（推荐 Tavily 或私有部署的 SearXNG）
RETRIEVER="tavily"
TAVILY_API_KEY="tvly-xxxxxxxxxxxxxxxxxxxxxxxx"

# 可选：配置深度爬虫并发与超时
SCRAPER_TIMEOUT=15
MAX_SEARCH_RESULTS_PER_QUERY=5
TOTAL_WORDS=3000
```

### 3. Python 核心调用与异步研报生成示例

```python
import asyncio
from gpt_researcher import GPTResearcher

async def generate_deep_research():
    query = "2026年全球开源大模型推理引擎技术对比：重点分析 vLLM, SGLang, FlashMLA 的架构演进与显存瓶颈破局"
    report_type = "research_report"  # 可选: research_report, detailed_report, outline_report
    
    # 1. 初始化研究员智能体
    researcher = GPTResearcher(query=query, report_type=report_type)
    
    # 2. 启动自主多步深度调研（自动拆解子问题 -> 并行爬取 -> 事实清洗）
    print("🚀 启动深度调研中，正在并发抓取全网事实...")
    research_context = await researcher.conduct_research()
    print(f"✅ 证据池构建完毕，共提取相关事实段落 {len(research_context)} 条。")
    
    # 3. 合成长篇结构化研报（注入严格引用与比对表格）
    print("📝 正在合成高保真结构化研报...")
    report = await researcher.write_report()
    
    # 4. 持久化保存为 Markdown 文件
    output_filename = "2026_Inference_Engines_Research_Report.md"
    with open(output_filename, "w", encoding="utf-8") as f:
        f.write(report)
    print(f"🎉 研报已成功归档至 {output_filename}")

if __name__ == "__main__":
    asyncio.run(generate_deep_research())
```

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **行业/竞品/技术前瞻调研**：企业战略部门或架构师团队针对新兴开源项目、技术路线（如 MLA 架构、Agent 协议）的自动化全网技术情报收集；
  - **学术文献综述与知识库冷启动**：快速扫描 arXiv 论文或专业技术博客，自动输出结构化的 Literature Review；
  - **企业私有化 Deep Research 智能体中枢**：替代昂贵且封闭的商业版 Deep Research 工具，结合企业内部 SearXNG 与本地私有 LLM 搭建完全合规的内网研究智囊。
- **潜在不足与工程注意点**：
  - **多源搜索 API 额度与限流**：高并发并行调研可能在数分钟内发起数十次搜索请求，生产落地中需做好 Token 桶限流与备用搜索引擎降级机制；
  - **网页反爬与动态渲染耗时**：部分目标站点存在 Cloudflare 阻断或重度客户端 JS 渲染，需结合轻量级无头浏览器（如 Crawl4AI / Playwright）实现弹性降级。
- **个人评估结论**：**建议全面引入生产（Adopted）**。作为开源 Deep Research 领域的黄金标准，其分层规划与严格溯源机制是大模型应用从“玩具级问答”迈向“专业级生产力中枢”的标杆实现。
