# HMAC Bridge (Next.js ↔ Python)

**Impact:** CRITICAL  
**Tags:** hmac, auth, wire-contract

## Contract

Both sides must change in the **same PR**:

| Python | TypeScript |
|--------|------------|
| `lib/hmac_auth.py` | `internal-hmac.ts` |

## Verification (Python)

```python
# Pseudocode — see lib/hmac_auth.py
def verify_hmac(request):
    timestamp = request.headers["X-BrewHub-Timestamp"]
    if abs(now() - int(timestamp)) > 60:
        raise HTTPException(401, "Stale signature")
    expected = hmac_sha256(SECRET, f"{timestamp}:{body}")
    if not hmac_compare(expected, request.headers["X-BrewHub-Signature"]):
        raise HTTPException(401, "Invalid signature")
```

Reject **before** ADK invoke.

## Signing (TypeScript)

```typescript
const timestamp = Math.floor(Date.now() / 1000).toString();
const signature = signHmac(INTERNAL_AGENT_SHARED_SECRET, timestamp, body);
```

## Secret storage

`INTERNAL_AGENT_SHARED_SECRET` in Doppler:

- `brewhub/prd_python` for Coolify
- `brewhub-netlify/prd` for Next.js projection

Never commit; never log.

## Protected routes

Maintain explicit list of paths requiring HMAC (`PROTECTED_AGENT_PATHS`) — don't default-open new endpoints.

## References

- `security-specialist` for OWASP API auth patterns
