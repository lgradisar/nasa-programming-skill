# Running project tools

A check must never change the code it checks or pull in software the project did
not choose. Build outputs in their normal folders are fine; reviewed files,
configuration, and dependencies must stay as they were.

## What you may run

Only tools the project has configured **and** installed locally: its virtualenv,
`node_modules/.bin`, the system compiler, the Go toolchain. No download-on-demand
fallbacks (`npx <tool>`, `pipx run`, `uvx`, `dotnet tool run` for an absent tool).
Never install anything, even when the project recommends a tool it lacks.

## How to run it

1. Read the command or script first; package scripts, `Makefile` targets, and CI
   steps often chain extra work.
2. Use the check-only form. Never run `--fix`, `--write`, formatters, code
   generation, migrations, or dependency installation. If a script bundles those,
   run only the underlying check yourself.
3. Keep the tool's supported scope. Pass single files only when the tool supports
   it; never compile isolated C files outside the project's build.
4. Run from the project root, in the project's environment, with the configuration
   nearest to the reviewed files and the tool's own inheritance rules.
5. Some checks fetch dependencies on their own. Use the form that does not:
   - .NET: `dotnet build --no-restore` (and check the project for generation
     targets that would rewrite source).
   - Go: `GOTOOLCHAIN=local GOPROXY=off GONOPROXY=none go vet ./...`.
   - TypeScript: `tsc --noEmit`, with `--incremental false` when `tsconfig.json`
     sets `incremental`; when it sets `composite`, that flag is rejected, so report
     the type check as not run safely.
   - Node: `npm run <check>` only when `node_modules` exists.
   If the command then fails for a missing prerequisite, report it; do not fetch.

## Where configuration lives

| Language | Look for |
|---|---|
| Python | `[tool.ruff]` in `pyproject.toml`, `ruff.toml`, `.pylintrc`, `[tool.mypy]` |
| JavaScript / TypeScript | `eslint.config.*`, `.eslintrc*`, `tsconfig.json`, `lint`/`typecheck` scripts |
| C | `.clang-tidy`, warning flags in `Makefile`/`CMakeLists.txt`, `compile_commands.json` |
| C# | `.editorconfig` analyzer rules, `Directory.Build.props`; `dotnet build` runs analyzers |
| Go | `go vet` (in the toolchain), `.golangci.yml` |

## Reading the result

- **Clean:** the tool ran and found nothing. Report it as run and clean.
- **Diagnostics:** the tool ran and found issues; a non-zero exit code is normal.
  Report them.
- **Execution problem:** executable missing, configuration broken, or a crash.
  Report it in the header as incomplete coverage; it is not a code finding.
- **Partial:** some files or checks were skipped. Keep the diagnostics and state
  the gap.

A diagnostic becomes a rule finding only after you confirm it violates the rule as
adapted in the skill: a linter flagging a C cleanup `goto`, an `assert` on an
internal invariant, or a constant macro is not a violation.

A suppression (`/* eslint-disable */`, `#pragma warning disable` without an id,
`# noqa` without a code) removes that tool's coverage for the region it spans.
Review that region by reading, keep other tools' diagnostics, and report a
file-wide or reasonless suppression under rule 9.

## Never modify

While checking, do not change source files, configuration, lock files, or installed
dependencies.
