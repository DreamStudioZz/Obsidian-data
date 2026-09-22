---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/browser-use/browser-use
stars: "32k+"
license: MIT
date_added: 2026-09-22
---

# 📦 Browser-Use

> **一句话简介**：连接 LLM 与真实 Web 世界的革命性开源 Agent 框架，通过“视觉感知 + 精简交互 DOM 树”融合定位，让大模型能像真人一样在浏览器中自主点击、输入、滚动与完成端到端多步交互任务。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: browser-use/browser-use](https://github.com/browser-use/browser-use) |
| **底层依赖** | Playwright, LangChain / 任意多模态大模型 (GPT-4o, Claude 3.5 Sonnet 等) |
| **核心机制** | 交互元素带序号 Bounding Box 标记、精简可交互 DOM 投影、Controller 自定义动作注入 |
| **应用场景** | 网页自主数据采集、跨系统表单自动化、端到端 E2E 自动化测试、跨站业务自动化 |
| **关联实践** | [[多模态网页自动化：基于 Browser-Use 的视觉驱动 Agent 设计与工程避坑]], [[smolagents]], [[Pi-Agent]] |

---

## 🚀 核心架构与创新亮点

传统网页自动化（如 Selenium / Puppeteer 脚本）最大的痛点是**选择器极度脆弱**，任何前端样式微调或 class 混淆都会导致脚本全面崩溃；而直接把整页 HTML 丢给 LLM 又会面临成千上万 Token 的严重膨胀与巨大延迟。

**Browser-Use 的破局思路**：

```mermaid
flowchart TD
    Task[用户自然语言任务] --> AgentLoop[Browser Agent 思考循环]
    subgraph Perception [双重感知提取 (Perception Engine)]
        Browser[Playwright 真实浏览器] --> Screenshot[页面实时截图]
        Browser --> FilteredDOM[精简可交互 DOM 树]
        Screenshot & FilteredDOM --> Grounding[带索引的交互标注: 按钮/输入框/链接标注序号]
    end
    Grounding --> VLLM[多模态大模型: 视觉 + 语义决策]
    VLLM --> ActionDecision[决策动作: click(14) / type(5, 'text') / scroll]
    ActionDecision --> Controller[Controller 执行驱动]
    Controller --> Browser
    ActionDecision --> Evaluator{任务是否完成?}
    Evaluator -->|否| Perception
    Evaluator -->|是| Done[返回最终提取数据或报告]
```

1. **精简 DOM 投影与 Token 极致压缩**：
   - 绝不将整个页面的庞杂 HTML 发送给大模型。它通过前端 JS 脚本过滤出当前视口内真正“可交互”（点击、输入、滚动）的元素，按层级压缩为极度紧凑的树形文本，Token 消耗缩减 90% 以上。
2. **视觉与坐标对齐（Visual Grounding & Set-of-Marks）**：
   - 在截图上对每个可操作元素动态打上醒目的彩色数字标签框（如 `[12] 提交按钮`, `[15] 搜索输入框`），使多模态大模型能极其直观、精准地指出要操作的编号。
3. **强大的 Controller 动作扩展机制**：
   - 支持开发者通过 `@controller.action` 装饰器，向 Agent 无缝注入自定义 Python 业务工具（如将网页抓取到的数据直接写入企业数据库、发送企业微信告警等）。
4. **持久化会话与反爬感知**：
   - 支持加载本地 Chrome 个人 Profile、保存 Cookies、伪装用户设备指纹，从容应对登录态与验证码挑战。

---

## 🛠️ 快速上手示例

### 1. 安装
```bash
pip install browser-use playwright
playwright install
```

### 2. 编写一个端到端自主浏览智能体
```python
import asyncio
from browser_use import Agent, Controller
from langchain_openai import ChatOpenAI
from pydantic import BaseModel

# 1. 定义期望结构化输出的模型
class SearchSummary(BaseModel):
    query: str
    top_result_title: str
    summary: str

controller = Controller(output_model=SearchSummary)

# 2. 注册自定义执行动作（扩展 Agent 能力）
@controller.action("将抓取到的核心摘要持久化保存")
def save_summary_locally(content: str):
    with open("saved_web_summary.txt", "w", encoding="utf-8") as f:
        f.write(content)
    return "本地持久化成功！"

async def main():
    agent = Agent(
        task="打开 GitHub Trending 页面，找到排在第一名的 AI 开源项目，提取其名称、Star 数和一句话简介并保存",
        llm=ChatOpenAI(model="gpt-4o"),
        controller=controller,
        use_vision=True # 开启视觉多模态精准定位
    )
    
    history = await agent.run(max_steps=10)
    print("最终执行结果：", history.final_result())

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 没有开放 API 的外部商业网站数据自动化流转与填报；
  - 现代化 SPA 单页应用的端到端（E2E）智能探索式自动化测试；
  - 辅助人工完成多步骤登录认证、报表导出等繁琐流程。
- **潜在不足 / 局限性**：
  - 依赖多模态 LLM（推荐 GPT-4o 或 Claude 3.5 Sonnet），纯文本纯小模型在此类复杂视觉定位任务上容易失焦；长时间长步数运行会消耗较多多模态 Token。
- **个人评估结论**：**建议引入（status/adopted）**。是目前开源 GUI / Web Agent 领域架构最扎实、实战落地体验最好的标杆框架。
