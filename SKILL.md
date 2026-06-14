---
name: python-ai-agents-specialist
description: Google ADK Python agents, FastMCP servers, Pydantic tools, HMAC bridges, allergen safety layers, and Coolify deployment for production AI backends.
version: "1.0.0"
tags:
  - python
  - adk
  - fastmcp
  - mcp
  - pydantic
  - agents
  - llm-tools
when_to_use:
  - Authoring ADK LlmAgent workers or FastAPI entrypoints
  - Building MCP tools (FastMCP) callable from Next.js or other clients
  - Writing lib/tools with Pydantic contracts and Supabase/Square integrations
  - Implementing three-layer allergen/medical safety parity with TypeScript
  - HMAC-signed service-to-service auth from Next.js to Python
  - Queue workers, cron dispatch, or Coolify entrypoint patterns
invocation_examples:
  - "Add a read-only ADK tool that fetches menu data with correct Supabase client"
  - "Wire a new FastMCP tool with member JWT context — no customer_id in args"
  - "Review this agent for missing allergen kill switch layers"
  - "Design idempotent Square write from ADK session context"
disable-model-invocation: true
---

# Python AI Agents Specialist

Production patterns for **Google ADK** + **FastMCP** Python backends. Centers on **enforcement over prompts**: tool lists, Pydantic schemas, middleware, and RPCs — not instruction-string hope.

## Overview

### Progressive disclosure map

| Need | Load |
|------|------|
| ADK `LlmAgent` workers | `patterns/adk-agents.md` |
| FastMCP / JSON-RPC bridge | `patterns/mcp-servers.md` |
| `lib/tools` authoring | `patterns/python-tools.md` |
| Allergen 3-layer kill switch | `patterns/safety-allergen.md` |
| HMAC + JWT bridge | `patterns/hmac-bridge.md` |
| Queue drainer / cron | `patterns/queue-workers.md` |
| FastAPI + Coolify deploy | `patterns/fastapi-deployment.md` |
| Mistakes | [anti-patterns.md](anti-patterns.md) |
| Abstract stack wiring | [examples/brew-hub-integration.md](examples/brew-hub-integration.md) |
| Links | [references/official-docs-links.md](references/official-docs-links.md) |

### Architecture principle: specialist brain, not main mouth

Customer-facing chat often lives in **Next.js** (`streamText`, SSE, safety gate). Python tier handles **staff MCP**, **workflow agents**, **crons**, and **queue drainers** — not direct WebView speech unless explicitly migrated.

---

## Core Principles

### 1. LLM tool args = untrusted edge

Same posture as browser JSON. Never accept `customer_id`, prices, or totals from tool input — resolve from JWT/HMAC context and recompute from DB.

### 2. Intent / Contract / Enforcement

| Layer | Lives in |
|-------|----------|
| Intent | Agent `instruction=`, tool descriptions |
| Contract | Pydantic models, tool signatures |
| Enforcement | Middleware, absent tools, RPC calls, regex scrubbers |

Markdown "must never" rules need a code enforcer.

### 3. Three-layer allergen safety

Pre-LLM gate → mid-stream scrub → post-response audit. Parity tests against TypeScript suite.

### 4. Call Postgres RPCs — don't re-implement

`checkStoreOpen`, `check_rate_limit`, `increment_api_usage` live in Supabase.

### 5. Background work via queue — not `after()`

Request-scoped containers don't guarantee post-response lifecycle. Use `ai_job_queue` + drainer.

---

## BrewHub Integration

Abstract patterns in [examples/brew-hub-integration.md](examples/brew-hub-integration.md). Wire-contract pairs with `nextjs-specialist` and `supabase-specialist`.

### Kill switches

- Ship agent without all three allergen layers
- Bind refund/write tools without human-in-loop design
- Trust LLM-supplied `customer_id` or line-item prices
- Skip HMAC verify before ADK invoke

---

## Integration with Other Specialists

| Skill | Handoff |
|-------|---------|
| `nextjs-specialist` | `/api/chat`, MCP bridge TS, `internal-hmac.ts` |
| `supabase-specialist` | `bearer_client`, RLS, advisory locks, SKIP LOCKED queue |
| `coolify-hetzner-specialist` | `entrypoint.sh`, `DOPPLER_WRAP`, `APP_COMMAND` |
| `cloudflare-specialist` | Access + tunnel to MCP hostname |
| `square-payments` | Webhook reconciliation, idempotency contract |
| `twilio-elevenlabs-resend-specialist` | Resend from cron agents |

---

## Book-to-skill

`references/book-summaries/` — validate with `node scripts/validate-skill.mjs`.
