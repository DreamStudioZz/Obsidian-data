---
tags:
  - ai-tool
  - open-source
  - generative-ui
  - agent-copilot
  - human-in-the-loop
  - status/adopted
category: Agents
github: "https://github.com/CopilotKit/CopilotKit"
stars: "17k+"
license: "MIT"
date_added: 2026-10-12
---

# 📦 CopilotKit

> **一句话简介**：生产级 Web 应用内置 AI Copilot、生成式 UI (Generative UI) 与 Co-Agent 双向状态同步框架，打通前端 React 组件生态与后端 LangGraph 智能体状态图。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - CopilotKit/CopilotKit](https://github.com/CopilotKit/CopilotKit) |
| **技术栈** | TypeScript, React, Next.js, Python, LangGraph, FastAPI |
| **许可证** | MIT |
| **相关关联** | 关联专题: [[LangGraph]], [[Composio]], [[PydanticAI]], [[FastMCP]] |

---

## 🚀 核心特性与亮点

1. **原生生成式 UI 交互范式（Generative UI & Streaming Components）**：
   - 告别单调纯文本与 Markdown 渲染：允许大模型在推理过程中直接“吐出”并流式渲染活生生的 React 交互组件（如审批表单、动态图表、富交互卡片、带操作按钮的确认卡）；
   - 用户可直接在卡片上点击、修改输入，前端事件即时回传给 Agent，形成自然的人机协同环路（Human-in-the-Loop）。
2. **Co-Agents：与 LangGraph 深度双向状态同步（Bidirectional State Sync）**：
   - 业内首个打通前端状态与后端状态机图（StateGraph）的协议层：前端 React State 与后端的 LangGraph Agent State 保持毫秒级双向响应式同步；
   - 无论前端发生用户点击还是后端 Agent 节点迁移（Node Transition），状态始终对齐，实现多步骤复杂事务的可视化追溯与断点恢复。
3. **前端端侧动作直连（In-App Client Actions & DOM Mutators）**：
   - 支持为 Copilot 注册前端组件级 Actions（如页面跳转、表单自动填充、高亮特定 DOM 元素、触发图表缩放）；
   - 赋予大模型直接操作用户界面的“手”，真正将 AI 从孤立的“问答侧边栏”转变为业务系统的“原生副驾驶”。
4. **即插即用的生产级开箱组件库（Turnkey UI Primitives）**：
   - 提供丰富且高度可定制的 UI 组件：`<CopilotSidebar>`（抽屉侧边栏）、`<CopilotPopup>`（浮动对话球）、`<CopilotChat>`（内嵌聊天区）、`<CopilotTextarea>`（基于上下文补全的智能输入框）。
5. **与同类工具对比**：
   - **相比于 Vercel AI SDK**：Vercel AI SDK 偏向基础流式传输管道与基础 RSC 渲染；CopilotKit 在应用层更进一步，封装了全套开箱即用的 Copilot 交互组件、端侧 Action 钩子以及与 LangGraph 深度绑定的 Co-Agent 状态机同步协议；
   - **相比于传统内嵌 iframe 聊天插件**：CopilotKit 与宿主 React 应用共享同一执行上下文与状态树，具备无缝的上下文感知与双向操作能力。

---

## 🛠️ 快速上手与配置

```bash
# 1. 安装前端 React 核心包
npm install @copilotkit/react-core @copilotkit/react-ui

# 2. 安装后端 Python Co-Agent 桥接包（配合 LangGraph）
pip install copilotkit
```

```tsx
// 前端集成示例：在 React 应用中挂载 Copilot 并注册前端 Action
import { CopilotKit } from "@copilotkit/react-core";
import { CopilotSidebar } from "@copilotkit/react-ui";
import { useCopilotAction, useCopilotReadable } from "@copilotkit/react-core";
import "@copilotkit/react-ui/styles.css";

function Dashboard() {
  const [selectedKpi, setSelectedKpi] = React.useState("GMV");

  // 1. 让 Agent 实时感知当前业务上下文
  useCopilotReadable({
    description: "当前仪表盘选中的核心业务指标",
    value: selectedKpi,
  });

  // 2. 赋予 Agent 直接操作前端界面的能力
  useCopilotAction({
    name: "switchKpiView",
    description: "切换仪表盘图表显示的指标（GMV, DAU, Retention）",
    parameters: [
      { name: "kpi", type: "string", description: "目标指标名称", required: true },
    ],
    handler: async ({ kpi }) => {
      setSelectedKpi(kpi);
      return `已成功将图表切换至 ${kpi}`;
    },
  });

  return (
    <div>
      <h1>企业运营仪表盘 - 当前展示: {selectedKpi}</h1>
      {/* 业务图表组件 */}
    </div>
  );
}

export default function App() {
  return (
    <CopilotKit runtimeUrl="/api/copilotkit">
      <Dashboard />
      <CopilotSidebar defaultOpen={false} clickOutsideToClose={false} />
    </CopilotKit>
  );
}
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **SaaS 平台内置交互式助手**：如 CRM、ERP、数据分析看板等，用户可通过自然语言让 AI 操作表格筛选、生成报表、批量修改表单；
  - **协同文档与编辑器增强**：利用 `<CopilotTextarea>` 实现富文本上下文续写、智能语法润色与自动提炼；
  - **复杂多步骤工作流的 HITL（人工审批）**：结合 LangGraph Co-Agents，在 Agent 执行高风险变更时，直接在界面流式渲染审批确认卡，用户确认后状态机无缝向下流转。
- **潜在不足 / 局限性**：
  - 对前端技术栈有一定倾向性（目前 React/Next.js 生态最为完善，Vue 等其他框架支持度稍低）；
  - 深度利用 Co-Agent 状态同步时，需维护前后端统一的契约类型，并注意长连接通信与网络断线重连的状态一致性。
- **个人评估结论**：**建议核心引入（Adopted）**。构建下一代企业级“生成式 UI + 应用内自治副驾驶”的杀手级前端/全栈基础设施。
