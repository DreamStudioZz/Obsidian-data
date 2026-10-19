---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/vanna-ai/vanna
stars: "23.5k+"
license: MIT
date_added: 2026-10-19
---

# 📦 Vanna

> **一句话简介**：基于三维元数据 RAG 检索与自纠错执行闭环的开源 **Text-to-SQL & 对话式数据分析框架**。通过将 DDL、业务文档注释以及历史黄金 SQL 对构建为向量知识库，精准化解百表级复杂数仓的上下文膨胀与幻觉难题，并支持自愈试执行与数据自学习飞轮。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: vanna-ai/vanna](https://github.com/vanna-ai/vanna) |
| **技术栈** | Python / Chroma / Qdrant / Milvus / OpenAI / Anthropic / HuggingFace |
| **许可证** | MIT |
| **核心特性** | 三维元数据向量化（DDL/Docs/SQL）、动态上下文粗排、自纠错试运行（Self-Correction）、SQLglot 方言转换、高频查询语义缓存 |
| **相关关联** | 关联实践: [[企业级 Text-to-SQL 实战：基于 Vanna 的三维元数据 RAG 检索与自纠错执行流水线]], [[2026-10-19-AI-Digest]], [[Qdrant]], [[Milvus]], [[LlamaIndex]], [[Instructor]] |

---

## 🚀 核心特性与架构亮点

### 1. 突破全量 Schema 限制：三维元数据向量化切片
在企业实际业务场景中，生产数据库往往包含数百张数据表、数千个字段，且伴随大量业务缩写、枚举映射与复合主外键：
- **传统 Text-to-SQL 的崩溃**：直接将全部 `CREATE TABLE` 语句塞入 Prompt，不仅会迅速击穿上下文窗口，带来昂贵 Token 成本，还会让大模型产生注意力分散，频发“字段拼写幻觉”与“错误笛卡尔积关联”；
- **三维分离索引（3D Vectorized Metadata）**：
  1. **DDL 索引**：精准抽取每张表的结构定义与主外键约束；
  2. **业务文档（Documentation）索引**：覆盖指标定义、字段业务含义（如“`is_deleted=0` 代表有效用户”）、口径说明；
  3. **黄金 SQL 对（SQL-Question Pairs）**：沉淀历史经过数据分析师严格审核的高频或高难度 SQL 查询模版。
  每次用户提问时，Vanna 仅通过向量相似度动态召回相关度最高的 Top-K 最小上下文集合，将上下文开销缩减 90% 以上。

### 2. 交互式自纠错试运行回路（Self-Correction & AST Validation）
模型一次性生成的 SQL 往往难以百分之百保证语法正确与业务有效：
- **沙箱编译与预执行**：Vanna 内置 AST 静态语法分析与只读数据库连接池；
- **自愈重试机制**：一旦在预执行或语法解析中抛出异常（如表名大小写敏感、缺少 GROUP BY 投影字段、方言语法不兼容等），Vanna 会自动将错误 Traceback、执行上下文与原 SQL 打包回传给大模型进行针对性自我反思与修复（Reflection），大幅提升最终交付执行率。

### 3. 自进化的企业级数据飞轮（Self-Learning Data Flywheel）
- **从交互中进化**：每次经业务人员或数据分析师确认无误的真实查询，均可一键标记沉淀为新的 `(Question, SQL)` 向量切片；
- **自动化标注与反馈**：系统随着日常运转自动丰富专有业务知识库，越用越聪明，实现低门槛的知识资产滚雪球。

### 4. 极致灵活的解耦架构（Modular & Agnostic）
- **数据库无关**：原生适配 PostgreSQL、MySQL、Oracle、Snowflake、BigQuery、ClickHouse、DuckDB、KingbaseES 等各大主流关系型与分析型数据库；
- **向量库与模型无关**：底层向量引擎可自由挂接 Chroma、Qdrant、Milvus、PGVector、FAISS 等，大模型层支持 OpenAI、Claude、DeepSeek、Ollama 及各类私有化部署开源底座。

---

## 🛠️ 快速上手与典型代码示例

### 1. 安装核心依赖

```bash
# 安装基础包 (以使用 OpenAI + Chroma 为例)
pip install vanna chromadb openai psycopg2-binary
```

### 2. 初始化 Vanna 实例并训练三维知识库

```python
import os
from vanna.openai.openai_chat import OpenAI_Chat
from vanna.chromadb.chromadb_vector import ChromaDB_VectorStore

# 1. 继承组装自定义 Vanna 类 (组合特定 LLM 与向量存储后端)
class EnterpriseVanna(ChromaDB_VectorStore, OpenAI_Chat):
    def __init__(self, config=None):
        ChromaDB_VectorStore.__init__(self, config=config)
        OpenAI_Chat.__init__(self, config=config)

vn = EnterpriseVanna(config={
    'api_key': os.getenv("OPENAI_API_KEY"),
    'model': 'gpt-4o',
    'path': './vanna_chroma_storage'
})

# 2. 注入三维元数据 (DDL + 业务文档 + 历史黄金 SQL)

# A. 训练 DDL 结构
vn.train(ddl="""
CREATE TABLE dim_customer (
    customer_id VARCHAR(64) PRIMARY KEY,
    customer_name VARCHAR(128) NOT NULL,
    vip_level INT DEFAULT 0 COMMENT '0-普通用户, 1-白银, 2-黄金, 3-钻石',
    status INT DEFAULT 1 COMMENT '1-正常, 0-冻结, -1-注销',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
""")

# B. 训练业务文档与口径注释
vn.train(documentation="在统计活跃高价值客户时，必须限定 status = 1 且 vip_level >= 2。")

# C. 训练黄金 SQL 对 (Few-Shot 引导)
vn.train(
    question="统计各类 VIP 等级的正常用户数量分别是多少？",
    sql="SELECT vip_level, COUNT(1) AS user_count FROM dim_customer WHERE status = 1 GROUP BY vip_level ORDER BY vip_level DESC;"
)

# 3. 连接目标业务数据库 (只读权限)
vn.connect_to_postgres(
    host="localhost",
    dbname="crm_analytics",
    user="readonly_analyst",
    password=os.getenv("DB_PASSWORD"),
    port=5432
)

# 4. 执行自然语言问答闭环 (生成 -> 校验 -> 执行 -> 输出图表)
user_query = "查询目前系统中有多少个正常状态的黄金和钻石 VIP 客户？"
sql = vn.generate_sql(user_query)
print("生成的 SQL:\n", sql)

# 在只读数据库执行并返回 DataFrame
df = vn.run_sql(sql)
print("执行结果:\n", df)
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **企业私域 BI 智能看板与即席分析**：为非技术背景的运营与业务人员提供免代码自然语言数据提取入口；
  - **复杂数仓大屏对话机器人**：跨维度查询百张宽表，精准锁定所需维度与指标；
  - **报表系统与自动化巡检**：定时由 Agent 根据自然语言提问生成聚合数据并预警指标波动。
- **潜在不足 / 局限性**：
  - 如果初始知识库中缺少主外键关联的 DDL 说明，多表关联容易漏选或连错桥接字段；
  - 极端复杂的窗口函数（Window Functions）与递归 CTE 仍需人工沉淀特定优质 Few-Shot 样本。
- **个人评估结论**：**生产级 Text-to-SQL 领域架构最扎实、实战落地转化率最高的开源首选方案（Adopted）**。
