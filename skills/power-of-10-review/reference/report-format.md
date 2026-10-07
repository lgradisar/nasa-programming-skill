# Report format

```markdown
## Power of 10 review

**Scope:** diff mode, 4 files changed (src/parser.py, src/cli.py, +2)
**Checks run:** ruff (completed, 3 diagnostics)
**Checks skipped:** mypy configured but not installed

| Rule | Location | Severity | Finding | Fix |
|---|---|---|---|---|
| 7 | src/parser.py:42 | high | `except Exception: pass` swallows parse errors; callers get `None` and continue. (ruff BLE001) | Catch `ValueError` only and re-raise with the line number. |

**Project diagnostics:** ruff F401 unused import `os` in src/cli.py:3

**Summary:** 1 high, 0 medium, 0 low.
**Pre-existing:** src/parser.py:42
**Recommendation:** none
```

## Filling it in

- `Checks run` names each tool and its outcome: clean, diagnostics, partial, or
  execution problem with its message. Write `none` when nothing ran, and say why
  under `Checks skipped`.
- One row per problem; several rule numbers are allowed, main one first. Cite the
  tool and rule code when a tool found it.
- `Location` is `path:line` or `path:start-end`.
- Severity by impact: **high** for wrong results, data loss, a crash, a hang, a leak,
  or a security hole; **medium** for hidden errors or a likely later failure;
  **low** for readability. Start a finding with "Possible:" when it depends on a
  condition the code does not settle.
- `Pre-existing` appears in diff mode only, for findings that existed before the
  change. Omit it in path mode.
- When the table is empty, write `**No findings.**` under the header and keep the
  header.
- `Recommendation` names one tool when a reviewed language has no configured
  linter; otherwise `none`. Never offer to install it.
