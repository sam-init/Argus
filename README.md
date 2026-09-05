# Repository

## Overview
Provides a backend API for repository analysis workflows; runs automated multi-agent code review passes; generates repository documentation from parsed source; uses retrieval indexing to ground analysis prompts.

Project type: Full-stack web application with an API service and AI-assisted analysis pipeline.

Parsed surface: **66 files** · **197 functions** · **45 classes**

## Architecture / How It Works
- API requests are handled by `backend/main.py` and dispatched to async background jobs.
- Repository input (URL/ZIP) is normalized into a workspace by `backend/services/ingestion.py`.
- Source files are discovered and parsed using `docs/repo_loader.py` and `docs/parser.py`.
- Parsed content is indexed in `rag/rag_pipeline.py` for retrieval-assisted review/docs generation.
- Review findings are produced by multi-agent analysis from `agents/` and orchestrated by `agents/orchestrator.py`.
- Documentation output is assembled in `backend/services/doc_service.py` and includes README, modular docs, and graphs.
- Frontend polling in `frontend/lib/api.ts` reads job progress via `GET /api/jobs/{job_id}`.

## Project Structure
```text
agents/
  - agents/__init__.py
  - agents/accessibility_agent.py
  - agents/architecture_agent.py
  - agents/base_agent.py
  - agents/bug_agent.py
  - agents/orchestrator.py
  - ...
backend/
  - backend/__init__.py
  - backend/main.py
  - backend/models/__init__.py
  - backend/models/schemas.py
  - backend/render.yaml
  - backend/services/__init__.py
  - ...
components/
  - components/GraphView.tsx
  - components/TreeView.tsx
docs/
  - docs/__init__.py
  - docs/graph_builder.py
  - docs/parser.py
  - docs/readme_generator.py
  - docs/repo_loader.py
  - docs/rot_detector.py
frontend/
  - frontend/app/components/FaultyTerminal.tsx
  - frontend/app/dashboard/page.tsx
  - frontend/app/layout.tsx
  - frontend/app/page.tsx
  - frontend/components/CodeExplainer.tsx
  - frontend/components/DocsResults.tsx
  - ...
github/
  - github/__init__.py
  - github/commenter.py
  - github/diff_fetcher.py
  - github/pr_handler.py
  - github/webhook.py
rag/
  - rag/__init__.py
  - rag/chunker.py
  - rag/embedder.py
  - rag/rag_pipeline.py
  - rag/vector_store.py
root/
utils/
  - utils/graphAdapter.ts
```

## Key Components
- **`backend/main.py`**: symbols `health()`, `_job_key()`, `_parse_workspace()`; imports __future__, asyncio, backend.models.schemas, backend.services.doc_service.
- **`docs/readme_generator.py`**: symbols `build_repo_facts()`, `_normalize_repo_path()`, `_looks_like_system_path()`; imports __future__, collections, pathlib, typing.
- **`backend/services/review_service.py`**: symbols `ReviewService`, `__init__()`, `collect_file_symbols()`, `_compact_structure()`; imports __future__, agents.base_agent, agents.orchestrator, asyncio.
- **`backend/services/nim_client.py`**: symbols `CallProfile`, `RatePacer`, `get_nim_client()`, `__init__()`, `_get_lock()`; imports __future__, asyncio, backend.utils.settings, dataclasses.
- **`backend/services/review_routing.py`**: symbols `RepoProfile`, `_ext_of()`, `classify()`, `complexity_score()`; imports __future__, backend.services.review_prompts, backend.utils.settings, dataclasses.
- **`backend/services/doc_service.py`**: symbols `DocumentationService`, `__init__()`, `_generate_docstrings()`, `_build_modular_docs()`; imports __future__, backend.services.nim_client, backend.services.structure_service, backend.utils.settings.
- **`backend/services/ingestion.py`**: symbols `IngestionError`, `_ensure_clean_dir()`, `create_workspace()`, `cleanup_workspace()`; imports __future__, io, logging, pathlib.
- **`backend/models/schemas.py`**: symbols `RepoInput`, `TokenVerifyInput`; imports __future__, pydantic, typing.
- **`frontend/app/dashboard/page.tsx`**: symbols `LoadingSkeleton()`, `EmptyState()`, `DashboardPage()`; imports @/components/DocsResults, @/components/ReviewResults, @/lib/types, @/src/components/GraphView.
- **`backend/services/review_graph.py`**: symbols `ReviewState`, `_build_graph()`, `_service()`, `_select_branch()`; imports __future__, agents.base_agent, asyncio, backend.services.
- **`utils/graphAdapter.ts`**: symbols `normalizePath()`, `uniquePaths()`, `add()`; imports @/lib/types.
- **`docs/graph_builder.py`**: symbols `build_dependency_graph()`, `build_execution_flowchart()`, `build_knowledge_graph()`; imports __future__, typing.

## Technologies Used
- **md**: 3 file(s)
- **py**: 43 file(s)
- **yaml**: 1 file(s)
- **json**: 4 file(s)
- **js**: 1 file(s)
- **ts**: 5 file(s)
- **tsx**: 9 file(s)
- **requests**
- **FastAPI**
- **Redis**
- **React**
- **Next.js**
- **Pydantic**
- **httpx**
- **PyJWT**

## Usage
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --host 0.0.0.0 --port 8000

cd frontend
npm install
npm run dev
```

