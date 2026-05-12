# 14 — Debugging Discipline

- **Two-attempt cap per bug.** If you have tried two distinct approaches and the
  test still fails, stop. Write what you tried in a PR comment, label the issue
  `bot-blocked-need-decision`, and move on.
- **Check infrastructure before blaming code.** Before assuming a test failure is
  a code bug, verify: Ollama reachable? `.venv` active? Fixture file present?
- **Do not suppress errors.** Never catch an exception just to make a test pass.
  Never use `# type: ignore` to hide a type error you introduced.
- **Read error messages completely.** The last line of a traceback is usually the
  actual problem. Read all of it before changing code.
- **Do not guess at fixes.** If you do not understand why something is failing,
  add a print/log statement to observe the actual value, then fix based on evidence.
- **Check Ollama before blaming the code.** The GPU host (`$OLLAMA_HOST`, default
  `http://172.30.30.50:11434`) serves multiple bots and the production pipeline
  simultaneously. Before concluding that a test failure is a code bug:
  1. `curl -s $OLLAMA_HOST/api/tags` — HTTP 200 means the server is up.
  2. `curl -s $OLLAMA_HOST/api/ps` — check whether another model is occupying VRAM.
  3. If the server is unreachable or returning 503, **wait 60–120 s and retry once**
     before reporting the failure. The loop.sh pre-flight already does this for
     iteration startup, but mid-test Ollama hiccups can still happen.
  4. All Python Ollama clients must use `timeout=600` and read `OLLAMA_HOST` from
     the environment (`os.environ.get('OLLAMA_HOST', 'http://localhost:11434')`).
     Never hardcode `localhost`.
