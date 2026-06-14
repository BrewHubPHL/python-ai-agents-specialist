# ADK Agents (LlmAgent)

**Impact:** CRITICAL  
**Tags:** adk, llm-agent, workers, fastapi

## Auto-discovery mount

```python
# main.py
from google.adk.cli.fast_api import get_fast_api_app

app = get_fast_api_app(agents_dir=".")
```

Every `workers/<name>/agent.py` exporting `root_agent` mounts at `app_name=<name>`.

```
POST /run_sse?app_name=ops   # HMAC-protected in production
```

## Canonical agent file

```python
# workers/ops/agent.py
from google.adk.agents import LlmAgent
from lib.model_config import DEFAULT_HAIKU, make_claude_model
from lib.tools.get_shift_report import get_shift_report

root_agent = LlmAgent(
    model=make_claude_model("OPS_AGENT_MODEL", DEFAULT_HAIKU),
    name="ops",
    instruction="""
You are the operations agent. Tools are read-only unless explicitly listed.
Allergen/medical questions are intercepted by middleware — do not volunteer ingredient claims.
""".strip(),
    tools=[get_shift_report],
)
```

## Rules

| Rule | Why |
|------|-----|
| Export `root_agent` | Required for discovery |
| `make_claude_model(env_key, default)` | No hardcoded model strings |
| `instruction` = intent only | Don't paste megabyte system prompts |
| `tools=[...]` enforces capability | No refund tool = no refunds (enforcement by absence) |
| Turn budget ~26s | Caller timeout discipline |

## Model config

```python
make_claude_model("OPS_AGENT_MODEL", DEFAULT_HAIKU)  # reads env override
```

All-Anthropic via LiteLLM + `CLAUDE_API_KEY`. Block direct Gemini SDK imports in CI if policy requires single provider.

## Middleware stack (enforcement)

- HMAC auth (`lib/hmac_auth.py`) — 60s freshness
- Allergen request path (`lib/safety/request_path.py`)
- Rate limit RPC before turn; `increment_api_usage` after

## ADK + MCP integration

ADK agents can consume **external MCP servers** via `MCPToolset` (ADK docs). A common pattern: reach Supabase/Square directly in `lib/tools/` — reserve the MCP layer for **cross-runtime** bridges (Next.js → Python).

## References

- [google/adk-python](https://github.com/google/adk-python)
- [ADK platform docs](https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/adk)
- [MCP tools in ADK](https://adk.dev/tools-custom/mcp-tools/)
