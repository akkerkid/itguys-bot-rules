# 11 — Work Phase (TDD)

Follow test-driven development strictly.

## Order of operations

1. Read the issue acceptance criteria carefully.
2. Write a failing test that captures the acceptance criteria.
3. Run it — confirm it fails for the right reason.
4. Implement the minimum code to make it pass.
5. Run the full test suite — confirm nothing regressed.
6. Commit (GPG-signed). Reference the issue: `Refs #NNN` or `Closes #NNN`.

## Pytest invocation

```bash
cd /home/bot/work/itguys-ai-pipeline
source .venv/bin/activate
pytest tests/ -v 2>&1
```

**Always paste the literal pytest output** — summary line + exit code footer — in
the PR body. Never claim "tests passed" without the output.

## Context ceiling

Keep `/home/bot/.bot-state.md` under 5 KB. Summarise and truncate at iteration end.
If the model context is running low (you notice rule files being ignored), stop,
write state, and resume next iteration.

## Ollama calls in tests

If the code under test calls Ollama, verify the daemon is reachable before assuming
a test failure is a code bug:

```bash
curl -sS --max-time 5 http://172.30.30.50:11434/api/tags | jq '.models | length'
```

If that returns 0 or errors, it is an infrastructure problem — label
`bot-blocked-hardware` and escalate.
