# 12 — Pre-PR Review

Before opening or updating a PR, spawn a fresh-context subagent to audit your diff.

## Subagent prompt template

```
You are a code reviewer auditing a diff against the ITGuys AI Platform invariants.

Read this file: /home/bot/work/bot-rules/02-itguys-invariants.md

Then review the following diff:

<diff>
[paste git diff here]
</diff>

Answer Q1-Q5:
Q1. Does any change violate tenant isolation or read client data without scoping to attributed_client_id?
Q2. Does any change write to halo_backup.db or bypass halo_sync.py?
Q3. Does any change hardcode localhost:11434 or 127.0.0.1:11434?
Q4. Does any change touch LLM_Database.db or halo_backup.db in a test without fixture isolation?
Q5. Does any change modify Everything.sh, the Whisper condition_on_previous_text setting, or stereo channel attribution?

For each Q: answer YES/NO and cite the specific line if YES.
Final verdict: PASS (all NO) or BLOCK (any YES).
```

## If the verdict is BLOCK

Do not open the PR. Label the issue `bot-blocked-need-decision`, post the blocking
finding as a comment, write state, and move on.

## If the verdict is PASS

Open the PR. Include in the body:
- The Q1-Q5 answers from the subagent review
- The literal pytest output
- `Closes #NNN`
- A `Backup:` line if any schema migration is included
