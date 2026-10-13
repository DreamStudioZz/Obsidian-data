---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/unclecode/crawl4ai
stars: "35k+"
license: Apache-2.0
date_added: 2026-10-13
---

# 📦 Crawl4AI

> **一句话简介**：专为大模型（LLMs）与 RAG 系统设计的高性能、全异步开源无头网页抓取与结构化清洗引擎，将复杂的动态网页以闪电般速度转化为纯净的 Markdown、精炼的分块及严格合规的 JSON 数据，彻底摆脱昂贵的商业爬虫云服务限制。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: unclecode/crawl4ai](https://github.com/unclecode/crawl4ai) |
| **研发作者** | unclecode (Hasan Abid) |
| **技术栈** | Python 3.10+, Playwright, Asyncio, Pydantic, Litellm |
| **许可证** | Apache-2.0 |
| **关联生态** | [[Firecrawl]], [[Docling]], [[Kotaemon]], [[MinerU]], [[Milvus]], [[Qdrant]] |
| **关联实践** | [[大模型友好型高效网页抽取：基于 Crawl4AI 的异步无头爬取与自适应分块实战]] |

---

## 🚀 核心架构与创新亮点

在构建企业级 RAG 知识库或训练数据摄入管道时，直接使用传统爬虫（如 Scrapy、BeautifulSoup）通常只能拿到原始 HTML。未清洗的 HTML 中充斥着样式表、脚本、广告弹窗、Cookie 条幅以及复杂的 DOM 嵌套，**90% 以上的内容均属 Token 噪声**。

商业 SaaS 爬虫（如 Firecrawl 云端 API）虽然能够生成 Markdown，但按量计费昂贵、并发数受限且存在数据出境安全隐患。

**Crawl4AI 的架构突破**：采用高度解耦的**异步管道（Pipeline）与策略模式（Strategy Pattern）**，在本地端侧实现毫秒级动态渲染、无监督 DOM 降噪与双轨数据抽取。

```mermaid
flowchart TD
    TargetURL[目标网页 URL / 批量站点] --> AsyncEngine[AsyncWebCrawler 核心调度]
    
    subgraph BrowserLayer [无头浏览器运行时 (BrowserConfig)]
        Playwright[Playwright Async 浏览器实例]
        Stealth[反爬指纹对抗 & User-Agent 伪装]
        SessionManager[Cookie 状态机 & 动态 JS 执行]
        Playwright --- Stealth --- SessionManager
    end
    
    AsyncEngine --> BrowserLayer
    BrowserLayer --> RawDOM[渲染完成的完整 DOM 树]
    
    subgraph CleaningLayer [内容过滤与清洗引擎 (Content Filters)]
        Pruning["PruningContentFilter<br>(基于文本/链接密度无监督修剪导航栏与条幅)"]
        BM25Filter["BM25ContentFilter<br>(基于查询关键词召回最核心语义区域)"]
        RawDOM --> Pruning & BM25Filter
    end
    
    CleaningLayer --> CleanMD[纯净版 Markdown (fit_markdown) 格式]
    
    subgraph ExtractionModes [双轨结构化提取策略 (Strategies)]
        Deterministic["【确定性免 LLM 策略】<br>JsonCssExtractionStrategy / XPath<br>毫秒级运行、零 Token 消耗、100% 格式稳定"]
        Semantic["【认知型大模型语义策略】<br>LLMExtractionStrategy<br>结合 Pydantic Schema 自动理解长尾非结构化文本"]
    end
    
    CleanMD --> ExtractionModes
    Deterministic & Semantic --> ResultBundle["CrawlResult 成果包<br>(Markdown, 结构化 JSON, 多媒体清单, 元数据)"]
    ResultBundle --> VectorStore[下游: 向量数据库 / RAG 管道 / LLM 上下文]
```

### 1. 深度解耦的双配置架构 (v0.4+)
- **`BrowserConfig`**：控制浏览器底层生命周期（无头模式、代理服务器轮换、反指纹探测、禁用媒体/CSS 以极致加速）；
- **`CrawlerRunConfig`**：控制页面抓取行为（页面滚动等待策略、动态 JavaScript 钩子、缓存机制、过滤策略及抽取策略），实现高度灵活的组装编排。

### 2. 独创的智能降噪过滤器 (Fit Markdown & Pruning)
- **`PruningContentFilter`**：无需预设任何关键词，通过算法自动分析 DOM 树中的文本密度（Text Density）与链接密度（Link Density），智能识别并切除侧边栏、页眉、页脚及隐私弹窗；
- **`BM25ContentFilter`**：基于信息检索的经典 BM25 算法，针对用户给定的主题 Query 自动给 DOM 块打分，精准保留与当前主题强相关的内容片段。

### 3. 双轨提取模式：确定性规则 vs 认知型 LLM
- **零成本结构化提取（LLM-Free）**：通过 `JsonCssExtractionStrategy` 或 `JsonXPathExtractionStrategy`，仅需配置简单规则即可批量将复杂列表页转换为嵌套 JSON，无需耗费一分钱的 API Token；
- **智能语义理解抽取（LLM-Based）**：通过 `LLMExtractionStrategy`，将过滤后的文本输入大模型，依据 Pydantic 模型自动抽取结构化数据，支持 OpenAI、Claude 及本地 Ollama。

### 4. 极致的并发性能与内置本地缓存
- 全异步（`asyncio`）多标签页并行采集，相比传统同步爬虫吞吐量提升数倍；
- 内置本地磁盘/内存缓存机制，相同 URL 多次测试与抓取无需重复发起网络握手，极大降低目标站点负载与调试成本。

---

## 🛠️ 快速上手与配置

### 1. 安装与浏览器驱动初始化
```bash
pip install -U crawl4ai
crawl4ai-setup # 自动安装并校验 Playwright 浏览器内核环境
```

### 2. 基础异步抓取与纯净 Markdown 提取
```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
from crawl4ai.content_filter_strategy import PruningContentFilter
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator

async def main():
    # 1. 配置浏览器参数 (启用反指纹与极速模式)
    browser_cfg = BrowserConfig(
        headless=True,
        verbose=False,
        text_mode=True # 禁用图片加载，极大提速
    )
    
    # 2. 配置过滤与 Markdown 生成器
    md_generator = DefaultMarkdownGenerator(
        content_filter=PruningContentFilter(threshold=0.48, min_word_threshold=10)
    )
    
    run_cfg = CrawlerRunConfig(
        markdown_generator=md_generator,
        word_count_threshold=15
    )

    # 3. 异步抓取并打印成果
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(
            url="https://github.com/trending",
            config=run_cfg
        )
        if result.success:
            print("--- 原始抓取字符数 ---:", len(result.cleaned_html))
            print("--- 降噪后 Fit Markdown 字符数 ---:", len(result.markdown.fit_markdown))
            print(result.markdown.fit_markdown[:500])

if __name__ == "__main__":
    asyncio.run(main())
```

### 3. 生产级零 Token 结构化抽取（基于 CSS 选择器）
```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, JsonCssExtractionStrategy

schema = {
    "name": "GitHub Trending Repos",
    "baseSelector": "article.Box-row",
    "fields": [
        {"name": "repo_name", "selector": "h2 a", "type": "text"},
        {"name": "repo_url", "selector": "h2 a", "type": "attribute", "attribute": "href"},
        {"name": "description", "selector": "p", "type": "text"},
        {"name": "stars", "selector": "a[href*='stargazers']", "type": "text"}
    ]
}

async def extract_trending():
    run_cfg = CrawlerRunConfig(
        extraction_strategy=JsonCssExtractionStrategy(schema=schema)
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://github.com/trending", config=run_cfg)
        if result.success:
            data = json.loads(result.extracted_content)
            print("成功提取项目数:", len(data))
            print(json.dumps(data[:2], indent=2, ensure_ascii=False))

if __name__ == "__main__":
    asyncio.run(extract_trending())
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **海量企业级 RAG 知识库构建**：将外部行业博客、官方文档库、新闻资讯以低成本、极高信噪比批量摄入；
  - **动态 SPA 复杂站点的数据清洗**：从容应对 Vue、React、Next.js 深度客户端渲染与懒加载长滚动页面；
  - **私有化安全内网部署**：完全本地自闭环运行，零外部 API 依赖，满足严苛的数据安全与合规审计要求。
- **与 Firecrawl 对比**：
  - **Firecrawl**：商业化更成熟，提供易用的 REST API 与托管服务，但依赖云端额度且成本高昂；
  - **Crawl4AI**：100% 开源免费、本地优先、支持深度定制中间件过滤规则，高并发场景下的吞吐与成本表现优异。
- **个人评估结论**：**建议引入（status/adopted）**。是目前开源 AI 生态中针对 LLM 场景打磨最极致、性能最强悍的网页抽取工具底座。
