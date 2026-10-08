---
name: code
description: Engineering rules adapted from NASA/JPL's "Power of 10" for everyday code. Use whenever writing, editing, or refactoring source code in any language (Python, JavaScript, TypeScript, C, C#, Go, or others). Covers control flow, loop termination, resource ownership, function size, input validation, scope, error handling, explicit over dynamic code, warnings, simplicity, and comments. Not for documentation-only or configuration-only changes.
---

# Power of 10 for everyday code

Eleven rules, generalized from NASA/JPL's safety-critical C rules so they fit
ordinary software.

## Application policy

Correctness and the project's existing contracts come first. Apply these rules to
the code being written or reviewed, in proportion to the change. Do not refactor
unrelated code, invent limits, or add abstractions only to satisfy a rule. Follow
established project patterns when they meet the rule's intent. When a rule needs a
number (a timeout, a limit, a retry budget) and the requirements do not give one, do
not pick one: make it a required parameter or read it from configuration, and say
so in your explanation to the user, not in the code. An exception needs a one-line
reason there too; add a source comment only when a future reader needs it.

## The rules

**1. Straightforward control flow.**
No `goto`, except structured cleanup jumps in C. Recursion only when the depth is
known and small; otherwise iterate. Deep nesting is a smell, not a hard limit; flatten
it with early returns when that reads better.

**2. Every loop terminates visibly.**
A loop doing finite work must make clear progress toward its exit. A loop waiting on
the outside world (network, user, another process) needs a deadline, a retry budget,
or a cancellation signal, supplied by the caller or configuration rather than a
default you choose. Long-lived service loops (event loops, workers) are allowed
and must have a shutdown path. Never truncate results silently to fake a bound.

**3. Predictable resources.**
Every resource (memory, file, socket, lock, task, subscription) has one clear owner
and a release point. Add limits or backpressure where load can exceed capacity.
Garbage collection does not remove this duty.

**4. Functions are cohesive and readable in one pass.**
A function does one thing at one level of detail. About 60 lines is a review signal,
not a cutoff. Do not extract helpers only to shrink a number.

**5. Validate at the boundary; assert invariants inside.**
External input (user, network, file, environment, other services) gets runtime
checks that produce a clear error at the boundary. Assertions are for internal
invariants only: side-effect-free, and never used for input validation, because they
can be compiled out. "Fail loudly" means an observable, appropriate error, not
necessarily crashing the process.

**6. Smallest possible scope.**
Declare data where it is used. Avoid uncontrolled shared mutable state. Shared state
that must exist has one owner, a clear lifecycle, and synchronization if it is
touched concurrently. Module-level constants and framework-managed registries are
fine.

**7. Check every result that can fail.**
Never swallow a failure silently or continue with invalid state. Recover only where
recovery is meaningful; otherwise propagate the error to the boundary with its
context intact and cleanup done. Continuing after a failure is allowed at a deliberate
isolation boundary (one bad record in a batch, one failed job in a worker) when the
failure is reported and cleanup is complete. Discarding a non-essential result is
fine; ignoring a failure-bearing result needs a stated reason.

**8. Explicit over dynamic.**
Never execute untrusted data as code: no `eval`/`exec` on input, no `new Function`,
no shell commands built from strings. Deliberate code execution in an interpreter or
developer tool is a design decision, not a violation. Avoid reflection,
metaprogramming, and indirection when a plain call does the job. Established
framework mechanisms (decorators, dependency injection, ORMs) are fine.

**9. Clean under the project's own checks.**
Introduce no new warnings under the project's existing compiler and linter settings.
Fix warnings rather than hide them; a narrow suppression of a confirmed false positive
is allowed if the check stays useful. Report pre-existing warnings; do not fix
unrelated ones.

**10. Prefer simplicity over complexity.**
Simple means fewer concepts to hold in the head and behavior that is easy to predict.
It does not mean fewer lines. This is the governing principle and the tie-breaker.

**11. Code is readable without comments.**
Names and structure carry the meaning. Comment only what the code cannot say: a
design decision, a non-obvious constraint, a workaround. Default to one line. A
longer comment is allowed when the reasoning truly needs it (concurrency, security,
protocol, numerical, or compatibility constraints). Keep required docstrings, license
notices, and tool directives.

## Language-specific guidance

Decide the language of each file you edit from its extension and the project around
it, and read the matching reference. In a mixed project, read one per language you
touch.

| Extension | Reference |
|---|---|
| `.py` | `reference/python.md` |
| `.js`, `.mjs`, `.cjs`, `.jsx` | `reference/javascript.md` |
| `.ts`, `.tsx` | `reference/javascript.md`, then `reference/typescript.md` |
| `.c`, `.h` in a C project | `reference/c.md` |
| `.cs` | `reference/csharp.md` |
| `.go` | `reference/go.md` |

Any other language: the general rules only.

## Verifying your change

If the project has configured check commands (linter, type checker, compiler
warnings, tests), run them as `reference/tooling.md` says and fix the findings your
change introduced. Never install or download anything. If the change alters
behavior, run the relevant existing tests; a clean lint run alone does not prove
correctness. If the project has no tooling, say so and move on.
