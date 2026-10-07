# Power of 10 in Go

**1. Control flow.** No `goto`. Recursion over user data grows the stack silently;
iterate with a slice as a stack. Early returns keep the happy path at the left
margin.

**2. Loops.** Every wait takes a `context.Context`: loops select on `ctx.Done()`,
HTTP calls use `http.NewRequestWithContext` with a client that has a `Timeout`,
subprocesses use `exec.CommandContext`. Retries use a counter or a backoff with a
max. `for {}` is fine for a worker loop whose `select` includes `ctx.Done()`.

```go
for !done(ctx) {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case <-time.After(time.Second):
    }
}
```

**3. Resources.** `defer f.Close()` right after a successful open; a written file's
`Close` error is checked. `defer resp.Body.Close()` on every path (discarding the
error is the accepted idiom for a body that was only read). Every goroutine has a
reason to stop; one without an exit is a leak. Bound concurrency with a buffered
channel or `errgroup.SetLimit`. Maps used as caches need eviction.

**4. Functions.** About 60 lines is the signal.

**5. Validation.** Go has no `assert`. Validate at the edge (handlers, flags, file
parsing) and return an error naming the field. `panic` is for programmer error, such
as a bad literal at construction (`regexp.MustCompile`), never for runtime input.

```go
p, err := strconv.Atoi(s)
if err != nil || p < 1 || p > 65535 {
    return 0, fmt.Errorf("port must be 1-65535, got %q", s)
}
```

**6. Scope.** No package-level mutable variables beyond the standard patterns
(`flag` definitions, `sync.Once`). Shorten scope with `if v, err := f(); err != nil`.
Export only what another package needs.

**7. Errors.** Every `err` is checked. `_ = f()` is explicit discarding, not
handling, and needs a reason. Wrap with `fmt.Errorf("loading %s: %w", path, err)`;
compare with `errors.Is`/`errors.As`. Check `resp.StatusCode`. At an isolation
boundary, log the item and continue.

**8. Explicitness.** Plain code and generics over `reflect`; `reflect` only in
encoders and frameworks. `unsafe` needs a stated reason. No
`exec.Command("sh", "-c", built)`; pass arguments separately, or use the `os`
API when that is the real job.

**9. Checks.** `go vet` ships with the toolchain (see `tooling.md` for the offline
form); `staticcheck` or `golangci-lint` where configured. A `//nolint:<linter> //
reason` names both.

**10. Simplicity.** A struct and a function over an interface with one
implementation. Accept interfaces, return structs.

**11. Comments.** Doc comments on exported identifiers start with the name. Delete
commented-out code. Inside functions, a comment says why:
`// the registry returns 429 under load; retry once`.

## Linter checks (golangci-lint linters)

| Rule | Checks |
|---|---|
| 1 | `gocognit`, `gocyclo`, `nestif` |
| 3 | `bodyclose`, `errcheck` |
| 4 | `funlen` |
| 6 | `gochecknoglobals` |
| 7 | `errcheck`, `errorlint`, `go vet` |
| 8 | `gosec` G204 |
| 9 | `nolintlint` |
| 11 | `revive` `exported` |
