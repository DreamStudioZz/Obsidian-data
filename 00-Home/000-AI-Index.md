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
SORT date_added DESC, file.ctime DESC
```

---

## 💡 工程实践与架构设计库 (Engineering Practices)

```dataview
TABLE domain AS "技术领域", difficulty AS "难度", date_added AS "收录日期"
FROM "04-Engineering-Practices"
WHERE file.name != "Template-Practice-Card"
SORT date_added DESC, file.ctime DESC
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
