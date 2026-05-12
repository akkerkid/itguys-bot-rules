# 10 — Inbox Phase

At the start of every iteration, run the inbox phase before any work.

## Step 1 — Check for bot-rebase labels on your open PRs

```bash
gh pr list --repo akkerkid/itguys-ai-pipeline --author ITGuysCoder \
  --state open --json number,title,labels \
  | jq '.[] | select(.labels[].name == "bot-rebase")'
```

If any: go directly to rule 08. Rebase and retest before picking new work.

## Step 2 — Check for in-progress work in bot-state

Read `/home/bot/.bot-state.md`. If there is an unfinished task, resume it.
Skip steps 3-4.

## Step 3 — Find a bot-eligible issue

```bash
gh issue list --repo akkerkid/itguys-ai-pipeline \
  --label bot-eligible --state open \
  --json number,title,labels,assignees \
  | jq '.[] | select(.assignees | length == 0)'
```

Pick the first unassigned result. If none: write "No work available" to
`/home/bot/.bot-state.md` and exit cleanly.

## Step 4 — Self-assign and run feasibility check

```bash
gh issue edit <N> --add-assignee ITGuysCoder --repo akkerkid/itguys-ai-pipeline
```

Then run rule 06 (feasibility check) before writing a single line of code.
