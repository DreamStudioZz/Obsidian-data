---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/princeton-nlp/SWE-agent
stars: "15.5k+"
license: MIT
date_added: 2026-10-28
---

# 📦 SWE-agent

> **一句话简介**：普林斯顿大学 NLP 实验室重磅开源的自主软件工程智能体标杆平台，首创面向大语言模型的专用计算机交互界面（ACI, Agent-Computer Interface），在真实 GitHub 复杂代码缺陷自动定位与修复基准 SWE-bench 上创下领先表现。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent) |
| **研发机构** | 普林斯顿大学 NLP 实验室 (Princeton NLP) |
| **技术栈** | Python, Docker, Jinja2, Bash / Tree-sitter |
| **核心评测** | SWE-bench / SWE-bench Lite / SWE-bench Verified |
| **关联实践** | [[自主软件工程演进：基于 SWE-agent 的专用 ACI 交互界面构建与代码缺陷自愈实战]], [[OpenHands]], [[Aider]], [[Cline]] |

---

## 🚀 核心特性与架构亮点

1. **首创 Agent-Computer Interface (ACI) 专用交互界面**：
   - 彻底打破直接将原生 Linux Bash 裸露给大模型的落后模式。人类终端命令（如 `cat`、`find`、`git diff`）动辄输出成千上万行文本，会瞬间击穿大模型上下文窗口并引发“迷失在中间（Lost in the Middle）”幻觉；
   - ACI 为大模型量身定制了轻量、结构化、带窗口滚动与自愈反馈的原子工具链（如受限窗口分页浏览 `open_file`、上下文匹配编辑 `edit_file`、语法解析校验器与目录深度剪枝搜索 `search_dir`）。
2. **实时语法检查与编辑自愈拦截（Linter-in-the-Loop）**：
   - 在执行代码覆写或局部行替换时，ACI 会自动调用底层语法分析器（如 Python Flake8/Tree-sitter）进行语法树校验；
   - 一旦检测到缩进错误、括号不匹配或未定义变量，命令直接就地拦截并在标准输出中回显精准语法错误，强制智能体在同一个推理步内即时重写，杜绝污染工作区。
3. **安全隔离的 Docker 沙箱与执行环境复现**：
   - 原生支持为每一个 GitHub Issue 动态构建或挂载对应的 Conda/Python 沙箱容器；
   - 包含完整的测试用例自动化运行引擎（Pytest/Unittest），智能体能够自主编写最小可复现脚本 `reproduce_issue.py`，先在沙箱中精准复现 Bug，修复后再次运行验证直到测试用例全绿通过。
4. **高可扩展的声明式配置体系（Config-Driven Hooks）**：
   - 智能体的系统提示词、可用 ACI 命令集、环境交互超时时间以及重试策略均通过清晰的 YAML 声明，便于团队针对 Java、Go、C++ 或专有企业代码库无缝迁移扩展。

---

## 🛠️ 快速上手与使用示例

### 1. 运行环境准备与安装

```bash
# 克隆仓库并安装核心依赖
git clone https://github.com/princeton-nlp/SWE-agent.git
cd SWE-agent
pip install -e .

# 确保宿主机 Docker 守护进程正常启动（用于启动隔离测试环境）
docker pull sweagent/swe-agent:latest
```

### 2. 通过 CLI 命令行自动修复单个 GitHub Issue

```bash
# 指定模型与 GitHub 目标仓库 Issue URL，启动全自动排查与修复流水线
python run.py \
  --model_name "claude-3-5-sonnet-20241022" \
  --data_path "https://github.com/marshmallow-code/marshmallow/issues/1867" \
  --config_file "config/default.yaml" \
  --per_instance_cost_limit 2.00
```

### 3. 基于 Python SDK 编程式调用与自愈验证

```python
import os
from sweagent.agent.agents import Agent
from sweagent.environment.swe_env import SWEEnv

# 1. 初始化容器化隔离环境
env_config = {
    "repo_path": "/workspace/flask",
    "base_commit": "e6a1b2c",
    "conda_env": "flask-dev"
}
env = SWEEnv(args=env_config)
env.reset()

# 2. 装配 ACI 强化智能体
agent = Agent.from_config("config/default_aci.yaml")

# 3. 驱动自主定位与修复循环
problem_statement = """
Issue #4521: Blueprint url_prefix trailing slash causes redirect loop in route matching.
Steps to reproduce: Register blueprint with url_prefix='/api/' and route '/status'.
Expected: 200 OK without 308 redirect.
"""

observation, info = env.reset()
trajectory = []

for step in range(30):
    action = agent.forward(observation, trajectory)
    observation, reward, done, info = env.step(action)
    trajectory.append({"action": action, "observation": observation})
    if done:
        print("🎉 Issue 修复完成，生成 Patch 并通过单测验证！")
        break

# 4. 导出 Git Patch
patch = env.get_git_patch()
print("Generated Git Diff:\n", patch)
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **严肃代码缺陷定位与修复（Bug Triaging & Fixing）**：接入 GitLab / GitHub Webhook，当用户创建 Issue 且附带错误日志时，自动生成复现测试用例并提 PR 修复；
  - **自动化代码重构与 API 弃用迁移**：在框架大版本升级时，自主遍历微服务代码库完成接口迁移与测试适配；
  - **安全漏洞紧急打补丁**：针对 CVE 漏洞描述，在隔离沙箱中快速验证攻击载荷并产出防御性补丁。
- **与同类工具对比**：
  - **相比于 OpenHands / Devin**：SWE-agent 更加专注和精细于 **ACI（命令交互界面）设计对模型解决率的提升**，在学术界与工业评测中具备极强的可解释性与学术严谨性；
  - **相比于 Aider**：Aider 侧重于开发者终端结对编程（Architect-Editor 双模式），而 SWE-agent 侧重于**全自主（Autonomous）端到端解决复杂 Issue** 并自建复现脚本验证闭环。
- **综合评估结论**：自主软件工程与 Coding Agent 底层接口设计范式开山之作，强烈推荐采纳（Adopted）。
