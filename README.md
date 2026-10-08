# 🤖 CVIP RAG System

> A lightweight Databricks-powered Retrieval-Augmented Generation app for Computer Vision and Image Processing questions.

[![Databricks](https://img.shields.io/badge/Powered%20by-Databricks-red)](https://databricks.com)
[![LLaMA](https://img.shields.io/badge/LLM-LLaMA%203.3%2070B-blue)](https://ai.meta.com)
[![Python](https://img.shields.io/badge/Python-3.10%2B-green)](https://python.org)
[![Streamlit](https://img.shields.io/badge/UI-Streamlit-FF4B4B)](https://streamlit.io)

---

## 📌 Overview

CVIP RAG is a focused question-answering app for topics in Computer Vision and Image Processing. It answers user questions by:

- retrieving top relevant chunks from a Databricks Vector Search index,
- building a context window from those chunks,
- sending the question and context to a Databricks-hosted LLM,
- returning the answer with source-style citation labels in the UI.

This repository is intentionally compact and practical: the actual runtime behavior is defined primarily by `app.py`, while `rag_components.py` contains supporting utilities for source organization, metadata handling, and schema setup.

---

## ✨ What the app does

- Answers CVIP questions using retrieved document chunks
- Shows a Streamlit chat interface for interactive use
- Uses a Databricks Vector Search index for semantic retrieval
- Uses a Databricks model endpoint for grounded answer generation
- Extracts citation labels from the model output and displays them in the app

---

## 🏗️ Current implementation architecture

```text
User Query
    │
    ▼
┌─────────────────────┐
│ Streamlit UI        │
│ app.py              │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Vector Search       │
│ endpoint: cvip_endpoint │
│ index: workspace.default.cvip_chunks_vs_index │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Context Builder     │
│ Top 5 relevant chunks │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Databricks LLM      │
│ databricks-meta-llama-3-3-70b-instruct │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Answer + Citations │
│ Rendered in UI     │
└─────────────────────┘
```

This is the actual runtime flow implemented in the repo today.

---

## 📁 Repository structure

```text
CVIP_RAG/
├── app.py                  # Main Streamlit application
├── rag_components.py       # Supporting Databricks/source utilities
├── requirements.txt        # Python dependencies
├── app.yaml                # Databricks Apps deployment config
├── README.md               # Project documentation
└── .gitignore              # Git ignore file (if present)
```

---

## 🧩 Main files

### `app.py`
This is the real user-facing application entry point.

It contains:

- Streamlit page configuration and styling
- chat session state management
- environment variable configuration
- `query_vector_search()` for Databricks Vector Search calls
- `query_llm()` for model inference via `mlflow.deployments`
- `ask()` orchestration logic for retrieval + generation
- UI logic for showing history, answers, citations, and metrics

Core runtime behavior:

- fetches the configured vector index with `VectorSearchClient`
- searches for `num_results=5`
- reads `chunk_id`, `content`, `citation_label`, and `page_number`
- builds a context string from the top chunks
- invokes the LLM endpoint with a system prompt instructing it to answer only from the provided context
- extracts citations from responses matching `[Source: ...]`

### `rag_components.py`
This file is a larger support script and not the main app driver. It contains patterns for:

- source classification and inventory tracking
- Databricks table schema definitions for documents and chunks
- scanning and cataloging source folders
- helper functions for metadata extraction and content processing
- classification documentation generation

It appears to be an internal pipeline/setup utility for preparing the knowledge base behind the app.

### `requirements.txt`
```txt
streamlit>=1.28.0
databricks-vectorsearch>=0.22
mlflow>=2.9.0
```

### `app.yaml`
```yaml
command: ["streamlit", "run", "app.py", "--server.port", "8080", "--server.address", "0.0.0.0"]
```

This is the configuration used to run the app in a Databricks Apps environment.

---

## ⚙️ Configuration

The app currently expects the following environment variables and resources:

| Component | Value |
|-----------|-------|
| `DATABRICKS_HOST` | Databricks workspace URL |
| `DATABRICKS_TOKEN` | Personal access token |
| Vector Search Endpoint | `cvip_endpoint` |
| Vector Search Index | `workspace.default.cvip_chunks_vs_index` |
| LLM Endpoint | `databricks-meta-llama-3-3-70b-instruct` |

These values are defined directly in `app.py` and should match your Databricks deployment.

---

## 🚀 Getting started

### Prerequisites

- Databricks workspace access
- Valid Databricks token
- Vector Search endpoint configured
- Vector index available and populated
- LLM serving endpoint available

### Install

```bash
pip install -r requirements.txt
```

### Set environment

```bash
export DATABRICKS_HOST="https://<your-workspace>.cloud.databricks.com"
export DATABRICKS_TOKEN="<your-personal-access-token>"
```

### Run locally

```bash
streamlit run app.py
```

---

## 🔄 Query flow in practice

```text
1. User enters a CVIP question
2. app.py sends query to vector search
3. Top relevant chunks are retrieved
4. Chunks are trimmed and assembled into context
5. LLM answers using only the retrieved context
6. Citations are extracted and displayed in the UI
```

This is a classic RAG pattern, implemented here as a lightweight, Databricks-native Streamlit app.

---

## 💡 Example prompts

- What is edge detection?
- How does Sobel edge detection work?
- Explain convolutional neural networks
- What are vision transformers?
- How do image segmentation and classification differ?

---

## 🛠️ Tech stack

| Layer | Technology |
|-------|------------|
| UI | Streamlit |
| Retrieval | Databricks Vector Search |
| Model serving | Databricks MLflow Deployments |
| LLM | LLaMA 3.3 70B Instruct |
| Language | Python |
| Deployment | Databricks Apps |

---

## 📊 Current project maturity

This repo is best described as a focused prototype / proof-of-concept app rather than a large-scale production system. It currently includes:

- a real user-facing chat app,
- retrieval against a Databricks vector index,
- LLM-based answer generation,
- a larger supporting script for source preparation and metadata workflow,
- but no large packaged service structure, no authentication layer, and no formal test suite.

---

## ⚠️ Important limitations

- Requires live Databricks connectivity
- Depends on a valid vector index and LLM endpoint
- No built-in user authentication in the app itself
- No offline fallback if Databricks services are unavailable
- No custom reranking logic beyond the underlying vector index behavior
- Chat history is session-scoped and not a full persistent app memory system

---

## 👨‍💻 Author

**Dhanush Kumar**  
Final Year Project — Computer Vision & Image Processing

---

## 📚 Learning resources

- [Databricks Vector Search Docs](https://docs.databricks.com/en/generative-ai/vector-search.html)
- [Streamlit Docs](https://docs.streamlit.io)
- [MLflow Docs](https://mlflow.org/docs/latest/index.html)
- [Meta LLaMA](https://ai.meta.com/llama/)

---

## ✅ Summary

This repository is a practical Databricks-integrated RAG app for CVIP question answering. It is small, focused, and grounded in the actual codebase: the app is driven by `app.py`, the supporting utilities live in `rag_components.py`, and the runtime depends on Databricks Vector Search plus a hosted LLM endpoint.

If you want to keep the README even more visually polished, I can also add a stronger landing-page style with a hero section, badges, feature cards, and a short architecture diagram while keeping the wording grounded to the implementation.
