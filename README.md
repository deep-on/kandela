<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>Persistent semantic memory for AI coding agents.</b><br>
  Your AI remembers decisions, failures, and context — across sessions, projects, and tools.
</p>

<p align="center">
  <b>English</b> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-0.6.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/license-AGPL--3.0-blue" alt="License">
  <img src="https://img.shields.io/badge/python-3.11+-green" alt="Python">
  <img src="https://img.shields.io/badge/docker-compose-blue" alt="Docker">
  <img src="https://img.shields.io/badge/MCP_tools-13-brightgreen" alt="MCP Tools">
  <img src="https://img.shields.io/badge/ChromaDB-vector_store-orange" alt="ChromaDB">
  <img src="https://img.shields.io/badge/embeddings-50+_languages-purple" alt="Multilingual">
</p>

<p align="center">
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ Watch demo on YouTube</a>
</p>

---

## Problem

Google, Claude, and ChatGPT each have their own memory — but it only works within their ecosystem.
Switch from GPT to Claude, and all your memories are gone. You can't share context across tools or teammates.

Kandela solves this with **cross-provider memory + full transparency + a 4-stage RAG pipeline**.

## Key Features

- **13 MCP Tools** — Store, search, delete, update, auto-recall, on-demand search, inbox, project management
- **Hybrid Search** — Semantic + BM25 keyword search (RRF fusion)
- **Importance Engine** — Auto-scoring 1~10 + 18 rule-based infra tagging
- **Lazy Retrieval** — Brief mode (~260 tokens) + context_search on-demand
- **Session Continuity** — Detects environment changes (CWD, host, client) + auto-includes infra memories
- **Prompt Guard** — Prevents bad decisions based on stale memories
- **Circuit Breaker** — Detects repeated failure patterns + auto-stores gotchas
- **Web Dashboard** — Per-project memory browser, search, stats, performance monitoring
- **One-click Install** — Auto-installs hooks + 16 slash commands
- **Multilingual Embeddings** — paraphrase-multilingual-MiniLM-L12-v2 (50+ languages)

## Benchmark

HIPAA medical data pipeline scenario (8 sessions, 14 decision traps) — Kandela ON vs OFF:

| | Kandela ON | Kandela OFF | Delta |
|---|:-:|:-:|:-:|
| **Decision Trap Avoidance** | **100%** | 11.9% | **+88.1pp** |
| **Task Time** | 77.9 min | 86.6 min | **-10.1%** |
| **Generated Code** | 2,152 lines | 3,441 lines | **-37.5%** |
| **Generated Files** | 40 | 62 | **-35.5%** |

> 3 runs (seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### Key Insights

- **Decisions not in code are what matters**: Auditor names, OOM incidents, data loss history — information invisible to code reading
- **Code-based decisions are self-defended by LLM**: In code-centric scenarios (InfraBot), LLM achieved 95.2% avoidance even without Kandela
- **Eliminates rework**: Without Kandela, AI re-implements previously rejected approaches — 37.5% code waste

### Large-Scale Data Analysis

830K real conversations (WildChat-1M) and 1M LLM conversations (LMSYS-Chat-1M):

| Finding | Metric | Implication |
|---------|:------:|-------------|
| Initial instruction reference drops 18%p after 10 turns | 66%→48% | General LLMs start forgetting early context |
| Small LLMs drop up to 66%p | chatglm 84%→19% | Memory augmentation critical for local/small models |
| 14% of user corrections are "forgot previous conversation" | 82K coding conversations | Core problem Kandela solves |
| Kandela maintains context after 49+ turns | Internal benchmark | Brief Recall prevents decay |

## Architecture

```
Client (Claude Code / Desktop / Cursor / Telegram Bot)
  │
  ├─ MCP Protocol ──→ Kandela Server
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── Hybrid Search (Semantic + BM25 RRF)
  │                      ├── Importance Engine (18 rules)
  │                      ├── Session Continuity
  │                      └── Prompt Guard / Circuit Breaker
  │
  ├─ Telegram Bot ──→ Natural language memory access (LLM intent classification)
  │
  └─ Web Dashboard ──→ REST API + Memory Browser
```

### Memory Types

| Type | Description | Auto/Manual |
|------|-------------|:-----------:|
| `fact` | Preferences, tech stack, environment info | Both |
| `decision` | Design decisions, trade-offs, rationale | Manual |
| `summary` | Session summaries at conversation end | Auto |
| `snippet` | Frequently used code patterns, configs | Manual |

### Search Pipeline

```
Query → Semantic (MiniLM-L12) ──────┐
                                     ├─→ RRF Fusion → Importance Rerank → MMR Diversity → Results
Query → BM25 (multilingual NLP) ────┘
        ├── Korean: kiwipiepy morphological analysis
        └── Others: regex tokenizer (English, German, Spanish, etc.)
```

## Get Started

### Hosted Service (Recommended)

Request beta access, then install in 2 minutes.

```bash
# 1. Email support@kandela.ai for beta invite code
# 2. Sign up at https://api.kandela.ai/dashboard with the invite code
# 3. Generate API key in Account page
# 4. One-line install (prompts for API key, sets up everything):
curl -fsSL https://api.kandela.ai/api/install | bash
```

→ [Getting Started Guide](https://kandela.ai/getting-started)

### Self-Hosted

Run your own Kandela instance. Single-user mode, full control over your data.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [Self-Host Repository](https://github.com/deep-on/kandela-selfhost)

## Hosted vs Self-Hosted

| | Hosted (api.kandela.ai) | Self-Hosted |
|---|:-:|:-:|
| Setup | 2 min | 5 min |
| Multi-user | ✅ | Single-user |
| Telegram Bot | ✅ | — |
| Remote Commands | ✅ | — |
| Activity Heatmap | ✅ | — |
| Tier Features (Pro/Max) | ✅ | All features |
| Data Location | Cloud | Your server |
| Maintenance | Managed | Self-managed |

## Tech Stack

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — vector database (persistent)
- **sentence-transformers** — local embeddings (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — input validation
- **Docker** — deployment

## Links

- **Homepage**: [kandela.ai](https://kandela.ai)
- **Self-Host Repo**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **Privacy Policy**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **Terms of Service**: [kandela.ai/terms](https://kandela.ai/terms)

## License

- **Server**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **Client-side files** (hooks, slash commands): [MIT](LICENSE-CLIENT)

## Disclaimer

This software is provided "AS IS" without warranty of any kind.
Users are responsible for backing up their own data.
