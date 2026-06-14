# Queue Workers & Cron

**Impact:** HIGH  
**Tags:** queue, drainer, cron, skip-locked

## No `after()` in Python request handlers

Post-response extraction / marketing jobs → **`ai_job_queue` table** + drainer worker.

## Drainer pattern

```python
# workers/ai_job_queue_drainer.py
# Loop: dequeue via RPC or SQL FOR UPDATE SKIP LOCKED
row = dequeue_next_job()
if row:
    process_job(row)
    mark_complete(row.id)
```

Use Postgres `SKIP LOCKED` — multiple drainers don't block each other. See `supabase-specialist`.

## Coolify deployment shapes

| Workload | Coolify type |
|----------|--------------|
| Continuous drainer | Long-lived app (`APP_COMMAND=python -m workers.ai_job_queue_drainer`) |
| Marketing crons | Scheduled Tasks → `APP_COMMAND` dispatch |
| Web brain + MCP | Main app with gunicorn/uvicorn |

## Shadow mode migration

Run new drainer parallel to legacy Edge Function; compare output quality on sample set before cutover.

## Idempotency

Jobs must tolerate retry — use unique job keys + status transitions (`pending` → `processing` → `done`).

## References

- `coolify-hetzner-specialist` → `doppler-secrets.md`, `APP_COMMAND`
- `supabase-specialist` → `rpc-atomic-operations.md`
