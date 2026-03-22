# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepWiki-Open is an AI-powered wiki generator that creates documentation for GitHub/GitLab/Bitbucket repositories. It clones repos, analyzes code, generates embeddings, and produces structured wiki pages with Mermaid diagrams and RAG-based Q&A.

## Architecture

**Two-service architecture:**
- **Backend** (`api/`): Python FastAPI server on port 8001. Handles repo cloning, code analysis, embedding generation, wiki generation, and RAG-based chat.
- **Frontend** (`src/`): Next.js 15 (App Router) + React 19 + TypeScript on port 3000. Renders wiki pages, Mermaid diagrams, and chat interface.

**Key backend modules:**
- `api/main.py` — Uvicorn entry point
- `api/api.py` — FastAPI route definitions
- `api/data_pipeline.py` — Code analysis and embedding pipeline
- `api/rag.py` — RAG implementation with FAISSRetriever and custom Memory class
- `api/websocket_wiki.py` — WebSocket handler for streaming wiki generation
- `api/config.py` — Configuration management with env var interpolation (`${ENV_VAR}`)
- `api/tools/embedder.py` — Embedder abstraction layer across providers

**Multi-provider LLM system:** Each provider (Google, OpenAI, OpenRouter, Ollama, Bedrock, Azure, DashScope) has its own client module (`api/*_client.py`). Provider/model config lives in `api/config/generator.json`, embedding config in `api/config/embedder.json`.

**Frontend key paths:**
- `src/app/[owner]/[repo]/` — Dynamic wiki page routes
- `src/components/` — UI components (Ask.tsx for Q&A, Mermaid.tsx for diagrams, WikiTreeView.tsx for navigation)
- `src/utils/websocketClient.ts` — WebSocket client for streaming

**Data storage:** All persistent data goes to `~/.adalflow/` (repos, embeddings/databases, wikicache).

## Common Commands

### Development
```bash
# Backend
poetry install -C api
python -m api.main                    # Start API server on :8001

# Frontend
yarn install
yarn dev                              # Start dev server on :3000 (Turbopack)
yarn build                            # Production build
yarn lint                             # ESLint
```

### Testing
```bash
python tests/run_tests.py             # All tests
python tests/run_tests.py --unit      # Unit tests only
python tests/run_tests.py --integration  # Integration tests (need API keys)
python tests/run_tests.py --api       # API tests only
```

Pytest config is in `pytest.ini`. Markers: `unit`, `integration`, `slow`, `network`.

### Docker
```bash
docker-compose up                     # Run full stack (ports 3000 + 8001)
```

## Configuration

- `api/config/generator.json` — LLM providers and models
- `api/config/embedder.json` — Embedding models, chunking (350 words, 100 overlap), retriever (top_k: 20)
- `api/config/repo.json` — File/directory filters for repo processing
- `api/config/lang.json` — Language configuration

Key env vars: `GOOGLE_API_KEY`, `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `OLLAMA_HOST`, `DEEPWIKI_EMBEDDER_TYPE` (openai|google|ollama|bedrock), `DEEPWIKI_CONFIG_DIR`.

## Tech Stack Notes

- **Package manager:** Yarn 1.22 for frontend, Poetry 2.0 for backend
- **Python:** 3.11 required
- **Node:** >=18
- **TypeScript path alias:** `@/*` → `./src/*`
- Frontend uses WebSocket (not HTTP streaming) for real-time wiki generation
