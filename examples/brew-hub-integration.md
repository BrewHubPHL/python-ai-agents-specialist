# BrewHub Integration (Abstract)

## Who talks to whom

```
Capacitor/Web ──▶ Next.js /api/chat (streamText, tools, safety)
                      │
                      ├─ Supabase-direct (loyalty reads — customer path)
                      │
                      └─ optional MCP bridge ──▶ Python FastMCP (/mcp)
                                                    staff tools, HMAC + JWT

Python ADK workers ◀── HMAC POST /run_sse (staff/automation — not customer WebView)

Coolify crons ──▶ APP_COMMAND ──▶ marketing agents, queue drainer
```

**Python is the specialist brain, not the customer mouth** — customer chat stays in Next.js until a deliberate migration.

## Wire-contract pairs (modify both sides)

| Concern | Python | TypeScript |
|---------|--------|------------|
| HMAC | `lib/hmac_auth.py` | `internal-hmac.ts` |
| Allergen | `lib/safety/allergen.py` | `allergen-safety.ts` |
| Loyalty MCP | `franklin_mcp/server.py` | `loyalty-mcp-bridge.ts` |
| Shadow queue | `ai_job_queue_drainer.py` | `shadow-agent.ts` enqueue |

## Supabase clients in Python

Mirror dual-client philosophy:

- Anon + bearer for user-scoped reads
- Service client for narrow system writes
- RPCs for rate limits, store open, atomic merges

## Square path

Python may initiate charges with session-derived idempotency keys → audit log → webhook finality. No inline retry on 5xx.

## Deployment

Hetzner Coolify box:

- `DOPPLER_WRAP=1` + `brewhub/prd_python`
- Tunnel + Access on `mcp.*` / `brain.*` hostnames
- `APP_COMMAND` clones for drainers without new Dockerfiles

## Kill switches

- `FRANKLIN_MCP_BASE_URL` unset for customer chat when loyalty is Supabase-direct (isolates customer path from single brain box)
- No production agent without allergen parity CI green

## Private overrides

Exact hostnames, env key names, agent folder list → `brew-hub-overrides/`.
