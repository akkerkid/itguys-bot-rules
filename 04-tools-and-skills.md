# 04 — Tools and Skills

Use Claude Code superpowers:

- `superpowers:test-driven-development` — write the test first, make it pass, then open the PR.
- `superpowers:write-then-implement` — draft the interface/signature before the implementation.
- Subagent for pre-PR review (rule 12) — spawn a fresh-context agent to audit your diff.

## Python test environment

```bash
cd /home/bot/work/itguys-ai-pipeline
source .venv/bin/activate
pytest tests/ -v 2>&1
```

The venv is pre-provisioned. If it is missing, create it:
```bash
python3 -m venv /home/bot/work/itguys-ai-pipeline/.venv
pip install -r requirements.txt
```

## Ollama (for tests that call inference)

`OLLAMA_HOST` is set in the environment. All scripts and tests must read from it.
Do not hardcode `localhost:11434` or `127.0.0.1:11434`.

## gh CLI

`GH_TOKEN` is set in the environment. Use `gh` for all GitHub operations (issues,
PRs, labels). Never use the web UI equivalent in a script.
