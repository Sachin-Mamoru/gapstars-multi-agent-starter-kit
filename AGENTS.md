# Agent Instructions — Gapstars Multi-Agent Starter Kit

> **Start here:** Read [README.md](README.md) first — it covers the full stack, project structure, quick-start commands, and the LangGraph agent overview.

This file adds what the README omits: agent-specific conventions, the SSE coupling contract, and pointers to per-app instructions.

## App Instructions

- [apps/api/AGENTS.md](apps/api/AGENTS.md) — FastAPI + LangGraph backend conventions
- [apps/web/AGENTS.md](apps/web/AGENTS.md) — Next.js + shadcn/ui frontend conventions

## Local Dev (without Docker)

```bash
docker compose up postgres redis   # infrastructure only
cd apps/api && uv run api dev      # hot-reload → :8000
cd apps/web && bun dev             # Turbopack → :3000
```

When running without Docker, set in `.env`:

- `DATABASE_URL` host: `postgres` → `localhost`
- `NEXT_PUBLIC_API_URL=http://localhost:8000`

## SSE Contract (the only coupling between apps)

The frontend calls `POST /api/chat/stream`. Events:

| Event   | Payload                                   |
| ------- | ----------------------------------------- |
| `token` | `{"token": "..."}`                        |
| `done`  | `{"thread_id": "...", "provider": "..."}` |
| `error` | `{"detail": "..."}`                       |

`NEXT_PUBLIC_API_URL` controls which API instance the frontend targets.

## packages/

Currently empty. Add a `package.json` or `pyproject.toml` before adding shared code.
