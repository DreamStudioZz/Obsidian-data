---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/mendableai/firecrawl
stars: "32k+"
license: AGPL-3.0
date_added: 2026-09-24
---

# 📦 Firecrawl

> **一句话简介**：专为大模型与 RAG 打造的企业级开源网页数据抽取引擎，可将任意复杂网页（含反爬、JS 动态渲染、SPA 单页应用）一键转化为整洁、无噪的 LLM 友好 Markdown 与结构化 JSON。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: mendableai/firecrawl](https://github.com/mendableai/firecrawl) |
| **核心特点** | 智能清洗转 Markdown、反爬虫穿透、全站拓扑映射（Map）、Schema 结构化抽取 |
| **技术栈** | TypeScript / Node.js / Playwright / Python SDK |
| **关联实践** | [[企业级 RAG 外部数据源清洗：基于 Firecrawl 的反爬对抗与智能 Markdown 结构化抽取实践]], [[Docling]], [[RAGFlow]] |

---

## 🚀 核心特性与技术亮点

1. **为 LLM 量身定制的纯净 Markdown 输出**：
   - 传统爬虫（如 BeautifulSoup、Scrapy）抓取的原始 HTML 夹杂大量 `<div>`、`<script>`、样式类名、Cookie 弹窗与广告导航，极其浪费 Context Window 并严重干扰语义向量计算。
   - Firecrawl 原生进行智能语义树剪枝，保留标题层级、列表、表格和超链接，直接输出 LLM 可即时消费的标准 Markdown。
2. **动态渲染与自动化反爬穿透**：
   - 内置无头浏览器池（Playwright 驱动），支持动态渲染 Vue/React 单页应用与无限滚动（Infinite Scroll）；
   - 内置代理轮换与反机器人检测规避策略，轻松突破 Cloudflare 等常见防护门槛。
3. **全站递归抓取与子路径映射（Crawl & Map）**：
   - 仅需提供一个顶级域名，即可在数秒内通过 `/map` 端点获取该网站所有的可访问有效 URL 拓扑树，配合 Glob 规则批量抓取子路径。
4. **零 Prompt 声明式结构化抽取（Extract）**：
   - 支持传入 Pydantic 模型或 JSON Schema，Firecrawl 在页面加载清洗后直接调用内置抽取器，输出强类型 JSON 数据，告别多步人工解析。

---

## 🛠️ 快速上手与集成

### 1. Docker 本地一键私有化部署

```bash
git clone https://github.com/mendableai/firecrawl.git
cd firecrawl
docker compose up -d
# 服务就绪后 API 默认监听于 http://localhost:3002
```

### 2. Python SDK 核心调用示例

```python
from firecrawl import FirecrawlApp
from pydantic import BaseModel, Field

# 初始化客户端（私有化或官方云端）
app = FirecrawlApp(api_url="http://localhost:3002")

# 1. 单页面深度抓取转 Markdown
scrape_result = app.scrape_url(
    "https://news.ycombinator.com",
    params={
        "formats": ["markdown"],
        "onlyMainContent": True,     # 仅提取正文主体，自动剥离侧边栏与页脚
        "waitFor": 1000              # 等待动态 JS 渲染完成
    }
)
print("提取的纯净 Markdown 前 300 字符：")
print(scrape_result["markdown"][:300])

# 2. 声明式结构化数据抽取
class ProductSpec(BaseModel):
    title: str = Field(description="产品标题")
    price: float = Field(description="产品标价")
    key_features: list[str] = Field(description="主要卖点列表")

extract_result = app.scrape_url(
    "https://example.com/product/xyz",
    params={
        "formats": ["extract"],
        "extract": {
            "schema": ProductSpec.model_json_schema()
        }
    }
)
print("结构化字段抽取：", extract_result["extract"])
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **动态网页 RAG 知识入库**：需要持续从竞品官网、技术文档库、新闻源同步最新网页知识并灌入向量数据库；
  - **智能体外部 Web 搜索工具**：作为 Agent 的 `WebSearchTool` 底层支撑，为大模型提供高质量上下文，而非脏 HTML。
- **与传统 Scrapy / Jina Reader 对比**：
  - 相比原生 Scrapy，开发成本从数天缩短至数分钟，无需人工编写脆弱的 XPath/CSS 选择器；
  - 相比 Jina Reader，Firecrawl 提供完整的自建私有化 Docker 栈与全站递归爬取能力，数据无需外泄至第三方。
- **综合评估结论**：企业级 Web RAG 数据源管道的不二之选，强烈建议采用（Adopted）。
