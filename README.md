# NASA Power of 10 for everyday code

Two agent skills that bring NASA/JPL's [Power of 10](https://spinroot.com/gerard/pdf/P10.pdf)
rules to ordinary software. The original rules were written for safety-critical C.
These are the same ideas, generalized so that they help in Python, JavaScript,
TypeScript, C, C#, and Go instead of getting in the way.

Works with Claude Code as a plugin, and with any agent that reads the open
`SKILL.md` format.

## What is inside

| Skill | What it does |
|---|---|
| `power-of-10:code` | Loads while the agent writes or edits code. Gives it the eleven rules plus a short reference for the language at hand. |
| `power-of-10:review` | Reviews code on request. Runs the project's existing linters in check-only mode, reads the rest, and reports each violation with rule number, location, severity, and a fix. Never changes code. |

Both skills share one set of language references, one per language, each about
400 words.

## Install

### Claude Code plugin

```bash
/plugin marketplace add lgradisar/nasa-programming-skill
```

```bash
/plugin install power-of-10@nasa-skills
```

The review command is then `/power-of-10:review`.

### Plain skills

Clone the repo and copy both folders from `skills/` into `~/.claude/skills/` (all
projects) or `.claude/skills/` (one project). Both folders are needed; the reviewer
reads the rules from its sibling. The review command is then `/review`. If that
name is taken, rename the folders; keep them next to each other.

## The eleven rules

| # | Rule |
|---|---|
| 1 | Straightforward control flow: no `goto` (except C cleanup jumps), bounded recursion, shallow nesting |
| 2 | Every loop terminates visibly; waiting loops get a deadline from the caller, service loops get a shutdown path |
| 3 | Predictable resources: one owner and a release point for every resource; limits where load can exceed capacity |
| 4 | Functions are cohesive and readable in one pass; about 60 lines is a signal, not a cutoff |
| 5 | Validate external input at the boundary; assertions only for internal invariants |
| 6 | Smallest possible scope; no uncontrolled shared mutable state |
| 7 | Check every result that can fail; never swallow a failure silently |
| 8 | Explicit over dynamic: never execute untrusted data as code; plain calls over reflection |
| 9 | Clean under the project's own checks; fix warnings, do not hide them |
| 10 | Prefer simplicity over complexity |
| 11 | Code is readable without comments; comment only decisions the code cannot express |

Rules 1 to 9 come from NASA; 10 and 11 were added. Above them sits one policy:
correctness and project conventions come first, rules apply in proportion to the
change, and the agent never invents a limit or a timeout. When a number is needed
and the requirements do not give one, it becomes a required parameter.

## Use

The writing skill activates on its own when the task is editing source code.
Activation is best-effort, as with any skill. You can also name it: "follow the
Power of 10 rules".

Review the current changes (staged, unstaged, and untracked files):

```bash
/power-of-10:review
```

Review one file or folder, changed or not:

```bash
/power-of-10:review src/parser.py
```

Plain words work too: "review this against the NASA rules".

The reviewer uses linters only when the project already has them configured and
installed. It never installs tools, never runs `--fix`, and never edits files. If a
language has no linter, the report names one you could add.

## Layout

```
.claude-plugin/        plugin.json, marketplace.json
skills/
  code/
    SKILL.md           policy, the eleven rules, language table
    reference/
      tooling.md       how to run project tools safely
      python.md  javascript.md  typescript.md  c.md  csharp.md  go.md
  review/
    SKILL.md           review procedure
    reference/
      report-format.md
```

## License

MIT
