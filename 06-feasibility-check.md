# 06 — Feasibility Check

Run this check BEFORE starting work on any issue.

## Refuse-on-sight conditions

Label `bot-blocked-need-decision` and move on immediately if the issue asks you to:

- Touch the FusionPBX live database (`10.12.14.150`) directly.
- Push Halo write operations without an idempotency strategy decided first.
- Change `condition_on_previous_text` behaviour, stereo channel mapping, or the
  `ITGUYS_OWNED_NUMBERS` exclusion list — these are AkkerKid-owned invariants.
- Modify the `Everything.sh` cron schedule or timing — it is load-bearing.
- Upgrade `open-webui` without pinning the version in the same PR.
- Run any script that contains a hardcoded `localhost:11434` or `127.0.0.1:11434`
  — these will silently fail on the bot VM (no local Ollama daemon).
- Access `LLM_Database.db` or `halo_backup.db` in a test without fixture isolation.

## Data-blocked issues

If the issue requires access to real client transcripts, real Halo ticket data, or
real FusionPBX CDR that you do not have a fixture for: label `bot-blocked-data`,
comment explaining what fixture is needed, and move on.

## Decision-blocked issues

If the issue requires an architectural decision (schema change, new dependency, new
service) that has not been pre-approved in the issue body: label
`bot-blocked-need-decision`, post a clear question as a comment, and move on.

## Hardware-blocked issues

If the issue requires GPU resources, a second Ollama model not in your allowlist,
or filesystem access outside `/home/bot/work/`: label `bot-blocked-hardware` and
escalate via Discord.
