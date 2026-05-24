# RAG Platform — Phase 0 Handoff Document
> Paste this document at the start of each new conversation to restore full project context.
> Update the **Current Status** and **Completed Phases** sections as you progress.

---

## Project Overview

An enterprise-grade, fully local RAG (Retrieval-Augmented Generation) platform for internal
company knowledge base Q&A. Multi-user environment targeting large enterprises.
Built as a learning project with CV impact in mind — every tool chosen appears in AI Engineer job ads.

**Two developers** working simultaneously from different computers on the same LAN.

---

## Team & Environment

| Item | Detail |
|---|---|
| Developer 1 | Project lead — Windows 11 Pro laptop, PyCharm Professional |
| Developer 2 | Colleague — connects via SSH from their machine |
| Dev server | Hyper-V Ubuntu 24.04 Desktop VM on Developer 1's laptop |
| VM location | E:\Hyper-V\ on host Windows machine |
| VM static IP | 192.168.1.10/24 |
| VM hostname | rag-dev-server |
| Project root | /home/ai_test_user/rag_platform/ |
| VM username | ai_test_user |
| Network | Same LAN — SSH + NoMachine remote desktop |
| Gateway | 192.168.1.1 (modem/router) |
| IDE | PyCharm Professional — SSH remote interpreter pointing to VM |
| Python | 3.12 (system default on Ubuntu 24.04, no PPA needed) |
| OS | Ubuntu 24.04 Desktop LTS |

---

## Locked Stack

### Backend

| Tool | Version | Role |
|---|---|---|
| Python | 3.12.3 | Runtime |
| FastAPI | Latest | API layer + JWT auth + BackgroundTasks |
| LangChain | Latest | RAG orchestration |
| Pydantic v2 | Latest | Data validation + settings |
| SQLAlchemy | 2.x | ORM for PostgreSQL |
| Alembic | Latest | Database migrations |
| python-jose | Latest | JWT token handling |
| passlib | Latest | Password hashing |

### AI & ML

| Tool | Version | Role |
|---|---|---|
| Ollama | Latest | Local LLM inference (no cloud) |
| llama3.2:3b | — | Generation model (fast, lightweight) |
| nomic-embed-text | — | Embedding model |
| LangChain | Latest | RAG pipeline orchestration |

### Databases & Storage

| Tool | Version | Role |
|---|---|---|
| PostgreSQL | 18 | Relational data, users, document metadata |
| Qdrant | Latest | Vector database (purpose-built, CV-relevant) |
| Elasticsearch | 8 | Full-text BM25 hybrid search |
| Redis | 7 | Response cache + session store |

### Frontend

| Tool | Version | Role |
|---|---|---|
| Next.js | 14 | Frontend framework (App Router) |
| TypeScript | 5.x | Type safety (default with Next.js) |
| Tailwind CSS | 3.x | Utility-first styling |
| shadcn/ui | Latest | UI component library |
| zustand | Latest | Client state management |
| react-query | Latest | Server state + API caching |
| react-markdown | Latest | Render LLM markdown responses |
| Node.js | 20 LTS | Next.js runtime (Docker container) |

### Infrastructure

| Tool | Version | Role |
|---|---|---|
| Docker | Latest | Containerization |
| Docker Compose | v2 | Multi-container orchestration (Phases 1-8) |
| Nginx | Alpine | Reverse proxy + static file serving |
| k3s | Latest | Lightweight Kubernetes (Phase 9+) |
| Helm | Latest | Kubernetes package manager (Phase 10+) |

### Observability & Testing

| Tool | Version | Role |
|---|---|---|
| Phoenix (Arize) | Latest | Local LLM observability + tracing (Docker) |
| pytest | Latest | Backend test framework |
| pytest-asyncio | Latest | Async FastAPI endpoint testing |
| httpx | Latest | HTTP test client for FastAPI |
| pytest-cov | Latest | Test coverage reports |
| factory-boy | Latest | Test data generation |
| Jest | Latest | Frontend unit testing (built into Next.js) |
| React Testing Library | Latest | Frontend component testing |
| Playwright | Latest | End-to-end browser testing |
| Postman | Latest | Manual API testing + collections |
| Newman | Latest | Run Postman collections in CI/CD |

### CI/CD

| Tool | Role |
|---|---|
| GitHub | Remote repository hosting |
| GitHub Actions | CI/CD pipeline |
| Self-hosted runner | Runs Actions workflows on the Ubuntu VM locally |

---

## Docker Compose Services (Phase 1 target)

```
services:
  api          → FastAPI app (port 8000)
  frontend     → Next.js Node server (port 3000)
  postgres     → PostgreSQL 18 (port 5432)
  qdrant       → Qdrant vector DB (port 6333)
  elasticsearch→ Elasticsearch 8 (port 9200)
  redis        → Redis 7 (port 6379)
  ollama       → Ollama LLM server (port 11434)
  nginx        → Reverse proxy (port 80)
  phoenix      → Arize Phoenix observability (port 6006)
```

---

## Project Structure (target)

```
rag-platform/
├── backend/
│   ├── app/
│   │   ├── api/           ← FastAPI routers
│   │   ├── core/          ← config, security, dependencies
│   │   ├── models/        ← SQLAlchemy models
│   │   ├── schemas/       ← Pydantic schemas
│   │   ├── services/      ← business logic, RAG pipeline
│   │   └── main.py
│   ├── tests/
│   ├── alembic/
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── app/               ← Next.js App Router pages
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (main)/
│   │   │   ├── dashboard/
│   │   │   ├── chat/
│   │   │   ├── documents/
│   │   │   ├── settings/
│   │   │   └── admin/
│   │   └── layout.tsx
│   ├── components/
│   ├── lib/
│   ├── Dockerfile
│   └── package.json
├── nginx/
│   └── nginx.conf
├── docker-compose.yml
├── docker-compose.override.yml
├── .env.example
├── .github/
│   └── workflows/
│       └── ci.yml
└── README.md
```

---

## Environments

Two isolated Docker Compose stacks run simultaneously on the VM:

| Service | Dev port | Prod port |
|---|---|---|
| Nginx | 8080 | 80 |
| FastAPI | 8000 | 8001 |
| Next.js | 3000 | 3001 |
| PostgreSQL | 5432 | 5433 |
| Qdrant | 6333 | 6334 |
| Elasticsearch | 9200 | 9201 |
| Redis | 6379 | 6380 |
| Ollama | 11434 | 11435 |
| Phoenix | 6006 | 6007 |

```
VM filesystem:
/home/ai_test_user/rag_platform/
├── dev/                                     ← DEV environment
│   ├── docker-compose.yml
│   ├── .env.dev                              ← gitignored
│   └── .venv-dev/                           ← Python 3.12 virtualenv
├── prod/                                    ← PROD environment
│   ├── docker-compose.yml
│   ├── .env.prod                             ← gitignored
│   └── .venv-prod/                          ← Python 3.12 virtualenv
└── repos/                                   ← bare git repo
    └── rag-platform.git/
```

Requirements split:
```
requirements/
├── base.txt    ← shared packages (fastapi, langchain, qdrant-client...)
├── dev.txt     ← base + pytest, ruff, black, httpx, factory-boy, pytest-cov
└── prod.txt    ← base + gunicorn
```

---

## CI/CD Flow

```
Developer (PyCharm SSH → VM dev env)
│
│  1. Write + test code locally in dev environment
│
▼
GitHub (push to feature branch)
│
│  2. GitHub Actions triggers (ci.yml)
│     ├── pytest (backend tests)
│     ├── ruff + black (linting)
│     ├── Playwright (frontend E2E)
│     └── Newman (Postman API tests)
│
▼
Pull Request (feature → develop or develop → main)
│
│  3. Code review — tests must pass before merge allowed
│     Manual approval by team member (1 click)
│
▼
Merge to main
│
│  4. GitHub Actions triggers (deploy.yml)
│     Self-hosted runner on VM executes:
│     ├── git pull latest main
│     ├── docker compose build (prod stack)
│     ├── docker compose up -d (prod stack)
│     ├── alembic upgrade head (DB migrations)
│     └── health check all services
│
▼
Production environment updated ✅
```

GitHub Actions workflow files:
```
.github/workflows/
├── ci.yml        ← runs on every push + PR (tests + linting)
└── deploy.yml    ← runs only on merge to main (prod deploy)
```

---

## Git Repository Setup

```
GitHub remote:     github.com/<org>/rag-platform
Project root:      /home/ai_test_user/rag_platform/
Bare repo on VM:   /home/ai_test_user/rag_platform/repos/rag-platform.git
Dev clone:         /home/ai_test_user/rag_platform/dev/
Prod clone:        /home/ai_test_user/rag_platform/prod/
```

Branch strategy:
```
main        ← production, protected, never commit directly
develop     ← shared dev integration branch
feature/xxx ← individual feature branches (short-lived)
fix/xxx     ← bug fix branches
```

Merge flow:
```
feature/xxx → develop    (daily work, PR required)
develop     → main       (releases to production, PR required)
```

Simultaneous SSH note: Both developers connect to the VM at the same time
via SSH — fully supported. PyCharm SSH remote interpreter is shared.
Coordinate via Git branches to avoid editing the same file simultaneously.

---

## Frontend Pages

```
/login              ← Login page (JWT auth form)
/register           ← User registration
/dashboard          ← Home, usage stats, recent chats
/chat               ← Main RAG chat interface
/chat/[id]          ← Individual chat session
/documents          ← Upload + manage knowledge base docs
/settings           ← User profile + preferences
/admin              ← User management (admin role only)
```

---

## Phase Plan

```
Phase 0:  Planning + VM setup                    ← CURRENT
Phase 1:  Project scaffold + Docker Compose + CI/CD
Phase 2:  Document ingestion pipeline
Phase 3:  Retrieval + RAG orchestrator
Phase 4:  Ollama model management
Phase 5:  Hybrid search + reranking
Phase 6:  JWT Auth + user management
Phase 7:  Next.js frontend
Phase 8:  Observability with Phoenix (Arize)
Phase 9:  Kubernetes migration with k3s
Phase 10: Helm charts for all services
Phase 11: Advanced — Celery async tasks + enterprise hardening
```

### Future considerations (not in scope yet)
- Keycloak — enterprise SSO + RBAC (replace JWT auth in Phase 6)
- Celery — async task queue (replace FastAPI BackgroundTasks)
- Multi-node Kubernetes cluster

---

## Decisions Log

| Decision | Choice | Reason |
|---|---|---|
| PostgreSQL version | 18 | Latest stable (Sep 2025), async I/O, native UUIDv7 support |
| Vector DB | Qdrant (not pgvector) | Purpose-built, CV-relevant, job market demand |
| Auth | FastAPI JWT (not Keycloak) | Simpler for learning, Keycloak deferred to Phase 11 |
| Async tasks | FastAPI BackgroundTasks (not Celery) | Simpler, Celery deferred to Phase 11 |
| Frontend | Next.js 14 (not plain React) | Superset of React, better CV impact, streaming LLM support |
| LLM serving | Ollama | 100% local, no cloud API keys needed |
| Observability | Phoenix/Arize | 100% local Docker, no cloud dependency |
| CI/CD | GitHub + self-hosted runner | Industry standard, runner executes locally on VM |
| Deployment | Docker Compose → k3s | Compose for Phases 1-8, Kubernetes learning in Phase 9+ |
| Dev environment | Hyper-V Ubuntu VM | Shared LAN access for both developers via SSH |
| Git strategy | Bare repo on VM + GitHub | Local collaboration + cloud backup + CI/CD |

---

## Current Status

### Phase 0 — VM Setup Progress

- [x] Hyper-V enabled on Windows 11 Pro (E: drive)
- [x] Ubuntu 24.04 Desktop ISO downloaded
- [x] VM created in Hyper-V
- [x] Ubuntu 24.04 Desktop installed
- [x] Static IP configured on VM (192.168.1.10/24, gateway 192.168.1.1)
- [x] SSH server verified and accessible
- [x] SSH keys set up for both developers
- [x] Project folders created (/home/ai_test_user/rag_platform/{dev,prod,repos})
- [x] Python virtualenvs created (.venv-dev, .venv-prod) + pip upgraded
- [x] PyCharm Pro SSH remote interpreter configured (192.168.1.10, ai_test_user, key-based)
- [x] Python 3.12.3 confirmed (system default, no PPA needed)
- [ ] Docker + Docker Compose installed
- [ ] Ollama installed + models pulled (llama3.2:3b, nomic-embed-text)
- [ ] Git installed + bare repo created
- [ ] GitHub repo created + self-hosted runner registered
- [x] VM IP confirmed: 192.168.1.10

### Completed Phases
- None yet

---

## Key Commands Reference

```bash
# Connect to VM from Windows
ssh ai_test_user@192.168.1.10

# Start all services
cd ~/rag_platform/dev && docker compose up -d

# View logs
docker compose logs -f api

# Run backend tests
cd backend && pytest --cov=app tests/

# Pull latest Ollama models
ollama pull llama3.2:3b
ollama pull nomic-embed-text

# Check Ollama running
curl http://localhost:11434/api/tags
```