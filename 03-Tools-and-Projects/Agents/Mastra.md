---
tags:
  - ai-tool
  - open-source
  - status/evaluating
category: "Agents"
github: "https://github.com/mastra-ai/mastra"
stars: "29.5k+"
license: "Apache-2.0"
date_added: 2026-11-20
---

# 📦 Mastra

> **一句话简介**：面向现代 TypeScript / Next.js 全栈开发者的生产级 AI 智能体与持久化工作流框架，结合 Zod 强类型工具契约、Durable 状态机与 Mastra Studio 本地全链路可观测。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub Repo](https://github.com/mastra-ai/mastra) |
| **技术栈** | TypeScript / Node.js / Next.js / Zod |
| **许可证** | Apache-2.0 (核心框架开源) / Mastra Enterprise License (企业模块) |
| **相关关联** | 关联专题: [[LangGraph]], [[PydanticAI]], [[CopilotKit]], [[FastMCP]] |

---

## 🚀 核心特性与亮点

1. **TypeScript 原生与端到端类型安全 (End-to-End Type Safety)**：
   - 彻底摆脱 Python 动态类型与松散字典在大型企业工程中的不可控性。Mastra 深度集成 **Zod**，工具定义入参与出参具备 100% 编译期类型推导与运行时 Schema 校验。
   - 大模型工具调用（Tool Calls）与结构化响应直接反序列化为类型确定的 TypeScript 对象，零手动字符串解析与规避隐式崩溃。

2. **可持久化状态机工作流 (Durable Workflows)**：
   - 传统脚本式 Agent 极易在多步链式调用时因为网络抖动、模型限流或超时而全盘重来。
   - Mastra 提供开箱即用的异步工作流引擎，原生支持分步执行、条件分支（`.step().then()`）、批量并发处理（`.foreach()`）以及长生命周期的人机协同断点挂起与恢复（Human-in-the-Loop Suspend & Resume）。

3. **开箱即用的本地开发者平台 (Mastra Studio)**：
   - 运行 `npx mastra dev` 即可在本地启动强大的交互式控制台（默认 `http://localhost:4111`）。
   - 开发者可直接在浏览器中与 Agent 实时交互、可视化调试多步 Workflow 图拓扑、逐帧回放 Span 级调用耗时与 Token 开销，并能一键导出标准 **OpenAPI / Swagger** 规范。

4. **模块化 RAG 原语与模型网关路由**：
   - 内置轻量向量检索与文档分块器，支持连接 pgvector、Pinecone、Qdrant 等主流存储底座。
   - 统一抽象主流大模型服务商（OpenAI、Anthropic、Google Gemini、Mistral 以及本地 Ollama），支持无缝切换与模型级 Fallback 容灾。

5. **与同类工具对比**：
   - **相比 LangChain / LangGraph (JS/TS)**：Mastra 的代码心智模型更贴近现代 TypeScript 开发者，摒弃了庞杂晦涩的抽象链条，启动和执行开销更轻，打包产物更小，对 Next.js App Router 与 Serverless 边缘运行时天然友好。
   - **相比 PydanticAI**：PydanticAI 专注于 Python 后端生态，而 Mastra 填补了 Node.js / Full-stack 工程师在构建生产级复杂 Agent 时的强类型与工程化空白。

---

## 🛠️ 快速上手与配置

### 1. 初始化项目与安装依赖

```bash
# 初始化基于 Mastra 的全新项目
npm create mastra@latest my-agent-app
cd my-agent-app
npm install

# 或者将 @mastra/core 集成进现有项目
npm install @mastra/core zod
```

### 2. 定义强类型工具与智能体

```typescript
// src/mastra/agents/index.ts
import { Agent } from "@mastra/core/agent";
import { createTool } from "@mastra/core/tools";
import { z } from "zod";

// 1. 定义具备 Zod 校验的强类型工具
const weatherTool = createTool({
  id: "get-weather",
  description: "获取指定城市的实时天气与温度预警",
  inputSchema: z.object({
    city: z.string().describe("城市名称，例如 Beijing, Shanghai"),
    unit: z.enum(["celsius", "fahrenheit"]).default("celsius"),
  }),
  outputSchema: z.object({
    temperature: z.number(),
    condition: z.string(),
    humidity: z.number(),
  }),
  execute: async ({ context }) => {
    // 业务实现逻辑
    return {
      temperature: 22.5,
      condition: "Clear Sky",
      humidity: 45,
    };
  },
});

// 2. 声明生产级智能体
export const weatherAssistant = new Agent({
  name: "Weather & Travel Assistant",
  instructions: "你是一位专业的出行气象助手，根据用户行程合理调用天气工具并给出穿衣与出行建议。",
  model: {
    provider: "OPEN_AI",
    name: "gpt-4o",
  },
  tools: { weatherTool },
});
```

### 3. 构建 Durable 状态机工作流

```typescript
// src/mastra/workflows/index.ts
import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";

const fetchStep = new Step({
  id: "fetch-travel-data",
  inputSchema: z.object({ destination: z.string() }),
  outputSchema: z.object({ itinerary: z.array(z.string()) }),
  execute: async ({ context }) => {
    return {
      itinerary: ["故宫博物院", "颐和园", "天坛公园"],
    };
  },
});

const generatePlanStep = new Step({
  id: "generate-plan",
  inputSchema: z.object({ itinerary: z.array(z.string()) }),
  outputSchema: z.object({ summary: z.string() }),
  execute: async ({ context }) => {
    return {
      summary: `规划完成，行程涵盖 ${context.itinerary.join("、")}`,
    };
  },
});

export const travelWorkflow = new Workflow({
  name: "travel-planning-workflow",
})
  .step(fetchStep)
  .then(generatePlanStep)
  .commit();
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  1. **现代全栈 Web 应用**：使用 Next.js、Remix、Nuxt 等构建由 AI 驱动的 SaaS 产品，需要直接在 Node.js / Edge 后端嵌入 Agent 能力；
  2. **企业级业务系统集成**：需要通过 TypeScript 强契约对接内网 CRM、ERP 或数据库，要求严格的参数校验、确定性错误处理与可审计日志；
  3. **长流程自动化任务**：包含数据拉取、LLM 结构化抽取、人工审批流等步骤的复杂异步作业。
- **潜在不足 / 局限性**：
  - 部分高级企业级能力（如复杂组织级 RBAC、FGA 细粒度权限控制与高级可视化构建器）归属于商业源许可（EE）；
  - 相比 Python 庞大的数据科学和微调训练生态，TypeScript 生态在底层算子和本地模型权重操作上仍需借助外部 API 或独立服务。
- **个人评估结论**：**建议重点引入与落地评估**。对于前端及全栈团队而言，Mastra 是目前市面上设计最克制、工程架构最清晰、DX（开发者体验）最出色的 TypeScript Agent 基础设施。