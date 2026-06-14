# FastAPI Deployment (Coolify)

**Impact:** HIGH  
**Tags:** fastapi, uvicorn, docker, coolify

## Entrypoint

```python
# main.py
from google.adk.cli.fast_api import get_fast_api_app
app = get_fast_api_app(agents_dir=".")
```

MCP may mount as ASGI sub-app (`<agent>_mcp.asgi:app`).

## Local dev

```bash
doppler run -- uv run uvicorn main:app --reload
```

Never commit `.env*`.

## Docker / Coolify

```dockerfile
# Dockerfile — multi-stage as needed
CMD ["/app/entrypoint.sh"]
```

```bash
# entrypoint.sh
# DOPPLER_WRAP=1 → doppler run -- gunicorn ...
# APP_COMMAND for worker clones
```

Secrets: `brewhub/prd_python` via Doppler at boot.

## Health endpoints

- `/health` or framework default — must pass through Access Service Auth for external monitors
- Don't expose unauthenticated admin routes

## Observability

- PostHog `capture_exception` on caught errors (mirror JS patterns)
- Structured log prefix `[CATEGORY][SUBCATEGORY]` for grep

## CORS

If any route is reachable from Capacitor WebView, match native CORS allowlist from `nextjs-specialist` / mobile skill.

## References

- [Coolify docs](https://coolify.io/docs)
- `coolify-hetzner-specialist` patterns
