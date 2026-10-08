# Power of 10 in Python

**1. Control flow.** Python has no `goto`. Recursion over user data can hit the
interpreter's depth limit; iterate with an explicit stack. Flatten nesting with
guard clauses.

**2. Loops.** `while True` is fine when the `break` is near and obvious. Polling
loops need a deadline built from `time.monotonic()`, not `time.time()`; retries
count attempts.

```python
deadline = time.monotonic() + timeout
while not job.done():
    if time.monotonic() > deadline:
        raise TimeoutError(f"job {job.id} did not finish in {timeout}s")
    time.sleep(1)
```

**3. Resources.** Use `with` for files, sockets, and locks, and check what a context
manager actually releases: `sqlite3.Connection`'s commits or rolls back but does
not close. Caches (`lru_cache(maxsize=None)`, a module-level dict) need a bound.
Prefer generators over whole lists for large data. In `asyncio`, every task has an
owner that awaits or cancels it.

**4. Functions.** One job per function. Many parameters are a signal that the
function does several.

**5. Validation.** `assert` is removed under `python -O`, so it must never check
input. Validate with `if` and raise `ValueError`/`TypeError` naming the field.

```python
def parse_port(text):
    port = int(text)
    if not 0 < port < 65536:
        raise ValueError(f"port must be 1-65535, got {port}")
    return port
```

**6. Scope.** No `global` statement; no module-level mutable state that functions
modify. `UPPER_CASE` constants are fine. Mutable default arguments are shared across
calls.

**7. Errors.** No bare `except:`, no `except Exception: pass`. Catch the narrowest
type you can handle; otherwise let it propagate. Re-raise with `raise ... from err`.
`subprocess.run` needs `check=True` or an explicit return-code check. Every
`asyncio` task has an owner that awaits it or otherwise observes its failure. At an
isolation boundary, log the item and the error, then continue.

**8. Explicitness.** No `eval`/`exec` on input; use `json` for JSON, and
`ast.literal_eval` only for trusted Python literals.
No `shell=True` with a built string; pass an argument list. A plain call beats
`getattr(obj, name)()` built from strings; plain classes beat metaclasses unless a
framework asks.

**9. Checks.** Run the configured `ruff`, `pylint`, or `mypy`. A `# noqa` needs a
code and a reason.

**10. Simplicity.** A `for` loop over a clever comprehension with side effects; a
dataclass over a hand-written `__init__`; a function over a one-method class.

**11. Comments.** Keep docstrings where the project uses them. Delete commented-out
code. A comment says why: `# retry once: the API returns 503 during deploys`.

## Linter checks (ruff codes)

| Rule | Checks |
|---|---|
| 1 | C901, PLR1702 |
| 3 | SIM115, B019, RUF006 |
| 4 | PLR0913, PLR0915 |
| 5 | S101 (judge each) |
| 6 | PLW0603, B006 |
| 7 | E722, BLE001, S110, S112, B904, PLW1510 |
| 8 | S102, S307, S602, S604 |
| 9 | PGH004 |
| 11 | ERA001 |
