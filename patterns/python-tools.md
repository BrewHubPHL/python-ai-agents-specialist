# Python Tools (`lib/tools`)

**Impact:** CRITICAL  
**Tags:** pydantic, tools, supabase, square

## Canonical structure

```python
# lib/tools/get_menu.py
from pydantic import BaseModel
from lib.supabase_clients import get_anon_client

class MenuToolInput(BaseModel):
    include_modifiers: bool = True
    category: str | None = None
    # NO customer_id field — deliberate absence

class MenuToolOutput(BaseModel):
    ok: bool
    items: list[dict]
    error: str | None = None

async def get_menu(
    include_modifiers: bool = True,
    category: str | None = None,
) -> dict:
    """Return active menu. Read-only."""
    client = get_anon_client()
    # ... query with RLS-safe anon client for reads
    return MenuToolOutput(ok=True, items=rows).model_dump()
```

## Supabase client selection

| Client | Use |
|--------|-----|
| `get_anon_client()` | Reads under RLS |
| `bearer_client(jwt)` | User-scoped reads/writes |
| `get_service_client()` | System writes — narrow, explicit |

Resolve `user_id` from `bearer_client` context — **not** from tool args.

## Pre-write recompute (money-adjacent)

```python
# Before order insert / Square charge:
# 1. Re-read price_cents + modifier deltas from Supabase
# 2. Compare to LLM-supplied total
# 3. Reject if drift > $0.01
```

## Square idempotency

```python
idempotency_key = sha256(f"{user_id}:{session_id}:{step_index}").hexdigest()
```

On Square 5xx: write `pending_orders`, return defer message — **webhook reconciles**, no inline retry loop.

## Read-only posture tests

Register tools with explicit posture (`READ_ONLY` vs `WRITE`) and test binding — e.g. shipping quote must stay quote-only.

## Testing

- Unit test tool body with mocked Supabase
- Schema tests: forbidden fields absent from input models
- Integration tests behind feature flags in CI

## References

- `supabase-specialist` → dual-client, RPC atomicity
- `square-payments` skill (monorepo) for webhook contract
