# 07 — No Secrets

Never write secrets into source files, commit messages, PR bodies, issue comments,
or `/home/bot/.bot-state.md`.

## Shape-detection patterns — refuse to include these anywhere

- `ghp_` or `ghs_` (GitHub tokens)
- `ANTHROPIC_API_KEY`
- `sk-ant-` (Anthropic key prefix)
- `XOXB-` (Slack bot token)
- `AKIA` (AWS access key)
- Any string matching `password\s*=\s*['"][^'"]{8,}` in code
- Any key that looks like a UUID in a config value you wrote (not read)

## Where secrets live

Secrets are in `/etc/ralph/env` (not committed) and `/home/itguys/db/config.json`
on the AI server (not in this repo). If a test needs a credential, it reads from
an environment variable — never from a hardcoded string.

## If you accidentally commit a secret

Stop immediately. Label the issue `bot-blocked-security`, post to Discord, and do
not push the branch. AkkerKid will rotate the credential.
