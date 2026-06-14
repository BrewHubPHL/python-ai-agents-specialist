# Anti-Patterns

## Agents & tools

### ❌ `customer_id` in Pydantic tool input
Resolve from JWT/`bearer_client` — OWASP BOLA.

### ❌ Trusting LLM line items or totals for checkout
Recompute from Supabase; reject drift.

### ❌ Refund/charge tool bound to agent `tools=[...]` without HITL design
Enforcement by absence for v1 refunds.

### ❌ Copy-paste megabyte system prompt into `instruction=`
Fetch SSOT from constants service; keep specialist instructions tight.

### ❌ Hardcoded model strings
Use `make_claude_model(env_key, default)`.

### ❌ Gemini/direct provider imports when policy is all-Anthropic
CI should block drift.

## Safety

### ❌ Shipping with only prompt-based allergen refusal
Need regex gate + scrub + audit.

### ❌ Tools returning "free of X" marketing strings
Raw data only; middleware owns refusal.

### ❌ Skipping parity tests vs TypeScript suite

## Integration

### ❌ Changing HMAC on one side only
Breaks bridge — same PR both languages.

### ❌ Python speaking directly to WebView
Route through Next.js SSE + safety gate.

### ❌ `after()`-style fire-and-forget in FastAPI
Use `ai_job_queue`.

### ❌ Inline retry loop on Square 5xx
Write `pending_orders`; webhook reconciles.

## Ops

### ❌ `.env` files committed
Doppler only.

### ❌ Re-implementing `checkStoreOpen` in Python
Call Postgres RPC.

### ❌ Background cron as OS crontab invisible to team
Coolify Scheduled Tasks for visibility.

## MCP

### ❌ Member ID as MCP tool parameter
Context from JWT middleware only.

### ❌ MCP cache TTL longer than Next.js bridge cache
Stale menu divergence.
