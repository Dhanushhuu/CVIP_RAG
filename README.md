# 🤖 CVIP RAG

**An intelligent Q&A system for Computer Vision and Image Processing**

Powered by Databricks Vector Search and LLaMA 3.3 70B

---

## What is CVIP RAG?

CVIP RAG is a retrieval-augmented generation application that answers questions about computer vision and image processing topics by retrieving relevant knowledge from a curated CVIP knowledge base and generating contextually grounded responses.

Ask it about edge detection, CNNs, vision transformers, image segmentation, or any core CVIP concept — and get cited, evidence-based answers.

---

## ✨ Key Features

- **📚 Smart Retrieval** — Searches a Databricks Vector Search index for the most relevant CVIP content
- **🧠 Expert Answers** — Leverages LLaMA 3.3 70B to generate high-quality, in-domain responses
- **📖 Full Citations** — Every answer includes source labels and page numbers
- **⚡ Fast & Responsive** — Typical response latency under 6 seconds
- **💬 Session Memory** — Chat history persists within each session
- **🎨 Clean UI** — Built with Streamlit for an intuitive, accessible interface

---

## 🚀 Quick Start

### Prerequisites

You'll need:
- A Databricks workspace (AWS, Azure, or GCP)
- A valid Databricks personal access token
- The following Databricks resources pre-configured:
  - A Vector Search endpoint (`cvip_endpoint`)
  - A Vector Search index (`workspace.default.cvip_chunks_vs_index`)
  - A serving endpoint for `databricks-meta-llama-3-3-70b-instruct`

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Dhanushhuu/CVIP_RAG.git
   cd CVIP_RAG
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables:**
   ```bash
   export DATABRICKS_HOST="https://<your-workspace>.cloud.databricks.com"
   export DATABRICKS_TOKEN="<your-personal-access-token>"
   ```

4. **Run the app:**
   ```bash
   streamlit run app.py
   ```

The app will start at `http://localhost:8501`

### Deploy to Databricks Apps

To deploy as a managed Databricks App, use the included `app.yaml`:

```bash
databricks apps deploy --source-code-path . --config app.yaml
```

---

## 💡 How It Works

```
User Question
      ↓
Vector Search Retrieval
      ↓
Context Assembly (top 5 chunks)
      ↓
LLM Generation (LLaMA 3.3 70B)
      ↓
Citation Extraction & Display
```

**The retrieval pipeline:**

1. **Vector Search** — Your question is embedded and matched against 10,000+ indexed CVIP chunks
2. **Context Building** — The top 5 most relevant chunks are trimmed to ~500 characters each and concatenated
3. **LLM Generation** — The context and question are sent to a Databricks-hosted LLaMA model with a system prompt instructing it to cite sources
4. **Source Extraction** — Citations in the format `[Source: ...]` are automatically extracted and displayed below the answer

---

## 📦 Repository Structure

```
CVIP_RAG/
├── app.py                    # Main Streamlit application
├── rag_components.py         # Data prep & metadata utilities
├── requirements.txt          # Python dependencies
├── app.yaml                  # Databricks Apps deployment config
└── README.md                 # This file
```

### app.py
The user-facing Streamlit application. Implements:
- Vector search retrieval (`query_vector_search`)
- LLM answer generation (`query_llm`)
- Session-based chat history
- Citation extraction and display
- Sidebar controls for new chats and example prompts

### rag_components.py
Supporting utilities for source classification, schema setup, and knowledge base organization. Includes:
- Tier-based PDF classification (textbook, survey, research paper)
- Delta table schemas for documents, chunks, and query logs
- Volume scanning and inventory management
- Metadata extraction from PDFs

### requirements.txt
Minimal dependencies:
- `streamlit>=1.28.0` — UI framework
- `databricks-vectorsearch>=0.22` — Vector search client
- `mlflow>=2.9.0` — Model serving client

### app.yaml
Databricks Apps configuration for managed deployment.

---

## 🎯 Use Cases

- **Students** learning computer vision concepts
- **Researchers** exploring foundational knowledge in image processing
- **Engineers** implementing vision algorithms
- **Educators** preparing course materials on CVIP topics

---

## ⚙️ Configuration

The app uses environment variables for configuration:

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABRICKS_HOST` | Databricks workspace URL | `https://dbc-9d1ce33e-6acf.cloud.databricks.com` |
| `DATABRICKS_TOKEN` | Personal access token | *(required)* |

Vector Search and LLM endpoints are hardcoded in `app.py`:

| Component | Value |
|-----------|-------|
| Vector Search Endpoint | `cvip_endpoint` |
| Vector Search Index | `workspace.default.cvip_chunks_vs_index` |
| LLM Model Endpoint | `databricks-meta-llama-3-3-70b-instruct` |

To customize, edit the corresponding lines in `app.py`.

---

## 📊 Performance

| Metric | Value |
|--------|-------|
| Knowledge Base Size | ~10,000 chunks |
| Top-K Retrieval | 5 chunks |
| Average Latency | 4–6 seconds (domain queries) |
| Memory Recall Latency | <5ms |
| Embedding Model | BGE-Large |
| LLM Temperature | 0.1 (low randomness, focused answers) |
| Max Tokens per Answer | 800 |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **UI** | Streamlit |
| **Retrieval** | Databricks Vector Search |
| **Embeddings** | BGE-Large |
| **Generation** | LLaMA 3.3 70B Instruct |
| **Storage** | Databricks Delta Lake |
| **Language** | Python 3.10+ |

---

## 🔐 Security & Best Practices

- Store `DATABRICKS_TOKEN` in environment variables or a secrets manager — never commit it
- Use a strong, time-limited personal access token
- Deploy in a secure Databricks workspace with appropriate network policies
- Monitor query logs in `cvip_query_logs` table for usage analytics

---

## 🚧 Known Limitations

- No built-in user authentication (rely on Databricks workspace policies)
- No offline mode — requires active Databricks connectivity
- No custom reranking beyond Vector Search defaults
- Knowledge base is static (updates require re-indexing)
- Single-session chat history (not persisted across sessions)

---

## 📖 Example Queries

Try asking:

- "What is edge detection?"
- "How does a convolutional neural network work?"
- "Explain the difference between image segmentation and classification"
- "What are vision transformers?"
- "How does the Sobel operator detect edges?"

---

## 👨‍💻 Author

**Dhanush Kumar**  
*Computer Vision & Image Processing | Final Year Project | 2026–2027*

---

## 📄 License

This project is provided as-is for educational and research purposes.

---

## 🤝 Contributing

Found a bug or have a suggestion? Open an issue or reach out directly.

---

## 📚 Learn More

- [Databricks Vector Search Documentation](https://docs.databricks.com/en/generative-ai/vector-search.html)
- [MLflow Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html)
- [Streamlit Documentation](https://docs.streamlit.io)

---

*Built with ❤️ using Databricks, LLaMA, and open-source tools.*
