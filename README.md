# CVIP RAG

A small Databricks-backed retrieval-augmented generation application for asking questions about Computer Vision and Image Processing.

This repository does not implement a large multi-module ML platform. The actual app is a single Streamlit interface that:

- searches a Databricks Vector Search index for relevant CVIP text chunks,
- builds a context window from the top results,
- sends that context and the user question to a Databricks-hosted LLM endpoint,
- returns the answer with lightweight source labels in the UI.

---

## What is in this repo?

The repository currently contains four project files:

- `app.py` — the runnable Streamlit application
- `rag_components.py` — a large supporting script that defines Databricks table schemas, source inventory logic, and volume scanning utilities
- `requirements.txt` — Python dependencies
- `app.yaml` — deployment configuration for Databricks Apps
- `README.md` — project documentation

---

## Current implementation summary

### 1. Streamlit app (`app.py`)

`app.py` is the real user-facing application entry point.

Key behavior:

- sets up a wide-layout Streamlit page with a custom answer box and citation styling
- creates session state for chat history and a session identifier
- defines environment variables:
  - `DATABRICKS_HOST` (default: a Databricks workspace URL)
  - `DATABRICKS_TOKEN` (optional, default empty string)
- uses a model endpoint named:
  - `databricks-meta-llama-3-3-70b-instruct`
- uses a Vector Search index named:
  - `workspace.default.cvip_chunks_vs_index`
- uses a Vector Search endpoint named:
  - `cvip_endpoint`

The main logic is:

- `query_vector_search(query)`
  - creates a `VectorSearchClient`
  - fetches the configured index using `get_index(endpoint_name=..., index_name=...)`
  - calls `similarity_search(...)` with:
    - `query_text=query`
    - `columns=["chunk_id","content","citation_label","page_number"]`
    - `num_results=5`
  - converts the returned rows into a list of chunk dictionaries with:
    - `content`
    - `citation_label`
    - `page_number`

- `query_llm(query, context)`
  - calls `mlflow.deployments.get_deploy_client("databricks")`
  - sends prompt messages to the Databricks model endpoint
  - system prompt:
    - "You are an expert in Computer Vision and Image Processing. Answer ONLY using the provided context. Cite sources using [Source: name] format."
  - passes `max_tokens=800` and `temperature=0.1`
  - returns the generated answer text

- `ask(query)`
  - retrieves 5 chunks
  - if none are found, returns:
    - `"No relevant information found."`
  - builds a context block by concatenating each chunk and trimming each to ~500 characters
  - sends the query and context to the LLM
  - extracts citations from the answer using a regex:
    - `re.findall(r"\[Source:([^\]]+)\]", answer)`
  - returns:
    - `answer`
    - `citations`
    - `latency_ms`
    - `chunks`

The Streamlit UI exposes:

- a sidebar with:
  - app title
  - readiness indicator
  - checkbox for showing sources
  - button to start a new chat
  - example prompts
- chat interface for user input
- answer cards with citation expansion
- metrics for chunks retrieved and latency

The app uses a simple chat history stored in `st.session_state` and reruns after each answer.

---

### 2. Supporting data/indexing utility (`rag_components.py`)

`rag_components.py` is much larger and looks like a Databricks notebook-style resource-setup and metadata-management script, not the runtime application itself.

From the visible code, it contains:

- SQL schema definitions for tables such as:
  - `cvip_documents`
  - `cvip_chunks`
  - `cvip_source_config`
  - `cvip_query_logs`
- Delta table configuration and settings
- support for indexing and classifying source files by tier
- logic for scanning volume directories and classifying PDFs by directory
- helper functions such as:
  - `load_source_config()`
  - `scan_volume_sources()`
  - `get_classification_from_path()`
  - `get_all_sources_by_tier()`
- a generated classification report routine for human-readable documentation

This file appears to be an internal tooling script for preparing and organizing CVIP source material and metadata for the RAG pipeline. It is not the main app entry point.

The main app (`app.py`) is the code that actually performs retrieval + generation in the user-facing workflow.

---

## Runtime architecture

The production flow in the current repository is straightforward:

1. User enters a question in the Streamlit app.
2. App queries Databricks Vector Search using the configured index and endpoint.
3. Top 5 relevant chunks are retrieved.
4. The text chunks are trimmed and concatenated into a context block.
5. The LLM endpoint is called with the instruction prompt and context.
6. The generated answer is displayed in the UI.
7. Any citations matching `[Source: ...]` are extracted and shown in the source panel.

This is a classic RAG pattern, but in this repo it is implemented as a lightweight single-file app rather than a complex modular system.

---

## Deployment configuration

`app.yaml` contains:

```yaml
command: ["streamlit", "run", "app.py", "--server.port", "8080", "--server.address", "0.0.0.0"]
```

This means the app is intended to run as a Databricks App or a similar server-managed Streamlit deployment.

---

## Dependencies

`requirements.txt` contains:

```txt
streamlit>=1.28.0
databricks-vectorsearch>=0.22
mlflow>=2.9.0
```

This is a minimal set for:

- UI rendering
- vector retrieval
- model invocation via Databricks MLflow deployment client

---

## How to run locally

Install dependencies:

```bash
pip install -r requirements.txt
```

Set the required environment variables before starting the app:

```bash
export DATABRICKS_HOST="https://<your-workspace>.cloud.databricks.com"
export DATABRICKS_TOKEN="<your-token>"
```

Run the app:

```bash
streamlit run app.py
```

If deployed via Databricks Apps, the `app.yaml` command will start it automatically.

---

## Prerequisites for the app to work

This repo expects the following Databricks resources to already exist:

- a Databricks workspace accessible via `DATABRICKS_HOST`
- a valid `DATABRICKS_TOKEN`
- a Databricks Vector Search endpoint named `cvip_endpoint`
- a Vector Search index `workspace.default.cvip_chunks_vs_index`
- a serving / deployment endpoint named `databricks-meta-llama-3-3-70b-instruct`

Without those resources, the app will not have any retrieval or generation backend to call.

---

## Important limitations of the current codebase

This repo is intentionally small and practical rather than highly abstracted. The current implementation has a few notable limitations:

- no test suite
- no formal package structure
- no database migration pipeline in the app itself
- no user authentication or multi-user management
- no offline fallback when Databricks services are unavailable
- no custom retrieval ranking logic beyond the Vector Search index defaults
- no persistent logging beyond the in-memory chat session

In other words, the repository is best understood as a focused Databricks RAG demo for CVIP content rather than a full production-grade product system.

---

## Repository purpose

The repo is meant to help answer questions in the fields of:

- computer vision
- image processing
- digital image fundamentals
- convolutional networks
- transformers in vision
- general CVIP concepts and methods

The app is grounded in chunk retrieval from a prebuilt CVIP knowledge base and then uses a language model to answer using the retrieved context.

---

## Bottom line

This project is a compact, Databricks-integrated Streamlit application for CVIP question answering. It is implemented as a practical retrieval + generation app, with a larger auxiliary script for source inventory and schema setup, but the actual end-user behavior is defined primarily by `app.py`.

That is the level of implementation reflected in the code today.
