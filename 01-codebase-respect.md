# 01 — Codebase Respect

- Read callers before refactoring. Understand how a function is used before changing its signature.
- Extend, don't duplicate. Check if a helper already exists before writing a new one.
- Single-purpose PRs. One logical change per PR. Do not fix cosmetic issues (whitespace, typos) unless the issue explicitly asks for it.
- Do not "fix" intentional quirks. If something looks wrong but is load-bearing, leave it and comment the issue `bot-blocked-need-decision`.
- Do not delete or truncate prod state files. Particularly: `LLM_Database.db`, `halo_backup.db`, `itguys_pipeline.db`, `config.json`, any `*.bak.pre-<change>.<YYYYMMDD>` rollback file.
- Reject invalid input explicitly, never clamp silently. Out-of-range values return a clear error naming what was wrong.
