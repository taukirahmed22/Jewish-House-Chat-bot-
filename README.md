# JH RAG Assistant

A retrieval-augmented question-answering assistant built on n8n, Qdrant, Ollama and
Google Gemini (via OpenRouter). The assistant answers from a curated set of approved
answers rather than free-form generation: a user question is embedded locally, matched
against a vector store of approved content, and used to ground the model's reply.

## Architecture

```
User ──► Chat Trigger ──► AI Agent ──► Gemini (OpenRouter)   [answer generation]
                               ├──────► Session Memory        [conversation context]
                               ├──────► Approved Answers (Qdrant)  [retrieval: jh_qa]
                               └──────► Ollama Embeddings      [nomic-embed-text, 768-d]
```

- **Orchestration:** n8n workflow `JH RAG Assistant (semantic + Gemini)` (6 nodes).
- **Embeddings:** Ollama `nomic-embed-text` (local, 768-dimension vectors).
- **Vector store:** Qdrant collection `jh_qa` (cosine similarity).
- **Answer model:** `google/gemini-*` via the OpenRouter API.
- **Memory:** windowed session buffer for multi-turn context.

## Repository Contents

| Path | Description |
|------|-------------|
| `JH_RAG_Assistant.workflow.json` | The exported n8n workflow (active version). |
| `docs/ollama_list.txt` | Installed Ollama models. |
| `docs/docker_ps.txt` | Running containers (n8n + Qdrant). |
| `docs/qdrant_jh_qa.txt` | Qdrant `jh_qa` collection stats. |

## Environment

- **Containers:** `n8n-n8n-1` (n8n), `jh-qdrant` (Qdrant) — see `docs/docker_ps.txt`.
- **Models:** see `docs/ollama_list.txt`.
- **Vector store:** `jh_qa` — 37 points, 768-dimension, cosine distance.

## Testing

- n8n execution history: **62 runs, 62 successful (100%)** at time of export.
- Each run exercises the full pipeline: embed → retrieve from `jh_qa` → generate.

## Setup (to reproduce)

1. Run Qdrant and n8n (Docker). Run Ollama and pull `nomic-embed-text`.
2. Create the Qdrant collection `jh_qa` (768-d, cosine) and load approved answers.
3. In n8n, **Import from File** → `JH_RAG_Assistant.workflow.json`.
4. Set credentials in n8n's credential store (OpenRouter API key; Qdrant/Ollama hosts).
   Credentials are **not** included in this repo.
5. Activate the workflow and open the chat trigger URL.

## Notes / Limitations

- Credentials live in n8n's encrypted credential store and are intentionally excluded here.
- The approved-answer corpus is small (37 entries) and is the main lever for coverage.
