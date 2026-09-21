---
tags:
  - guide
  - obsidian-plugin
  - dataview
date_added: 2026-09-21
---

# 📘 Dataview 插件手把手使用指南

> **很多新手安装 Dataview 后的第一个疑问**：“我已经安装并开启了插件，为什么界面上什么都没变？”
> **原因**：Dataview **不是一个弹窗式界面的插件**，而是一个**“在笔记中写类似 SQL 语句的动态查询引擎”**！
> 只有当你在笔记中写下特定查询块，并在**阅读视图（Reading View）**或**实时预览（Live Preview）**模式下，它才会动态渲染出表格。

---

## 🔍 第一步：确认你的 Obsidian 视图模式

1. 在 Obsidian 右上角或按快捷键 `Ctrl + E`，切换视图模式。
2. 确保不是纯**源码模式（Source mode）**，而是 **实时预览（Live Preview）** 或 **阅读视图（Reading View）**。
3. 现在，你直接打开 `[[000-AI-Index]]`，就可以看到它已经变成了 3 张自动汇总的动态数据表！

---

## 💡 第二步：理解 Dataview 是怎么工作的？

我们在为你创建工具卡片（如 `vLLM`、`FastMCP`、`Graphify`）时，每个文件开头都有这样一段 YAML Frontmatter：

```yaml
---
tags:
  - ai-tool
category: LLM-Inference
github: https://github.com/vllm-project/vllm
date_added: 2026-09-21
---
```

**对 Dataview 来说，这一段就是数据库的“数据列”！**
- `category` 对应分类
- `github` 对应链接
- `date_added` 对应添加日期
- `tags` 对应标签

---

## 🚀 第三步：常用四大查询语法与实战范例

Dataview 支持四种核心查询：`TABLE`（表格）、`LIST`（列表）、`TASK`（待办任务）、`CALENDAR`（日历）。

### 1. 表格查询 (TABLE) —— 最常用
```text
```dataview
TABLE category AS "类别", github AS "仓库", date_added AS "收录日期"
FROM "03-Tools-and-Projects"
SORT date_added DESC
```
```
- **解释**：
  - `TABLE ...`：列出哪些字段，并用 `AS` 给表头起中文名。
  - `FROM "文件夹名"`：只扫描指定文件夹（也可以写 `FROM #ai-tool` 按标签查）。
  - `SORT ... DESC`：按日期从新到旧倒序排列。

---

### 2. 列表查询 (LIST) —— 快速列出清单
比如想找所有难度为“进阶”的工程实践笔记：
```text
```dataview
LIST "难度: " + difficulty
FROM "04-Engineering-Practices"
WHERE difficulty = "进阶"
```
```

---

### 3. 按标签多条件过滤 (WHERE)
查找包含 `#ai-tool` 且分类是 `Agent` 的项目：
```text
```dataview
TABLE github AS "地址"
FROM #ai-tool
WHERE category = "Agent"
```
```

---

## 🛠️ Dataview 常用字段参考

除了你自己定义的 YAML 字段（如 `category`, `domain`），Dataview 自带了以下隐式元数据：
- `file.name`：笔记文件名
- `file.folder`：所在文件夹
- `file.tags`：包含的标签
- `file.ctime`：创建时间
- `file.mtime`：最后修改时间
- `file.size`：文件大小

---

## 🎯 立即去体验一下！
点击打开 👉 `[[000-AI-Index]]`，体验全自动知识库看板带来的快感！
后续只要 AI 助手为你添加了新卡片，看板都会**全自动更新**，你完全不需要手动维护列表！
