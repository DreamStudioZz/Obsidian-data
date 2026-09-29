---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: RAG-and-Data
github: https://github.com/opendatalab/MinerU
stars: "28k+"
license: Apache-2.0
date_added: 2026-09-29
---

# 📦 MinerU

> **一句话简介**：由上海人工智能实验室 OpenDataLab 开源的一站式工业级多模态高精度文档提取工具（Magic-PDF），专注将包含复杂双栏排版、稠密数学公式与嵌套表格的学术论文与专业研报转换为高质量 Markdown。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: opendatalab/MinerU](https://github.com/opendatalab/MinerU) |
| **核心特点** | 复杂学术版面分析（LayoutLM/YOLO）、行内/块级 LaTeX 公式高保真解析、跨页表格/段落重组、纯净 Markdown 输出 |
| **技术栈** | Python / PyTorch / UniMERNet / PaddleOCR / PDFminer.six |
| **关联实践** | [[复杂文档高保真结构化：基于 MinerU 的论文研报公式还原与跨页版面解析实践]], [[企业级 RAG 深度文档解析：基于 RAGFlow 的视觉模板分块与防幻觉实战]], [[Docling]] |

---

## 🚀 核心特性与技术亮点

1. **复杂阅读版面流精细重构（Reading Order Restoration）**：
   - 传统 PDF 提取工具（如 pypdf、PyMuPDF）基于简单的坐标或字符流，遇到学术论文双栏（Double Column）、图文环绕或图表跨页时，文本行常被横向错误拼接导致逻辑混乱；
   - MinerU 引入经过千万级学术文档训练的版面分析深度模型（Layout Detection），精准识别正文、多级标题、题注（Caption）、页眉、页脚及脚注，剔除噪音并按人类自然阅读顺序线性化组装。
2. **顶尖的 LaTeX 数学公式提取与还原能力**：
   - 依托自主研发的 **UniMERNet** 等公式识别大模型，无论行内公式（Inline Formula: `$E=mc^2$`）还是跨多行的独立块级复杂公式（Block Formula），均能精确识别并输出标准 LaTeX 语法代码，彻底解决 RAG 知识库检索金融/工科公式时沦为乱码的痛点。
3. **表格结构高保真还原与转写**：
   - 集成 Table Structure Recognition 视觉模型，自动识别无框线、合并单元格（Colspan/Rowspan）及多层表头的复杂表格，精准转换为语义清晰的标准 HTML 表格或 Markdown 表格，并自动提取关联的 Table Caption 作为上下文元数据。
4. **多模态图表裁剪与独立索引**：
   - 自动截取正文中的插图、架构图与示意图，保存为独立高清晰度图片文件，并在生成的 Markdown 相对路径中保留标准引用图片标签，为后续多模态多路检索与图文混合回答提供完整物料。

---

## 🛠️ 快速上手与集成

### 1. 安装 Magic-PDF (MinerU 核心引擎)

```bash
# 推荐在独立虚拟环境安装（支持 CUDA 加速）
pip install -U magic-pdf[full] --extra-index-url https://wheels.myhloli.com

# 首次使用下载内置版面与公式模型权重
magic-pdf-models-download
```

### 2. 命令行批量提取

```bash
# 单文件高速转 Markdown 并导出提取图片与表格
magic-pdf -p sample_paper.pdf -o ./output_dir -m auto
```

### 3. Python 管道级调用代码

```python
import os
from magic_pdf.pipe.UNIPipe import UNIPipe
from magic_pdf.rw.DiskReaderWriter import DiskReaderWriter

def parse_pdf_to_markdown(pdf_path: str, output_dir: str):
    pdf_bytes = open(pdf_path, "rb").read()
    image_writer = DiskReaderWriter(os.path.join(output_dir, "images"))
    
    # 初始化多任务解析管道
    pipe = UNIPipe(pdf_bytes, {"_pdf_type": "", "model_list": []}, image_writer)
    pipe.pipe_classify()
    pipe.pipe_analyze()
    pipe.pipe_parse()
    
    # 导出纯净 Markdown 内容
    md_content = pipe.pipe_mk_markdown(os.path.join(output_dir, "images"), drop_mode="none")
    return md_content
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - 专业领域 RAG（科研学术机构、金融券商研报、半导体与生物医药技术文档）：对数学公式、数据表格的解析准确率有严苛要求；
  - 大模型预训练语料合成：将数十万本历史扫描版或电子版 PDF 批量转换为干净的预训练或 SFT 纯文本语料；
  - 配合 RAG 知识切块系统（如 Chunking 按段落/标题切分），大幅降低大模型阅读上下文时的格式幻觉。
- **潜在不足 / 局限性**：
  - 依赖深度学习多阶段推理（Layout + OCR + Formula），单页解析耗时（GPU 下约 1~3 秒/页）高于纯规则正则提取库，海量文档建议配置异步分布式 Celery / Ray 工作队列。
- **评估结论**：**学术与专业复杂文档解析首选（Adopted）**。与注重纯规则的轻量库形成完美互补，是高保真数据入库的标杆工具。
