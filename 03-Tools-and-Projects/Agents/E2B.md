---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/e2b-dev/code-interpreter
stars: "7.2k+"
license: Apache-2.0
date_added: 2026-10-04
---

# 📦 E2B

> **一句话简介**：专为 AI 智能体与大模型应用打造的安全云端代码沙箱运行时（Code Interpreter），基于 AWS Firecracker 轻量级 MicroVM 架构，提供毫秒级冷启动、高密度多租户硬件级强隔离与完整富媒体产物交互能力。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: e2b-dev/code-interpreter](https://github.com/e2b-dev/code-interpreter) |
| **核心特点** | Firecracker MicroVM 强隔离、亚秒级冷启动（<150ms）、原生支持 Python/JS/Bash、图表/数据表产物自动捕获、会话状态持久化 |
| **技术栈** | Go / Rust / Python / TypeScript / AWS Firecracker |
| **关联实践** | [[Agent 安全沙箱隔离：基于 E2B MicroVM 的代码解释器与无感运行时环境实战]], [[自主软件工程智能体：基于 OpenHands 的沙箱隔离与微智能体架构实践]], [[代码驱动智能体：基于 CodeAgent 的复杂多步任务执行范式]] |

---

## 🚀 核心特性与技术亮点

1. **Firecracker MicroVM 硬件级内核隔离**：
   - 摒弃了传统容器（Docker）共享 Linux 内核带来的安全隐患与逃逸风险，每个沙箱均运行在由 KVM 驱动的独立轻量级虚拟机中，彻底阻断宿主机越权与内网横向渗透；
   - 相比传统虚拟机拥有极度精简的虚拟硬件设备模型与极小的内存开销，实现单台物理服务器承载上千个高密并发沙箱。
2. **毫秒级极速冷启动（Sub-second Spin-up）**：
   - 借助预置虚拟机快照与内存分页即时载入技术，沙箱实例从创建到就绪仅需 100~200ms，为交互式 Agent（如数据分析、代码补全与自主修复）提供近乎零等待的执行响应。
3. **原生富媒体产物与图表解析（Rich Output Artifacts）**：
   - 解释器不仅能流式捕获 `stdout` 与 `stderr`，还能原生拦截并解码 Jupyter 内核中由 Matplotlib、Seaborn、Plotly 生成的 PNG/SVG 图像及 Pandas DataFrame 数据，自动转为高保真 Artifacts 推送给宿主应用。
4. **多语言混合沙箱与长效会话持久化**：
   - 开箱即用支持 Python 3.x（预装科学计算全家桶）、Node.js、Bash 与通用 Linux 命令行环境；
   - 支持跨交互维持沙箱环境（Filesystem / Variables / Process State），Agent 可分步执行复杂的数据处理流水线而无需重复初始化。

---

## 🛠️ 快速上手与配置

### 1. 安装 Python SDK 与设置 API Key
```bash
pip install e2b-code-interpreter
export E2B_API_KEY="e2b_***"
```

### 2. 运行 Python 代码并提取图表产物
```python
from e2b_code_interpreter import CodeInterpreter

# 初始化 MicroVM 沙箱实例
with CodeInterpreter() as sandbox:
    print("沙箱已就绪，正在执行数据分析与绘图...")
    
    code = """
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(8, 4))
plt.plot(x, y, label='sin(x)', color='royalblue')
plt.title('Agent Generated Sine Wave')
plt.legend()
plt.grid(True)
plt.show()
    """
    
    # 执行代码并获取富媒体结果
    execution = sandbox.notebook.exec_cell(code)
    
    # 打印标准输出与图表
    for log in execution.logs.stdout:
        print(f"[STDOUT] {log}")
        
    for result in execution.results:
        if result.png:
            # 获取 Base64 编码的图表文件并落地
            with open("output_chart.png", "wb") as f:
                import base64
                f.write(base64.b64decode(result.png))
            print("已成功捕获图表产物: output_chart.png")
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **自主数据分析 Agent（Data Analysis Copilot）**：需要频繁写 Python 脚本拉取数据、进行回归分析并生成可视化图表的场景；
  - **代码生成与调试助手（Coding Agent）**：如 OpenHands、Cline、SWE-agent，需在受控隔离环境中运行测试用例、编译项目与排查报错；
  - **多租户 SaaS AI 应用**：严格禁止不同租户之间数据窥探与文件系统篡改的生产级合规场景。
- **潜在不足 / 局限性**：
  - 云端托管版依赖公网通信延迟，且需要按运行时间付费；自建私有化 E2B 集群需要宿主机开启 KVM 虚拟化硬件支持（如裸金属服务器或支持嵌套虚拟化的云主机实例）。
- **个人评估结论**：**建议全面采用（status/adopted）**。是当前构建任何具备“代码执行能力”的智能体必不可少的安全基础设施底座。
