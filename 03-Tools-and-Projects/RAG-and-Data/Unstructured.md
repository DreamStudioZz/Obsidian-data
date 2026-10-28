---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/Unstructured-IO/unstructured
stars: "11.5k+"
license: Apache-2.0
date_added: 2026-10-28
---

# 📦 Unstructured

> **一句话简介**：企业级非结构化数据摄取与预处理事实标准开源库，提供高保真版面结构解析（Layout-Aware Partitioning）、表格结构 HTML 化提取与标题语义自适应分块（Chunk-by-Title），为大模型 RAG 和知识库构建提供统一、干净、富元数据的文档数据流。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: Unstructured-IO/unstructured](https://github.com/Unstructured-IO/unstructured) |
| **研发机构** | Unstructured Technologies |
| **技术栈** | Python, ONNX, Tesseract OCR, Poppler / PDFium, Hugging Face |
| **支持格式** | PDF, DOCX, PPTX, XLSX, HTML, EPUB, Markdown, CSV, EML 等 25+ 格式 |
| **关联实践** | [[企业级多模态文档摄取治理：基于 Unstructured 的版面元数据感知与语义自适应分块实战]], [[Docling]], [[MinerU]], [[RAGFlow]] |

---

## 🚀 核心特性与架构亮点

1. **统一的非结构化文档解析接口（Universal Partitioning Pipeline）**：
   - 彻底屏蔽了底层格式的碎片化差异。无论是扫描版 PDF、复杂多栏排版 Word 文档、带嵌入图表的 PPT 还是带多重附件的邮件，均通过统一函数 `partition(filename=...)` 自动识别并解析为标准元素流（Elements Stream）。
2. **多策略版面感知（Layout-Aware Strategies）**：
   - **`fast` 策略**：基于底层文本流与字形坐标直接提取，适用于纯文本排版规整的数字原生 PDF/文档，吞吐速度高达 10~50 页/秒；
   - **`hi_res` 策略**：集成计算机视觉版面分析模型（如 YOLOX / LayoutLM），精确定位文档中的页眉、页脚、正文段落（NarrativeText）、标题（Title）、图表区域及图片；
   - **`ocr_only` 策略**：自动调用 Tesseract / PaddleOCR 引擎处理模糊图片与古籍扫描文档。
3. **表格结构高保真 HTML 提取（High-Fidelity Table Extraction）**：
   - 传统解析器处理多行多列、跨行跨列（Rowspan/Colspan）表格时常导致列数据错位、内容乱序；
   - Unstructured 会将表格精准重建为结构化 HTML 代码字符串，并注入元素元数据 `metadata.text_as_html`，供下游大模型完整理解表头与数值映射。
4. **标题感知与语义自适应分块（Chunk-by-Title & Element Combining）**：
   - 告别简单机械的固定字数切块（Fixed-length Splitting）；
   - 根据文档各级标题层级（Title/Header）自动划分逻辑章节，将短段落向上合并（Combine Text Under N Chars），并在遇到新标题或独立表格时严格切分，确保表格不被拦腰斩断、上下文语义不破碎。
5. **企业级连接器生态（Connectors Ecosystem）**：
   - 原生支持连接 S3、Azure Blob、Google Drive、Notion、Confluence、SharePoint 等企业数据孤岛，并直接对接 Chroma、Pinecone、Weaviate、Qdrant 等主流向量数据库。

---

## 🛠️ 快速上手与调用示例

### 1. 安装核心库与文档解析依赖

```bash
# 安装基础库
pip install unstructured

# 根据需要安装 PDF 高分辨率解析与 OCR 额外扩展
pip install "unstructured[pdf]" "unstructured[docx]" "unstructured[pptx]"
```

### 2. 高保真解析复杂 PDF 并输出结构化 Elements

```python
from unstructured.partition.pdf import partition_pdf

# 采用 hi_res 策略解析研报，同时提取表格 HTML
elements = partition_pdf(
    filename="financial_report_2026.pdf",
    strategy="hi_res",
    infer_table_structure=True,        # 开启表格结构重建
    extract_images_in_pdf=False,
    languages=["chi_sim", "eng"]        # 支持中英文混合 OCR
)

# 遍历查看解析出的元素类型与元数据
for el in elements[:5]:
    print(f"[{el.category}] (页码: {el.metadata.page_number}): {el.text[:60]}...")
    if el.category == "Table":
        print(f" -> 提取的 HTML 表格:\n{el.metadata.text_as_html[:150]}...")
```

### 3. 基于标题层级语义自适应切块（Chunk-by-Title）

```python
from unstructured.chunking.title import chunk_by_title

# 语义分块：依据标题层级组织，保留表格独立性
chunks = chunk_by_title(
    elements=elements,
    max_characters=1500,              # 单块最大字符上限
    new_after_n_chars=1000,           # 超过 1000 字符后若遇新标题则开新块
    combine_text_under_n_chars=300,   # 过短小标题/碎段落自动合并
    multipage_sections=True           # 允许跨页但属于同一逻辑小节的内容合并
)

print(f"总计切分成 {len(chunks)} 个高质量语义块：")
for i, chunk in enumerate(chunks[:3]):
    print(f"--- Chunk #{i+1} [长度: {len(chunk.text)}] ---")
    print(chunk.text)
```

---

## 💡 工程实战点评与选型建议

- **最佳适用场景**：
  - **金融研报与财务审计底稿摄取**：精准保留资产负债表、利润表的完整 HTML 结构，避免数字错行错位；
  - **企业多数据源异构知识库构建**：跨越 Word、PPT、PDF、邮件等格式阻隔，沉淀统一的数据资产标准；
  - **合规审计与可溯源 RAG**：每个 Chunk 保留精确的 `page_number`、`coordinates`（坐标边界框）与版面层级元数据，支持前端一键高亮原文溯源。
- **与同类工具对比**：
  - **相比于 PyPDF / pdfplumber**：底层脚本仅能抽取无序纯文本或需要手动计算边界，Unstructured 提供了端到端的**计算机视觉版面检测与深度元素分类能力**；
  - **相比于 MinerU / Docling**：MinerU 偏重学术论文高精度公式推导还原，Docling 偏向轻量快速转 Markdown，而 Unstructured 则是**工业化企业级 ETL 流水线与连接器生态最完善的标准底座**。
- **综合评估结论**：企业级 RAG 知识库与数据预处理基石工具，强烈推荐采纳（Adopted）。
