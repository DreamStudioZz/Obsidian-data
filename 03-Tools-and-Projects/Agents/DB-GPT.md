---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/eidos-ai/DB-GPT
stars: "17.8k+"
license: MIT
date_added: 2026-10-26
---

# 📦 DB-GPT

> **一句话简介**：业内领先的开源**数据驱动型多智能体系统（Data-Driven Multi-Agent Framework）**。基于自研 **AWEL（智能体工作流表达式语言）**、多模型服务化管理（SBM）与增强型 RAG 架构，专注于将企业私有化数据资产（关系型数据库、数仓、知识库）转化为安全可控的自主业务智能体与 Text-to-SQL 决策中台。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: eidos-ai/DB-GPT](https://github.com/eidos-ai/DB-GPT) |
| **技术栈** | Python 3.10+ / FastAPI / SQLAlchemy / NetworkX / Ray / LangChain / Chroma / DuckDB |
| **许可证** | MIT License |
| **核心特性** | AWEL（声明式有向无环图智能体工作流编排语言）、多智能体自主协同分析（Data Scientist, SQL Reviewer, Visualizer）、端到端 Text-to-SQL 闭环与语法自愈修复、异构私有数仓连接器（MySQL, PG, ClickHouse, Hive, StarRocks）、SBM 本地轻量化模型集群服务、知识库混合检索 RAG |
| **相关关联** | 关联实践: [[企业级数据驱动多智能体协同：基于 DB-GPT 与 AWEL 的私有化 Text-to-SQL 与智能数据分析工程实战]], [[2026-10-26-AI-Digest]], [[Vanna]], [[MetaGPT]], [[Agno]], [[Dify]], [[Langflow]], [[Open-WebUI]] |

---

## 🚀 核心架构与杀手级特性

### 1. 数据驱动型多智能体架构（Data-Driven Multi-Agent Architecture）
传统通用 Agent 框架往往面向开放网络或简单本地文件交互，在面对企业深层业务数据时，存在结构不清晰、幻觉大、难以溯源等致命缺陷。DB-GPT 专为企业级复杂数据分析而设计：
- **角色三权分立**：预置数据架构师（负责数仓 Schema 解析与关联推理）、SQL 审计员（负责语法安全检查、慢查询拦截与数据权限过滤）以及商业智能专家（负责执行结果统计与交互式图表推荐）；
- **数据上下文与状态持久化**：多智能体在分布式环境中共享有状态会话与上下文工作区，能够基于企业数据执行从宏观趋势分析到微观明细下钻的连续探查。

### 2. AWEL（Agentic Workflow Expression Language 智能体工作流表达式语言）
DB-GPT 首创了声明式代码化工作流规范 AWEL，彻底解决了复杂 Agent 流水线在运行期黑盒不可控的难题：
- **代码即工作流（DAG as Code）**：通过简洁直观的管道操作符（如 `>>`）将模型节点、数据连接器、分支判定、异常重试逻辑无缝链接成有向无环图；
- **强类型插槽与流式传输**：各个算子之间严格遵循数据类型契约，支持 Token 级与数据分块级别的流式传输；
- **编排可视化与版本纳管**：所有 AWEL 脚本均可直接序列化为拓扑图，方便与现有 CI/CD 及 GitOps 流程整合。

```
[AWEL 核心声明流向示意]
Natural Question >> Schema Retriever >> Prompt Builder >> LLM (Text-to-SQL) >> SQL Parser & Sandbox Validator >> DB Execution >> Chart Generator
```

### 3. 企业级 Text-to-SQL 增强闭环
Text-to-SQL 不再仅仅是单轮 Prompt 询问，而是由一套精密的工业级工程链路保障：
- **动态 Schema RAG 剪枝**：面对包含数百张表、上万个字段的企业数仓，利用元数据向量索引与属性图自动召回与问题语义最相关的表结构与枚举值，规避上下文溢出；
- **语法校验与自愈回溯（Self-Healing）**：在 SQL 正式提交物理数据库执行前，通过内置 AST 解析器与沙箱解释器预检，若捕获语法错误或字段不存在异常，自动将错误堆栈注入上下文触发 LLM 再次微调修正；
- **只读沙箱与安全合规门禁**：强制注入只读事务隔离、行数限制（LIMIT 兜底）以及危险关键字（DROP, DELETE, UPDATE, TRUNCATE）硬拦截。

### 4. SBM（Service-Based Model）模型治理与私有化多后端管理
- **跨平台模型编排**：原生纳管本地与分布式集群上的推理后端（如 vLLM、LMDeploy、llama.cpp、Ollama）以及外部商用模型（DeepSeek、OpenAI、Qwen、Claude 等）；
- **动态路由与故障自动转移**：在主模型高负载或不可用时，SBM 能够根据预设策略自适应降级至备用本地模型，确保企业业务连续性。

---

## 🛠️ 快速上手与配置

### 1. 环境安装
```bash
# 安装 DB-GPT 核心及常用数据库驱动
pip install "dbgpt[all]"
```

### 2. 命令行一键初始化与启动
```bash
# 初始化配置并拉起 DB-GPT 交互后台（默认端口 5670）
dbgpt start webserver
```

### 3. 基于 AWEL 构建自定义 Text-to-SQL 管道示例
```python
import asyncio
from dbgpt.core.awel import DAG
from dbgpt.core.operator import SimpleOperator, MapOperator
from dbgpt.datasource.rdbms import RDBMSConnector

# 1. 创建带有唯一标识的 AWEL DAG 拓扑
with DAG("enterprise_text_to_sql_pipeline") as dag:
    # 算子 1: 用户问题前处理与意图识别
    question_node = SimpleOperator(operator_name="user_query_input")
    
    # 算子 2: 结合本地元数据索引的 Schema 动态裁剪
    schema_retriever = MapOperator(
        operator_name="dynamic_schema_retriever",
        map_function=lambda query: {
            "query": query,
            "relevant_tables": ["dim_orders", "dim_customers"],
            "ddl_context": "CREATE TABLE dim_orders (order_id INT, amount DECIMAL, create_time DATE);"
        }
    )
    
    # 算子 3: 大模型 Text-to-SQL 生成与语法校验自愈
    sql_generator = MapOperator(
        operator_name="sql_generator_and_verifier",
        map_function=lambda ctx: f"SELECT SUM(amount) AS total_revenue FROM dim_orders WHERE create_time >= '2026-01-01';"
    )
    
    # 算子 4: 数据库安全只读执行
    db_executor = MapOperator(
        operator_name="safe_db_executor",
        map_function=lambda sql: {"status": "SUCCESS", "sql": sql, "result": [{"total_revenue": 1850000.00}]}
    )

    # 声明管道拓扑依赖 (>> 链式操作符)
    question_node >> schema_retriever >> sql_generator >> db_executor

# 4. 执行管道
async def run_pipeline():
    result = await dag.run({"user_query_input": "请统计 2026 年以来的总营业额是多少？"})
    print("Pipeline 执行结果:", result)

if __name__ == "__main__":
    asyncio.run(run_pipeline())
```

---

## 📊 架构方案对比矩阵

| 特性维度 | DB-GPT | Vanna | MetaGPT | 传统 LangChain SQLChain |
| :--- | :--- | :--- | :--- | :--- |
| **架构定位** | **数据驱动多智能体与 AWEL 编排底座** | 纯 Python Text-to-SQL RAG 工具包 | 通用软件工程 SOP 多智能体框架 | 基础单轮 Prompt 调用链 |
| **工作流控制** | **AWEL 声明式强类型 DAG 编排** | 函数级显式调用 | SOP 状态机订阅分发机制 | 弱类型动态链式装配 |
| **数仓生态支持** | **极其广泛（MySQL/PG/ClickHouse/StarRocks）** | 较好（支持主流关系型数仓） | 需自行扩展数据库驱动 | 依赖 SQLAlchemy 基础层 |
| **多角色协同** | **原生具备（数据分析/审查/可视化三角色）** | 无（需自行拼装） | 极强（但缺乏数据领域定制能力） | 无 |
| **安全与合规治理** | **内置只读沙箱、AST 校验与敏感拦截** | 依赖开发者自定义安全层 | 依赖沙箱插件 | 缺乏完备的 AST 级语法自愈 |

---

## 💡 工程实战点评与适用场景

- **推荐选用场景**：
  - **企业私有化 BI 与对话式数仓分析**：需要直接对接内网生产库或离线数仓（ClickHouse、PostgreSQL 等），由业务人员通过自然语言自主生成报表与图表；
  - **严苛数据安全与合规场景**：要求完全离线私有化运行、具备严格数据访问审计、禁止向公网模型传输真实表数据的金融与政企系统；
  - **复杂多步数据分析 Agent 开发**：涉及数据抽取、清洗、多维度交叉统计、多图表联动展示的复合型数据决策流水线。
- **潜在不足 / 局限性**：
  - AWEL 规范相比传统无类型胶水代码具备一定学习曲线；
  - 极复杂跨业务域千行级嵌套 SQL 仍需配合领域词典与 Schema 持续微调训练。
- **个人评估结论**：**生产级强烈推荐采用（status/adopted）**。作为目前将“大模型智能体”、“数据治理”与“声明式工作流”融合得最深入的开源框架，DB-GPT 是构建企业私有化数据智能大脑的首选基础设施。
