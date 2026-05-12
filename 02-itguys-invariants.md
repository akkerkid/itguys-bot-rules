# 02 — ITGuys AI Platform Invariants (mirrors host/CODEX.md hard rules)

These rules are non-negotiable. If your work would violate one, label the issue
`bot-blocked-need-decision`, escalate to Discord, and move on. Never push through.

## Client data and tenant isolation

- **Tenant isolation is absolute.** Every record that touches client data carries
  `attributed_client_id`. RAG context-assembly NEVER reads a collection unkeyed by
  `client_id`. There is a regression test (`test_rag_isolation.py`); if you touch
  the prompt-assembly layer, run it and paste the literal output (including exit
  code footer) in the PR body.
- **On-prem only for client data.** Cloud LLMs may only see synthetic or fully
  anonymised inputs. Phase 4 (Teams real-time) is BLOCKED and will remain blocked
  until a SIP-mirror or DMZ alternative is designed.
- **30,001 out-of-scope transcripts are quarantined.** Always filter client-facing
  queries by `WHERE provenance_source = 'fusionpbx:v_xml_cdr'`. A leak into a
  client RAG is a serious incident.
- **ITGuys-owned numbers are excluded from caller_directory.** The static exclusion
  list lives in `BuildCallerDirectory.py`. Do not bypass it.

## Data discipline

- **Provenance first, extraction later.** `attributed_client_id` must be set before
  any Pass-2 or Pass-3 processing touches a record. Schema migrations are gated on
  `*.bak.pre-<change>.<YYYYMMDD>` existing before the migration runs.
- **Composite PKs when natural IDs are per-parent.** Halo Action IDs are per-ticket
  sequential — `PRIMARY KEY (ticket_id, id)`, never just `id`. Audit external IDs
  for global uniqueness before making them sole PKs.
- **Local read cache + API writes only.** Halo is read from `halo_backup.db` (synced
  every 30 min by `halo_sync.py`). The live Halo API is for writes only. No live SSH
  tunnels or external API calls on hot paths.
- **Mutable rosters from `halo_backup.db` at runtime.** Never hardcode tech names or
  extension lists.
- **Atomic writes for state files.** `tempfile + os.replace()`. Never `open('w')` on
  shared SQLite cache state. All multi-row DB inserts in `with conn:` blocks.
- **Read `halo_backup.db`, never write it.** It is a sync replica; direct writes are
  overwritten by the next `halo_sync.py` run.

## LLM extraction rules

- **`temperature=0` + `format=<json schema>` + no few-shot examples.** Schema
  enforcement beats examples on weak models. Few-shot contaminates extraction tokens
  at 7B (Phase 2b lesson: 3/10 contamination rate).
- **`/no_think` in system prompt for `qwen3:*` models.** Without it, `<think>` blocks
  leak into JSON output.
- **No hedging language** ("(assumed)", "(presumed)") in extraction outputs. Explicit
  rule in the prompt; flag violations in PR review.

## Telephony rules

- **Stereo channel attribution is structural, not acoustic.** L=ITGuys, R=external.
  Never override based on transcript content.
- **`answer_epoch=0` means "not answered"** (FusionPBX quirk). Use the `_epoch()`
  helper. NEVER compare against `None` alone.
- **No `condition_on_previous_text=True` on recordings longer than 2 minutes.**
  Repetition death-loop rate is ~80%. Also required: `compression_ratio_threshold=2.0`,
  temperature ladder, `vad_filter=True`.

## Test isolation

- **Tests that touch `halo_backup.db` must copy it to a tmpdir first.** Never run
  tests against the shared replica; the next `halo_sync.py` run would overwrite it.
- **Tests that touch `LLM_Database.db` must use a fixture copy.** Never mutate the
  production database from a test.
- **Paste literal pytest output in the PR body.** Include the summary line and exit
  code footer. Never just claim "tests passed."

## Shared GPU host — 172.30.30.50 (RTX A6000, 48 GB VRAM)

This host is shared. `speaches` STT jobs (port 8000) are **highest priority** — they
block live call recordings from being transcribed. LLM inference is lower priority.
Govern yourself accordingly.

**Python client setup (mandatory)**
```python
from ollama import Client
import os
client = Client(
    host=os.environ.get('OLLAMA_HOST', 'http://localhost:11434'),
    timeout=600   # GPU may be busy loading another model — be patient
)
```

**Before starting an iteration / batch**
Check `/api/ps` — if more than ~35 GB is occupied, wait 60 s and retry (up to 10
times) before failing. The host may be mid-generation.
```python
import time, urllib.request, json
def wait_for_vram_headroom(needed_gb=20, host=None):
    host = host or os.environ.get('OLLAMA_HOST', 'http://localhost:11434')
    for _ in range(10):
        with urllib.request.urlopen(f'{host}/api/ps', timeout=10) as r:
            used = sum(m.get('size_vram', 0) for m in json.loads(r.read()).get('models', []))
        if used / 1e9 < (48 - needed_gb):
            return True
        time.sleep(60)
    return False
```

**After your iteration / batch completes (mandatory)**
Release VRAM immediately so STT and other bots can use it:
```python
client.generate(model=os.environ.get('RALPH_OLLAMA_MODEL'), keep_alive=0)
```
Or via curl in a shell script:
```bash
curl -s -X POST "${OLLAMA_HOST}/api/generate" \
  -d "{\"model\": \"${RALPH_OLLAMA_MODEL}\", \"keep_alive\": 0}" -o /dev/null || true
```

**Rules**
- Never hardcode `172.30.30.50` or `localhost:11434` — always use `$OLLAMA_HOST`
- One parallel inference at a time — Ollama queues but VRAM is shared
- Do NOT set `keep_alive=0` on every individual call within a batch (the model
  would thrash — load/unload on every record). Only on the **last** call.
- If your request fails with 503 or connection error mid-iteration, wait 120 s and
  retry once before treating it as a code bug
