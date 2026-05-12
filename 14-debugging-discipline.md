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
