# Safety — Allergen Kill Switch (3 Layers)

**Impact:** CRITICAL  
**Tags:** allergen, safety, parity, audit

**No agent ships without all three layers** — even staff agents (prompt injection lateral movement).

## Layer 1 — Pre-LLM regex gate

```python
def is_allergen_or_medical_query(text: str) -> bool:
    ...

def allergen_safe_response() -> str:
    return CANONICAL_REFUSAL  # deterministic, not model-generated
```

If match → return canonical response **without** calling the model.

## Layer 2 — Mid-stream scrubber

Wrap ADK/SSE text stream; replace dangerous patterns before tokens reach client:

```python
async def scrubbing_text_stream(stream):
    async for chunk in stream:
        yield scrub_chunk(chunk)
```

## Layer 3 — Post-response audit

Every Layer-1 hit or scrub event writes `franklin_safety_audit`:

- `agent_name`, `user_id`, `model`, `tool_calls`, `matched_pattern`

SOC 2 evidence + forensics.

## Parity with TypeScript

Mirror `allergen-safety.ts` exactly. CI runs `tests/safety/test_allergen_parity.py` against JS test vectors — **block deploy on regression**.

## Tool-layer rule

Tools return **raw catalog data** — never construct `"free of {allergen}"` strings. Model + middleware own refusal UX.

## References

- OWASP LLM01 prompt injection awareness
- Parent monorepo: `src/lib/chat/allergen-safety.ts` ↔ `lib/safety/allergen.py`
