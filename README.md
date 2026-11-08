# Argus

### Autonomous Review for GitHub Understanding & Security

Argus is an AI-assisted code review and repository documentation platform. It combines a FastAPI backend, a Next.js dashboard, specialized review agents, and interactive visualizations to help developers understand and improve a codebase.

## Features

- **Multi-agent code review:** Six focused agents analyze code for bugs, security issues, performance, readability, architecture, and accessibility.
- **LangGraph review workflow:** Routes and coordinates review work, including parallel agent execution.
- **Documentation generation:** Creates a repository README, modular documentation, onboarding guidance, and code docstrings.
- **Interactive codebase graphs:** Explores dependency, execution-flow, and knowledge graphs in the dashboard.
- **Code explanations:** Presents file summaries and review findings in a dashboard designed to make results easier to understand.
- **GitHub integration:** Supports repository ingestion, optional README publishing, and pull-request review webhooks.
- **Persona-aware results:** Tailors review explanations for an intern, student, frontend developer, or backend developer.

## Technology

- **Frontend:** Next.js 14, React 18, TypeScript, GSAP, and React Flow
- **Backend:** Python, FastAPI, and Uvicorn
- **Review orchestration:** LangGraph
- **AI provider:** NVIDIA NIM-compatible chat API
- **Job state:** Redis when configured, with an in-process fallback
- **Repository analysis:** Python AST parsing, code-file discovery, and an in-memory RAG pipeline

## How it works

1. Submit a public GitHub repository URL or upload a ZIP from the dashboard.
2. The backend creates a temporary workspace, discovers and parses source files, and prepares repository context.
3. For a review, LangGraph coordinates the specialized agents and produces findings, summaries, and file explanations.
4. For documentation, the backend generates repository documentation and graph data.
5. The dashboard polls the job status and displays the results, including interactive graph views.

## Repository layout

```text
agents/                     Specialized code-review agents and orchestrator
backend/
  main.py                   FastAPI app and API routes
  models/                   API request/response schemas
  services/                 Review, LangGraph, documentation, and ingestion services
  utils/                    Backend configuration
docs/                       Source parsing and graph/documentation builders
frontend/
  app/                      Next.js pages, landing page, and dashboard
  components/               Review, documentation, and code-explainer UI
  src/components/           Interactive graph and tree views
github/                     GitHub webhook, diff, and comment integrations
rag/                        Retrieval and in-memory vector-store utilities
data/workspaces/            Temporary repository workspaces
```

## Requirements

- Python 3.10 or later
- Node.js 18 or later and npm
- An NVIDIA NIM API key for AI-powered review and documentation
- Redis is optional for local development; without it, job state uses an in-process fallback

## Run locally

### 1. Configure the backend

From the repository root, create a local environment file:

```powershell
Copy-Item .env.example .env
```

Set `NIM_API_KEY` in `.env`. GitHub tokens and the webhook secret are optional and only needed for their corresponding integrations. Do not commit `.env` or paste real credentials into source files.

Install dependencies and start the API:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r backend\requirements.txt
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

The API is available at `http://localhost:8000`; interactive API documentation is at `http://localhost:8000/docs`.

### 2. Configure and run the frontend

In another terminal:

```powershell
Copy-Item frontend\.env.example frontend\.env.local
cd frontend
npm install
npm run dev
```

Open `http://localhost:3000`. If the backend runs at a different address, set `NEXT_PUBLIC_API_BASE_URL` in `frontend/.env.local` to that API base URL.

## API overview

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Check API health |
| `POST` | `/api/review/repo` | Start a review for a repository URL |
| `POST` | `/api/review/upload` | Start a review from an uploaded ZIP |
| `POST` | `/api/docs/repo` | Generate documentation for a repository URL |
| `POST` | `/api/docs/upload` | Generate documentation from an uploaded ZIP |
| `POST` | `/api/github/verify-docs-token` | Verify a token for publishing generated documentation |
| `POST` | `/api/github/webhook` | Receive configured GitHub webhook events |
| `GET` | `/api/jobs/{job_id}` | Poll a background job until it completes |

Review and documentation requests run as background jobs. Submit a request, then poll the returned `job_id` at `/api/jobs/{job_id}` until its status is `done` or `error`.

## Configuration

The root `.env.example` documents backend settings, including NVIDIA NIM models and request limits, Redis, CORS, GitHub integration tokens, and job timeouts. `frontend/.env.example` documents `NEXT_PUBLIC_API_BASE_URL`.

Only configure the credentials needed for the features you use:

- `NIM_API_KEY` enables AI-backed review and documentation.
- `REDIS_URL` configures Redis-backed job status; local development can use the in-process fallback.
- `GITHUB_REVIEW_TOKEN` and `GITHUB_WEBHOOK_SECRET` are used for pull-request review integration.
- `GITHUB_DOCS_TOKEN` or a dashboard-provided token enables publishing generated README files.

## My contributions

- Built the specialized code-review agents and connected their findings to the review workflow.
- Added LangGraph orchestration for coordinating review tasks and parallel agent execution.
- Built the interactive dashboard for submitting work and exploring review and documentation results.
- Added interactive codebase graph visualizations, including dependency, execution-flow, and knowledge graphs.
- Integrated the dashboard with the backend review, documentation, and job-status APIs.

## Development checks

```powershell
# Frontend production build
cd frontend
npm run build

# Backend tests, if present or added
cd ..
python -m pytest
```

## License

No `LICENSE` file is currently included. Although the landing page mentions MIT, include the actual license text in the repository before distributing it under that license.
