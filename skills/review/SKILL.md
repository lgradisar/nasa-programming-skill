---
name: review
description: Review code against the NASA Power of 10 rules (NASA/JPL safety-critical coding rules, generalized) and report violations with rule number, location, severity, and a suggested fix. Use when asked to review, check, or audit code against the NASA rules, the Power of 10, or this skill. Runs the project's existing linters in check-only mode when available. Reviews the current changes by default, or a given file or folder.
argument-hint: "[path]"
allowed-tools: Read, Grep, Glob, Bash
---

# Power of 10 review

You review code; you do not change it. Report findings, then stop.

## Load the rules

Read, relative to this file's folder:

1. `../code/SKILL.md` for the policy and the 11 rules.
2. `../code/reference/tooling.md` for how to run project tools safely.
3. The language reference for each language present, per the extension table in
   `SKILL.md` (TypeScript needs `javascript.md` and `typescript.md`).

If the sibling skill is missing, say so and stop; both skills must be installed
together.

## Scope

**Diff mode** (no argument): staged and unstaged changes against `HEAD`, plus
untracked source files. Read changed lines with enough context to judge them, and
mark issues that existed before the change as pre-existing. With no commits or no
git repository, ask for a path.

**Path mode** (argument given): the contents of that file or folder, changed or not.

Skip generated, vendored, binary, lock, and minified files unless the user names
them.

## Procedure

1. Run the project's configured checks as `tooling.md` says. Execution problems go
   in the report header, never in the findings.
2. Review all 11 rules by reading. Tool diagnostics are evidence to confirm against
   the rules as adapted here, not findings by themselves, and a clean run does not
   prove compliance. Keep diagnostics that match no rule under "project
   diagnostics".
3. A finding needs evidence in the code: a declared type, a signature, a value that
   is used unchecked. Say what condition is unresolved when there is one. Invented
   caller behavior is not evidence, and an internal interface with an established
   contract is not a boundary.
4. Report each problem once, with every rule it breaks.

## Report

Use `reference/report-format.md`. Always include the header (scope, checks run,
checks skipped and why), and write "No findings" explicitly when the table is empty.

## Never modify

Do not edit source, configuration, lock files, or dependencies during a review. If
the user wants fixes applied, that is a separate request after the report.
