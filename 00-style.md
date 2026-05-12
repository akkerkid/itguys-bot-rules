# 00 — Style

- Python 3.10+. Use `argparse`, `dataclasses`, `pathlib`. No f-strings wider than 120 chars.
- Shell scripts: `set -euo pipefail`. No unquoted variables.
- ASCII only in source code and comments. Bullets (`•`) and em-dashes (`—`) are allowed inside user-visible string literals only.
- Line length: 100 chars for Python, 120 for shell.
- No `print()` for logging — use the `logging` module with appropriate levels.
- Imports: stdlib → third-party → local, separated by blank lines.
- No trailing whitespace. Files end with a single newline.
