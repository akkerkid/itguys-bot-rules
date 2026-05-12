# 13 — Cross-References

When your change affects behaviour documented elsewhere, add a `DOCS:` marker comment
pointing to where the documentation should be updated.

Format: `# DOCS: update <filename> <section> to reflect <what changed>`

Examples:
```python
# DOCS: update CODEX.md "Phase 2b" section to reflect new extraction field
# DOCS: update README.md "Caller Directory" to reflect new exclusion list format
```

Do not update the documentation yourself unless the issue explicitly asks for it.
The marker is a signal to AkkerKid to update docs during review.
