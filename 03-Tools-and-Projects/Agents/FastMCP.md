---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/modelcontextprotocol/python-sdk
stars: "10k+"
license: MIT
date_added: 2026-09-21
---

# 📦 FastMCP

> **一句话简介**：Anthropic 官方 MCP Python SDK 核心组件，让开发者像写 FastAPI 一样用最少代码构建标准化 Model Context Protocol 服务。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **项目地址** | [GitHub: modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) |
| **核心协议** | MCP (Model Context Protocol) |
| **前身背景** | 原由 Prefect 团队创建，已被官方吸纳合并为标准库核心实现 |
| **关联概念** | [[000-AI-Index]], [[2026-09-21-AI-Digest]] |

---

## 🚀 为什么说 MCP 是智能体工具交互的新标准？

过去每个 Agent 框架（LangChain, AutoGen, CrewAI, 各 IDE）都有一套自己的 Tool 定义规范，生态严重割裂。
**Model Context Protocol (MCP)** 类似于语言服务器协议（LSP, Language Server Protocol）：
- 一次编写，可以在任何支持 MCP 的客户端（Cursor, Claude Desktop, Antigravity, 各种开源 Agent）中无缝挂载。

**FastMCP 的优势**：
1. **装饰器语法驱动**：无需手写冗长的 JSON Schema，自动从 Python 类型注解和 Docstring 生成模型可理解的工具描述。
2. **支持三种核心原生实体**：
   - **Tools（工具）**：可供模型调用的函数操作（如执行代码、查库、发邮件）。
   - **Resources（资源）**：像只读文件或数据上下文一样被读取。
   - **Prompts（预设提示词）**：可复用的提示词模板。

---

## 🛠️ 快速上手示例

### 1. 编写一个最简单的 FastMCP 服务
```python
from mcp.server.fastmcp import FastMCP

# 初始化服务实例
mcp = FastMCP("DeveloperToolbox")

@mcp.tool()
def calculate_growth_rate(old_val: float, new_val: float) -> float:
    """计算两期数值之间的增长率百分比。
    
    Args:
        old_val: 基期数值
        new_val: 现期数值
    """
    if old_val == 0:
        return 0.0
    return ((new_val - old_val) / old_val) * 100.0

@mcp.resource("db://schema/users")
def get_user_schema() -> str:
    """暴露数据库用户信息 Schema 作为上下文资源"""
    return "TABLE users (id INT PRIMARY KEY, name VARCHAR, role VARCHAR);"

if __name__ == "__main__":
    mcp.run()
```

### 2. 在 IDE / 客户端配置挂载
在客户端的 `mcp_servers.json` 中添加配置即可直接生效：
```json
{
  "mcpServers": {
    "dev-tools": {
      "command": "python",
      "args": ["d:/workspace/my_mcp_server.py"]
    }
  }
}
```

---

## 💡 工程落地建议
- **Docstring 就是 Prompt**：大模型依赖函数注释来推断“何时该调用这个工具”。务必在函数 Docstring 中清晰描述工具的使用边界、输入单位和返回结构。
- **安全沙箱**：暴露系统级命令或文件修改的 MCP 工具，建议在生产环境中置于隔离容器或指定安全根目录下。
