---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/docling-project/docling
stars: "18k+"
license: MIT
date_added: 2026-09-22
---

# 📦 Docling

> **一句话简介**：IBM 出品、LF AI & Data 基金会旗下的下一代高精度多模态文档解析引擎，专为 LLM 与 RAG 应用打造，精准将 PDF、DOCX、PPTX、扫描件等复杂文档解析为富结构化 Markdown 与 JSON。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: docling-project/docling](https://github.com/docling-project/docling) |
| **出品方** | Deep Search for Science and Data (DS4SD) / IBM Research |
| **核心特点** | 复杂版面深度解析（DocLayNet）、精准表格恢复、数学公式 OCR、原生 Markdown 导出 |
| **生态集成** | LangChain, LlamaIndex, Haystack, LightRAG |
| **关联实践** | [[双层图谱增强检索：基于 LightRAG 的轻量化 GraphRAG 架构实践]], [[RAG 生产环境优化：多路召回与 Rerank 最佳实践]] |

---

## 🚀 核心特性与亮点

在 RAG 流程中，“Garbage In, Garbage Out”已是共识。传统文本提取工具（如 PyPDF2、pdfminer）往往产生文本断行混乱、表格撕裂成乱码文本、跨页排版错乱等严重问题。

**Docling 的核心技术优势**：
1. **深度版面分析（Advanced Layout Analysis）**：
   - 基于自研视觉模型（DocLayNet 架构），能精准识别多栏混排、页眉页脚、标题层级、浮动侧边栏与正文流向，彻底避免段落拼接错位。
2. **卓越的表格结构重建（Table Structure Recovery）**：
   - 不仅能识别普通表格，更能高精度复原跨行合并单元格（Rowspan/Colspan）、表头嵌套，并无损渲染为语义清晰的 Markdown 或 HTML 表格供 LLM 充分理解。
3. **公式与图表理解（Formula & Code Parsing）**：
   - 支持多模态数学公式识别并转为 LaTeX 格式，代码块保留原生缩进与高亮。
4. **统一文档数据模型（DoclingDocument）**：
   - 输出高度结构化的对象模型，内建标题层级树，非常便于后续执行按照自然段落或章节的 Chunking 切割。
5. **本地离线全私有化运行**：
   - 全程在本地 CPU/GPU 推理执行，数据不离内网，完全满足高敏感合规需求。

---

## 🛠️ 快速上手与配置

### 1. 安装
```bash
pip install docling
```

### 2. 基础文档转换
```python
from docling.document_converter import DocumentConverter

# 初始化转换器（自动加载轻量本地版面识别模型）
converter = DocumentConverter()

# 支持本地路径或远程 URL（PDF, DOCX, PPTX, HTML 等）
source = "https://arxiv.org/pdf/2408.09869"
result = converter.convert(source)

# 1. 导出为工业级清洗后的 Markdown（保留表格、标题与链接）
markdown_content = result.document.export_to_markdown()
print(markdown_content[:1000])

# 2. 导出为带有完整版面坐标与层级树的 JSON
doc_json = result.document.model_dump_json()
```

### 3. 高级配置：启用深度 OCR 与表格增强
```python
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling.datamodel.base_models import InputFormat
from docling.datamodel.pipeline_options import PdfPipelineOptions

pipeline_options = PdfPipelineOptions(
    do_ocr=True,                # 对扫描件/图片执行 OCR
    do_table_structure=True,    # 开启深度表格结构分析
    generate_page_images=False  # 节省内存，关闭页面图像持久化
)

converter = DocumentConverter(
    format_options={
        InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
    }
)

result = converter.convert("financial_report_complex.pdf")
markdown_table_ready = result.document.export_to_markdown()
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 金融研报、科研论文、技术手册、法律合同等富表格、多分栏 PDF 解析。
  - 作为 RAG 数据清洗管道的第一站（Ingestion Pipeline），替代脆弱的规则解析器。
- **潜在不足 / 局限性**：
  - 首次运行需要下载版面分析视觉权重模型（约几百 MB）；在纯 CPU 上解析数百页超长扫描件速度相对受限，建议在生产环境配置 GPU 节点批量处理。
- **个人评估结论**：**建议引入（status/adopted）**。是目前开源界解析复杂版面 PDF 最扎实、输出格式最规整的利器。
