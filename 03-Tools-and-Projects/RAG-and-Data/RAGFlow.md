---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/infiniflow/ragflow
stars: "85k+"
license: Apache-2.0
date_added: 2026-09-23
---

# 📦 RAGFlow

> **一句话简介**：基于深度文档理解（Deep Document Parsing）的企业级开源 RAG 引擎，通过视觉版面分析与结构感知切块，完美破解 PDF 表格错乱、双栏混淆与多模态扫描件解析难题，提供具备原文高亮可溯源的“零幻觉”检索生成流水线。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: infiniflow/ragflow](https://github.com/infiniflow/ragflow) |
| **核心特点** | 深度版面理解（DeepDoc）、模板感知智能分块、多路召回 + 重排、图文原文锚定 |
| **存储底座** | Infinity 搜索引擎 / Elasticsearch、主流向量数据库 |
| **关联实践** | [[企业级 RAG 深度文档解析：基于 RAGFlow 的视觉模板分块与防幻觉实战]], [[Docling]], [[LightRAG]] |

---

## 🚀 核心特性与技术亮点

1. **“所见即所得”的深度版面分析（Deep Document Parsing, DeepDoc）**：
   - 传统 RAG 的字符截断往往将表格行拆碎、跨栏文字串读，导致向量切片语意完全残缺。
   - RAGFlow 基于视觉 OCR 与版面识别模型，精准识别文章标题层级、段落流向、页眉页脚，并将复杂合并单元格表格完整还原为结构化数据。
2. **场景化智能切块模板（Template-based Chunking）**：
   - 内置通用文档（General）、学术论文（Paper）、财务研报（Financial）、操作手册（Manual）、法规条款（Laws）、问答对（Q&A）等多种针对性切块模板；
   - 保证切分出的每一个 Chunk 在语义上高度自洽完整，彻底避免无效截断。
3. **精准原文高亮溯源（Verifiable Grounded Citations）**：
   - 生成的每个回答均精确对应到原始文档的页码与高亮矩形区域，用户点击即可一键核验事实真伪，极大降低大模型幻觉对企业决策的负面影响。
4. **内置 Agentic 智能编排流程**：
   - 不仅支持基础 RAG 问答，还提供类似 Dify / Langflow 的可视化工作流编排，支持在检索前后插入实体抽取、条件路由和 API 动作。

---

## 🛠️ 快速上手与部署

### 1. 基于 Docker Compose 快速拉起服务

```bash
git clone https://github.com/infiniflow/ragflow.git
cd ragflow/docker
# 拉取镜像并启动
docker compose up -d
# 服务就绪后访问 Web 控制台: http://localhost
```

### 2. 通过 Python SDK 上传文档并执行检索

```python
from ragflow_sdk import RAGFlow

# 初始化客户端
rag = RAGFlow(api_key="ragflow-api-key", base_url="http://localhost:9380")

# 创建知识库并指定智能切块模板
dataset = rag.create_dataset(
    name="Enterprise_Policies",
    parser_config={"name": "manual"} # 使用设备手册/规章制度模板
)

# 上传复杂 PDF
document = dataset.upload_document(filepath="./2026_Annual_Report.pdf")

# 执行问答检索
res = rag.retrieve(
    dataset_ids=[dataset.id],
    question="2026年第三季度海外业务的净利润率是多少？",
    top_k=5
)
for chunk in res:
    print(f"得分: {chunk.similarity:.4f} | 页码: {chunk.page_num}")
    print(f"内容: {chunk.content}\n")
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **金融研报 / 法律合同 / 医药文献检索**：文档中包含海量多栏排版、跨页嵌套表格、扫描印章等复杂元素；
  - **企业内部知识库与私有合规审计**：对答案必须提供 100% 可追溯的原始凭证引用。
- **与 Naive RAG / 传统切块工具对比**：
  - 传统按 Token 盲切的检索准确率在复杂表格场景通常低于 45%，而基于 RAGFlow 深度版面重构后，**关键事实召回率可稳定提升至 85% 以上**。
- **综合评估结论**：企业级复杂文档解析与防幻觉 RAG 事实标杆，强烈推荐采纳（Adopted）。
