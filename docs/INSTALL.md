# Install

Five ways to run MiroShark. Pick one.

| Path | GPU? | Best for |
|---|---|---|
| [Railway / Render](#one-click-cloud) | No | Fastest path to a live deployment |
| [`./miroshark`](#quick-start-miroshark-launcher) | Optional | Local dev, lowest friction |
| [Cloud API (manual)](#option-a-cloud-api-no-gpu) | No | Local Neo4j + cloud LLM |
| [Docker + Ollama](#option-b-docker--local-ollama) | Yes | Fully self-hosted, one command |
| [Manual + Ollama](#option-c-manual--local-ollama) | Yes | Fully self-hosted, manual control |
| [Claude Code CLI](#option-d-claude-code-no-api-key) | No | Uses your Claude Pro/Max subscription |

## Prerequisites

- An OpenAI-compatible API key (OpenRouter, OpenAI, Anthropic…), Ollama for local inference, **or** Claude Code CLI
- Python 3.11+, Node.js 18+, Neo4j 5.15+

**Installing Neo4j** (pick whichever fits your OS — the launcher detects whichever is running):

- **macOS** — `brew install neo4j && brew services start neo4j`
- **Linux** — `sudo apt install neo4j` *(or your distro's equivalent)*
- **Zero-install** — create a free [Neo4j Aura](https://neo4j.com/cloud/aura-free/) cloud instance and set `NEO4J_URI` / `NEO4J_PASSWORD` in `.env`

After first launch, set the password — MiroShark's default is `miroshark` to match `.env.example`:

```bash
# macOS / Linux native install (one-time)
neo4j-admin dbms set-initial-password miroshark
```

## Hardware

**Local (Ollama):**

| | Minimum | Recommended |
|---|---|---|
| RAM | 16 GB | 32 GB |
| VRAM | 10 GB | 24 GB |
| Disk | 20 GB | 50 GB |

**Cloud mode:** no GPU needed — just Neo4j and an API key. Any 4 GB RAM machine works.

---

## One-click cloud

Deploy to the cloud in under 3 minutes — no local setup required.

**Before you deploy, create:**

1. A free [Neo4j Aura](https://neo4j.com/cloud/aura-free/) instance — grab the `NEO4J_URI` (starts with `neo4j+s://`) and password.
2. An [OpenRouter](https://openrouter.ai/) API key — used for LLM calls and embeddings. Free credits on signup.

### Railway (recommended — persistent storage, free trial)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.app/new/template?template=https://github.com/aaronjmars/MiroShark)

After clicking, set these environment variables in the Railway dashboard:

| Variable | Value |
|---|---|
| `LLM_API_KEY` | Your OpenRouter key (`sk-or-v1-...`) |
| `NEO4J_URI` | Your Aura URI (`neo4j+s://...`) |
| `NEO4J_PASSWORD` | Your Aura password |
| `EMBEDDING_API_KEY` | Same OpenRouter key |
| `OPENAI_API_KEY` | Same OpenRouter key |

### Render (free tier — 750 hrs/month, spins down after 15 min idle)

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/aaronjmars/MiroShark)

Render reads `render.yaml` automatically. Set the same env vars above when prompted.

> Cloud deploys use OpenRouter for all LLM calls — Ollama is not available in this mode. Both platforms expose MiroShark on a public HTTPS URL, no port forwarding needed.

---

## Quick start: `./miroshark` launcher

**The recommended path** — one [OpenRouter](https://openrouter.ai/) key and the launcher. No GPU, no Ollama, no model downloads.

**Prereqs** — Python 3.11+, Node 18+, Docker (for Neo4j), and a free OpenRouter key.

```bash
git clone https://github.com/aaronjmars/MiroShark.git && cd MiroShark
cp .env.example .env
```

Edit `.env` and paste your OpenRouter key into all five slots (Best preset shown — swap `LLM_MODEL_NAME` to `google/gemini-2.0-flash-001` for the Cheap preset):

```bash
LLM_API_KEY=sk-or-v1-YOUR_KEY
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_MODEL_NAME=anthropic/claude-haiku-4.5

SMART_PROVIDER=openai
SMART_API_KEY=sk-or-v1-YOUR_KEY
SMART_BASE_URL=https://openrouter.ai/api/v1
SMART_MODEL_NAME=anthropic/claude-sonnet-4.6

NER_MODEL_NAME=google/gemini-2.0-flash-001
NER_BASE_URL=https://openrouter.ai/api/v1
NER_API_KEY=sk-or-v1-YOUR_KEY

WONDERWALL_MODEL_NAME=google/gemini-2.0-flash-lite-001

OPENAI_API_KEY=sk-or-v1-YOUR_KEY
OPENAI_API_BASE_URL=https://openrouter.ai/api/v1

EMBEDDING_PROVIDER=openai
EMBEDDING_MODEL=openai/text-embedding-3-small
EMBEDDING_BASE_URL=https://openrouter.ai/api
EMBEDDING_API_KEY=sk-or-v1-YOUR_KEY
EMBEDDING_DIMENSIONS=768
```

Then launch:

```bash
./miroshark
```

What the launcher does:

1. Checks Python 3.11+, Node 18+, uv, Neo4j/Docker
2. Starts Neo4j if not already running (Docker or native)
3. Installs frontend + backend deps if missing
4. Kills stale processes on ports 3000/5001
5. Launches Vite dev server (`:3000`) and Flask API (`:5001`)
6. Ctrl+C to stop everything

Open `http://localhost:3000`. First simulation ≈ 15–25 min, ~$1.20 (Cheap) to ~$3.50 (Best). See [Models](MODELS.md) for the full preset benchmark.

> Prefer to run everything local? Skip to [Option B (Docker + Ollama)](#option-b-docker--local-ollama) or [Option C (manual Ollama)](#option-c-manual--local-ollama) below.

---

## Option A: Cloud API (no GPU)

Only Neo4j runs locally. LLM and embeddings use a cloud provider.

```bash
# 1. Start Neo4j (see "Prerequisites → Installing Neo4j" above)
brew install neo4j && brew services start neo4j    # macOS
# sudo apt install neo4j                            # Linux

# 2. Configure
cp .env.example .env
```

Edit `.env` — uncomment the **Cheap** or **Best** preset block in `.env.example` (both pre-written and benchmarked; see [Models](MODELS.md)) and paste in your OpenRouter key. Or set the four model slots directly:

```bash
LLM_API_KEY=sk-or-v1-your-key
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_MODEL_NAME=anthropic/claude-haiku-4.5

SMART_MODEL_NAME=anthropic/claude-sonnet-4.6
NER_MODEL_NAME=google/gemini-2.0-flash-001
WONDERWALL_MODEL_NAME=google/gemini-2.0-flash-lite-001

EMBEDDING_PROVIDER=openai
EMBEDDING_MODEL=openai/text-embedding-3-small
EMBEDDING_BASE_URL=https://openrouter.ai/api
EMBEDDING_API_KEY=sk-or-v1-your-key
EMBEDDING_DIMENSIONS=768
```

```bash
npm run setup:all && npm run dev
```

Open `http://localhost:3000`. Backend API at `http://localhost:5001`.

---

## Option B: Docker — local Ollama

```bash
git clone https://github.com/aaronjmars/MiroShark.git
cd MiroShark
docker compose up -d

# Pull models into Ollama
docker exec miroshark-ollama ollama pull qwen2.5:32b
docker exec miroshark-ollama ollama pull nomic-embed-text
```

Open `http://localhost:3000`.

---

## Option C: Manual — local Ollama

```bash
# 1. Start Neo4j (macOS; for Linux: sudo apt install neo4j)
brew install neo4j && brew services start neo4j

# 2. Start Ollama & pull models
ollama serve &
ollama pull qwen2.5:32b
ollama pull nomic-embed-text

# 3. Configure & run
cp .env.example .env
npm run setup:all
npm run dev
```

See [Models](MODELS.md) for the Ollama context-window override (important — defaults to 4096 tokens but MiroShark needs 10–30k).

---

## Option D: Claude Code (no API key)

Use your Claude Pro/Max subscription as the LLM backend via the local `claude` CLI. No API key or GPU required — just a logged-in installation.

```bash
# 1. Install Claude Code (if not already)
npm install -g @anthropic-ai/claude-code

# 2. Log in (opens browser)
claude

# 3. Start Neo4j (macOS; for Linux: sudo apt install neo4j)
brew install neo4j && brew services start neo4j

# 4. Configure
cp .env.example .env
```

Edit `.env`:

```bash
LLM_PROVIDER=claude-code
# Optional: pick a specific model (default uses your Claude Code default)
# CLAUDE_CODE_MODEL=claude-sonnet-4-20250514
```

You still need embeddings (Claude Code doesn't support them) and a separate LLM for the CAMEL-AI simulation rounds. Use Ollama or a cloud API for both.

```bash
npm run setup:all && npm run dev
```

### What Claude Code covers

When `LLM_PROVIDER=claude-code`, MiroShark services route through Claude Code. The only exception is the CAMEL-AI simulation engine itself, which manages its own LLM connections internally.

| Component | Claude Code | Needs separate LLM |
|---|---|---|
| Graph building (ontology + NER) | Yes | — |
| Agent profile generation | Yes | — |
| Simulation config generation | Yes | — |
| Report generation | Yes | — |
| Persona chat | Yes | — |
| CAMEL-AI simulation rounds | — | Yes (Ollama or cloud) |
| Embeddings | — | Yes (Ollama or cloud) |

> **Performance note:** each LLM call spawns a `claude -p` subprocess (~2-5s overhead). Best for small simulations or hybrid mode — use Ollama/cloud for high-volume simulation rounds, Claude Code for everything else.
