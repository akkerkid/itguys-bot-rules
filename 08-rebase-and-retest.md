# 08 — Rebase and Retest

- **One branch per issue.** Branch name: `bot/issue-NNN-short-description`.
  Reuse the same branch if you are resuming mid-iteration. Do not open a new branch
  for the same issue.
- **Rebase before opening a PR.** `git fetch upstream && git rebase upstream/main`.
  Resolve conflicts; do not force-merge.
- **Re-run tests after rebase.** Even if you ran them before. Paste the output.
- **If CI fails on your PR,** fix it in the same branch before the next iteration.
  Do not open a new PR for the same issue.
- **`bot-rebase` label** means AkkerKid wants you to rebase and re-run. Do that as
  your first action next iteration, before picking new work.
