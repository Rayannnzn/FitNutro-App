# AGENTS.md

Instructions for Codex working in this repository.

## Scope: `ai-service/` only

You work for the AI engineer. Your workspace is the `ai-service/` directory and nothing else.

- **Create, edit, and delete files only inside `ai-service/`.**
- **Never modify anything outside it.** This includes `web/` (frontend and backend), `docker-compose.yml`, `vercel.json`, and the root-level docs (`CLAUDE.md`, `ARCHITECTURE.md`, `DECISIONS.md`, `FEATURES.md`).
- You may **read** files outside `ai-service/` to understand a contract (for example, how the backend calls an AI endpoint). Reading is fine; writing is not.
- Do not run commands that change files outside `ai-service/` (installs, formatters, code generators, `git` operations on other paths). Run commands from inside `ai-service/`.
- Commit only files under `ai-service/`. Never use `git add -A` or `git add .` from the repo root.

If a task cannot be finished without a change outside `ai-service/` (a new backend route, a frontend call, a new compose env var), **stop and do not make that change**. Finish the `ai-service/` part, then tell the developer exactly what the web team must change: the file, the endpoint, and the request and response shape.

## What this service is

FastAPI service, Python 3.12+, port 8000. It holds the LLM logic for FitNutro.

- Only the backend (`web/backend`, port 4000) calls this service. The browser never does. CORS allows `http://localhost:4000` only; keep it that way.
- LLM keys live in `ai-service/.env` and never leave this service. Never commit `.env`, never log keys, never return them in a response.
- The service has no public route in production. Do not add auth-free endpoints that assume public access is safe.

## Layout

| Path | Purpose |
|---|---|
| `app/main.py` | App setup, CORS, `/health`. Includes the AI router. |
| `app/routers/ai.py` | All AI endpoints, mounted under `/ai`. Add new endpoints here. |
| `requirements.txt` | Dependencies. Add new ones here. |
| `.env.example` | Document every new env var here (name only, no value). |

## Commands

Run inside `ai-service/`:

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

There is no test setup yet. If you add tests, keep them inside `ai-service/`.

## The API is a contract

The backend depends on the paths and JSON shapes under `/ai`. Changing or removing an existing endpoint, field, or status code breaks the web team's code, which you cannot fix.

- Adding endpoints and adding optional fields is safe.
- Before you rename, remove, or change the type of anything existing, tell the developer so they can coordinate with the web team.
- Use Pydantic models for every request and response body.

## Product rules

These come from `FEATURES.md` and apply to anything the AI generates:

- Workout-plan changes are **suggestions**. The user must approve them; never present them as applied.
- Nutrition values are **estimates**. Label them as such.
- The chatbot gives **no medical diagnoses**.
- Trainers only see data for users who granted permission. Do not return another user's data on the assumption that the caller is allowed to see it.
