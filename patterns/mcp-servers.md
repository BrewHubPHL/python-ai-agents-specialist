# MCP Servers (FastMCP)

**Impact:** HIGH  
**Tags:** fastmcp, mcp, json-rpc, bridge

## When to use MCP

| Use MCP | Skip MCP |
|---------|----------|
| Next.js / Cursor calls Python tools over HTTP | ADK agent calling Supabase in-process |
| Staff loyalty/menu bridge | Internal-only Python orchestration |
| Standardized tool discovery for clients | Hot path where RPC overhead hurts |

## FastMCP server shape

```python
from mcp.server.fastmcp import FastMCP
from franklin_mcp.member_jwt_context import get_current_member_id
from lib.tools.get_loyalty_snapshot import fetch_loyalty_snapshot

mcp = FastMCP("My MCP Server")

@mcp.tool(
    name="loyalty.snapshot",
    description="Loyalty tier and points for the authenticated member.",
)
async def loyalty_snapshot_tool() -> dict:
    member_id = get_current_member_id()
    if not member_id:
        return {"ok": False, "code": "unauthenticated"}
    return await fetch_loyalty_snapshot(member_id=member_id)
```

## Auth layers

1. **HMAC** — `INTERNAL_AGENT_SHARED_SECRET`, 60s timestamp window (middleware)
2. **Member JWT** — `X-BrewHub-Member-Jwt` header → context var; never tool arg
3. **Cloudflare Access** — service token at edge (client sends `CF-Access-*`)

## Client (Next.js) pattern

```
StreamableHTTPClientTransport → POST /mcp
Headers: HMAC signature + member JWT + Access tokens
Fail-open: bridge returns null on transport error (UX degradation, not unsafe defaults)
```

## Tool naming

- `snake_case` or dotted (`loyalty.snapshot`)
- Descriptions are **intent** — contracts live in Pydantic models in `lib/tools/`

## Caching parity

MCP tool TTL must match or undercut JS-side cache TTL for same data — avoid stale menu divergence.

## References

- [ADK MCP tools guide](https://adk.dev/tools-custom/mcp-tools/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
