# Agent Instructions — API

> Part of a monorepo — read [../../README.md](../../README.md) and [../../AGENTS.md](../../AGENTS.md) first.

FastAPI + LangGraph backend. Python 3.12, `uv` package manager.

## Dev Commands

```bash
uv sync              # install / update dependencies
uv run api dev       # hot-reload dev server → :8000
uv run api serve     # production (--workers N for multi-process)
uv run pytest        # run tests
uv run ruff check .  # lint
uv run ruff format . # format
```

## Source Layout

```
src/api/
├── main.py            App factory — CORS, router mounts, lifespan (Postgres init)
├── config.py          Pydantic Settings — all env vars, LLMProvider type alias
├── cli.py             Typer CLI — `dev` and `serve` commands
├── agent/
│   ├── graph.py       build_graph() + build_llm() — ReAct graph assembly
│   ├── state.py       AgentState TypedDict
│   ├── tools.py       @tool definitions + TOOLS list
│   └── checkpointer.py  AsyncPostgresSaver context manager
└── routers/
    ├── __init__.py    Exports agent_router
    └── agent.py       POST /api/chat  +  POST /api/chat/stream
```

## Endpoint Schemas

**ChatRequest** (shared by both chat endpoints):

```json
{
  "message": "string",
  "thread_id": "uuid (optional)",
  "provider": "openai|mistral (optional)"
}
```

**ChatResponse** (`/api/chat`): `{ "thread_id", "content", "provider" }`

`GET /api/providers` → `{ "providers": ["openai", "mistral"], "default": "openai" }`

## Graph Invocation Patterns

```python
# Non-streaming
config = {"configurable": {"thread_id": thread_id}}
result = await graph.ainvoke({"messages": [HumanMessage(content=msg)]}, config=config)

# Streaming (used by /api/chat/stream)
async for event in graph.astream_events({...}, config=config, version="v2"):
    if event["event"] == "on_chat_model_stream":
        token = event["data"]["chunk"].content
```

## How to Add a Tool

1. Add a `@tool`-decorated function to `src/api/agent/tools.py` — the docstring becomes the LLM tool description
2. Append it to the `TOOLS` list — `build_graph()` binds `TOOLS` automatically, no other changes needed

```python
@tool
def my_tool(param: str) -> str:
    """One-sentence description the LLM will see."""
    ...

TOOLS = [get_current_time, calculate, my_tool]
```

## How to Add a Router

1. Create `src/api/routers/<name>.py` with a `FastAPI APIRouter`
2. Export it from `src/api/routers/__init__.py`
3. Mount in `src/api/main.py`: `app.include_router(my_router, prefix="/api")`

## How to Add a New LLM Provider

1. Add the provider string to `LLMProvider` in `config.py`
2. Add API key + model fields to `Settings` in `config.py`
3. Add an `if resolved == "<provider>":` block in `build_llm()` in `agent/graph.py`

## Coding Conventions

- Every module starts with `from __future__ import annotations`
- All I/O is async — `async def` / `await` throughout
- Ruff line-length: **100** — run `uv run ruff format .` before committing
- Graph nodes return `{"messages": [new_msg]}` — `add_messages` reducer merges automatically
- Tools return plain strings (including errors) — never raise exceptions to the graph
