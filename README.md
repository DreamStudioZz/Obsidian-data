<div align="right">
  <strong>English</strong> | <a href="README_CN.md">中文</a>
</div>

# 🧠 Personal AI Knowledge Vault (Obsidian)

> An industrial-grade personal AI knowledge base and daily radar powered by an AI pair-programming assistant, built with **Obsidian**, **Dataview**, and bidirectional linking.

This repository documents the structural taxonomy of the knowledge vault and outlines the automated workflow for daily AI intelligence tracking, engineering card distillation, and continuous cloud backup.

---

## 🔄 Daily Automation Workflow: How to Trigger AI Collection

### Method 1: On-Demand Natural Prompt Trigger (Recommended)
Whenever you want to capture the latest AI developments or distill cutting-edge knowledge, simply send in the chat:
- **`今日AI日报`** / **`收集今日AI知识`** (or *“Collect today's AI knowledge”*)
- The AI assistant will automatically execute the end-to-end closed loop:
  1. **Intelligence Tracking**: Scrapes and monitors trending GitHub repositories, conferences, and top-tier open-source engineering practices;
  2. **Value Distillation**: Filters out hype and extracts production-grade frameworks, architecture patterns, and pitfall avoidance strategies;
  3. **Daily Digest Archival**: Compiles the daily brief under `01-Daily-Digest/YYYY/YYYY-MM/YYYY-MM-DD-AI-Digest.md` following standard templates;
  4. **Card Extraction**: Extracts high-star tools and breakthrough practices into atomic cards under `03-Tools-and-Projects/` and `04-Engineering-Practices/`, cross-linking them with bidirectional wikilinks;
  5. **Auto Git Push**: Automatically runs `git add`, crafts a standardized `git commit`, and pushes directly to the remote repository (`git push origin main`) for seamless multi-device cloud synchronization.

### Method 2: Custom Deep-Dive Research
When investigating a specific technical topic or architecture, you can prompt directly:
- *"Research the most popular open-source Coding Agent frameworks and summarize into tool cards."*
- *"Write a best practice guide on Parent-Child Chunking in enterprise RAG systems."*
- The AI assistant creates formatted Markdown cards into the designated directories and pushes them to GitHub.

---

## 🗂️ Directory Taxonomy & Card Conventions

The root directory is structured into clean, purposeful modules:

- **`00-Home/`**:
  - `000-AI-Index.md`: Master Map of Content (MOC), powered by Dataview dynamic tables that automatically aggregate and render all cards grouped by category.
- **`01-Daily-Digest/`**:
  - Daily intelligence streams organized chronologically: `YYYY/YYYY-MM/YYYY-MM-DD-AI-Digest.md` (e.g., `2026/2026-09/2026-09-21-AI-Digest.md`).
- **`03-Tools-and-Projects/`**:
  - Open-source frameworks and libraries categorized into 4 core domains:
    - **`Agents/`**: Agentic loops, protocols, and memory systems (e.g., PydanticAI, FastMCP, Mem0)
    - **`Inference/`**: Inference engines, private deployment backends, and gateway routers (e.g., vLLM, SGLang, LiteLLM)
    - **`RAG-and-Data/`**: Vector retrieval, code/knowledge graphs, data enrichment (e.g., Graphify)
    - **`Evaluation-and-Ops/`**: Observability, evaluation & scoring, prompt management (e.g., Langfuse)
- **`04-Engineering-Practices/`**:
  - Production architecture solutions, performance optimization, and troubleshooting guides (e.g., Prompt Caching architecture, Hybrid Search & RRF Reranking).
- **`99-Templates/`**:
  - Standard Obsidian templates (Daily Digest, Tool Card, Practice Card) ensuring consistent YAML Frontmatter and layout across all notes.

---

## 💡 Recommended Obsidian Plugins

1. **Dataview** (Essential):
   - Enables SQL-like querying over note metadata. Used on the dashboard `000-AI-Index.md` to dynamically list cards without manual index maintenance.
2. **Omnisearch**:
   - Provides AI-powered semantic and hybrid search across all local markdown notes.
