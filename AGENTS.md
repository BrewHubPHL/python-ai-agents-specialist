# Python AI Agents — Navigation

## Pattern priority

| Priority | File |
|----------|------|
| CRITICAL | `patterns/python-tools.md`, `patterns/safety-allergen.md`, `patterns/hmac-bridge.md` |
| HIGH | `patterns/adk-agents.md`, `patterns/mcp-servers.md` |
| MEDIUM | `patterns/queue-workers.md`, `patterns/fastapi-deployment.md` |

## Stack defaults

Python 3.12 · uv · Google ADK · FastAPI · FastMCP · Pydantic v2 · LiteLLM (Claude) · Supabase Python

## Operating principles

The fleet-wide philosophy (zero-trust the client, fail closed, idempotency on the money
path, etc.) lives in [PRINCIPLES.md](PRINCIPLES.md). Apply it on top of the domain
patterns in this repo.
