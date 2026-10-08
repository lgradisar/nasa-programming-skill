# Power of 10 in C

The original rules were written for C, so most apply directly.

**1. Control flow.** `goto` only as forward jumps to cleanup labels at the end of
the function, in release order. No backward `goto`, no `setjmp`/`longjmp`, no
recursion unless the depth is bounded by a constant you can name.

```c
    if (fread(buf, 1, size, f) != size) goto out_buf;
    rc = 0;
    goto out_file;
out_buf:
    free(buf);
    buf = NULL;
out_file:
    if (fclose(f) != 0 && rc == 0) { free(buf); buf = NULL; rc = -1; }
    out->data = buf;
    return rc;
```

**2. Loops.** A loop over data shows its bound (`i < n`, a pointer walk to a
terminator). A loop waiting on hardware or a socket has an iteration or time limit.
Service loops may run until shutdown and check a visible stop condition.

**3. Resources.** Every `malloc` has one owner: the function frees on every error
path, and on success either frees or hands the buffer to the caller, as its contract
says. Check `malloc` results. No unbounded `realloc` growth without a cap.

**4. Functions.** About 60 lines is the signal.

**5. Validation.** `assert()` is removed under `-DNDEBUG`, which most release
builds set; use it for invariants only. The boundary is where data enters the
program: entry points, parsers of external data, the public surface of a library.
Validate their parameters with `if` and return an error code. Internal functions may
rely on a documented non-NULL contract.

**6. Scope.** Declare at first use, `static` for file-local functions and data,
`const` wherever the value does not change. A non-const global needs one owner and a
reason.

**7. Errors.** Check the return of every function that can fail: `malloc`, `fopen`,
`fread`, `snprintf`, `pthread_*`, system calls. Use `(void)` only for a result that
is truly irrelevant. Mark your own functions `warn_unused_result` when ignoring them
is a bug.

**8. Explicitness.** The preprocessor is for includes, guards, and constants. A
`static inline` function beats a function-like macro; no macro that changes control
flow or hides declarations. No `system()`/`popen()` with a built string. Function
pointers are fine for callbacks and fixed dispatch tables selected by a validated
key; untrusted data must never supply or compute the pointer.

**9. Checks.** Build with the project's warning flags and run `clang-tidy` or
`cppcheck` where configured. A `push`/`ignored`/`pop` pragma around one confirmed
false positive, with a reason, is acceptable; a file-wide `ignored` is not.

**10. Simplicity.** Plain structs and functions over opaque pointers with vtables,
unless the project already uses them.

**11. Comments.** A comment states why: `/* the sensor is big-endian; swap first */`.
Delete `#if 0` blocks.

## Linter checks (clang-tidy unless stated)

| Rule | Checks |
|---|---|
| 1 | `misc-no-recursion`; `cppcoreguidelines-avoid-goto` (also flags cleanup jumps) |
| 2 | `bugprone-infinite-loop` |
| 3 | `clang-analyzer-unix.Malloc`, `clang-analyzer-core.NullDereference` |
| 4 | `readability-function-size` |
| 6 | `cppcoreguidelines-avoid-non-const-global-variables`, `-Wshadow` |
| 7 | `cert-err33-c`, `bugprone-unused-return-value`, `-Wunused-result` |
| 8 | `cert-env33-c`; `cppcoreguidelines-macro-usage` (also flags constants) |
