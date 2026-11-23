# IT Support Copilot

A portfolio project for an IT help desk assistant. It searches a private knowledge base first, checks whether the retrieved material supports an answer, and uses web search when the internal documents are not enough. The interface shows the source and the agent's decision trace.

This is a customized project based on [Enterprise IT Support Agentic RAG Copilot](https://github.com/entbappy/Enterprise-IT-Support-Agentic-RAG-Copilot). I adapted the branding, setup and deployment configuration, and safety defaults for this portfolio build. The upstream Apache 2.0 license is retained.

## What it does

- Routes greetings directly and IT questions through a LangGraph workflow.
- Retrieves company documents from Pinecone and grades the evidence before answering.
- Falls back to Tavily web search when private evidence is weak, then retries once with a rewritten query.
- Returns citations and a visible decision trace in a responsive chat UI.
- Accepts PDF, TXT, Markdown, and DOCX documents through a protected ingestion endpoint.
- Writes a basic SQLite audit record for each chat request.

## Stack

FastAPI, LangGraph, OpenAI chat and embeddings, Pinecone, Tavily, SQLite, HTML/CSS/JavaScript, and Docker.

## Run locally

Requirements: Python 3.11+ and API credentials for OpenAI, Pinecone, and Tavily. Pinecone is used for the private knowledge base; OpenAI provides both chat and embeddings.

1. Create and activate a virtual environment.

   ```powershell
   py -3.11 -m venv .venv
   .\.venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

2. Copy `.env.example` to `.env` and fill in your own credentials. Keep `.env` private; it is ignored by Git.

3. Add your own company IT documents under `data/sample_kb/`, then index them:

   ```powershell
   python ingest_sample_kb.py
   ```

   The first indexing run creates the configured Pinecone index if it does not exist. Check the index dimension against `EMBEDDING_MODEL` before reusing an existing index.

4. Start the app:

   ```powershell
   python run.py
   ```

   Open `http://127.0.0.1:8080`.

## Environment variables

| Variable | Purpose |
|---|---|
| `OPENAI_API_KEY` | Chat completions and text embeddings |
| `PINECONE_API_KEY` | Private vector index |
| `TAVILY_API_KEY` | Public web fallback |
| `ADMIN_API_KEY` | Required to upload/index documents through `/api/ingest` |
| `PINECONE_INDEX_NAME` | Pinecone index name |
| `PINECONE_NAMESPACE` | Knowledge base namespace |
| `OPENAI_MODEL` | Chat model (default `gpt-4o-mini`) |
| `EMBEDDING_MODEL` | Embedding model (default `text-embedding-3-small`) |
| `TOP_K` | Number of private KB chunks retrieved |
| `MAX_RETRIES` | Query rewrite retries after weak web evidence |

## Deploy to Render

The repository includes `render.yaml` and a Dockerfile. After pushing this repository to GitHub, create a **Blueprint** in Render and connect the repository. Add `OPENAI_API_KEY`, `PINECONE_API_KEY`, `TAVILY_API_KEY`, and a long random `ADMIN_API_KEY` in the Render service environment, then deploy. The service health check uses `/api/health`.

Render's default filesystem is temporary. The Pinecone knowledge base persists independently, but uploaded files and the SQLite audit database can be lost when the service restarts or redeploys. Use a persistent disk or managed database if those records must persist. See [Render filesystem guidance](https://render.com/docs/deploys#ephemeral-filesystem).

## API

- `GET /api/health` — service health
- `POST /api/chat` — JSON body: `{"question":"How do I connect to VPN?"}`
- `POST /api/ingest` — multipart file upload with the `X-Admin-Key` header

## Project layout

```text
app/                 FastAPI routes, configuration, RAG workflow and services
data/sample_kb/      Example knowledge documents; replace with your own
static/              Browser UI styles and behavior
templates/            Chat page
render.yaml           Render Blueprint configuration
Dockerfile            Container build and start command
```
