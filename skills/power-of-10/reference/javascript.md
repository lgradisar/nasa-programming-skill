# Power of 10 in JavaScript

**1. Control flow.** No labeled `break`/`continue` across blocks. Recursion over
user data can overflow the stack; iterate with an explicit stack. Prefer
`async`/`await` over nested callbacks.

**2. Loops.** Polling loops need a deadline from `performance.now()` (monotonic;
`Date.now()` moves with the wall clock) or an `AbortSignal`.

```js
const deadline = performance.now() + timeoutMs;
while (!job.done) {
  if (performance.now() > deadline) throw new Error(`job ${job.id} timed out`);
  await sleep(1000);
}
```

**3. Resources.** Every `addEventListener`, `setInterval`, subscription, and open
stream has a matching removal, usually a returned stop function. A `Map` used as a
cache needs a bound. Limit concurrency with a pool instead of `Promise.all` over
thousands of requests.

**4. Functions.** One job per function.

**5. Validation.** Validate request bodies, query strings, environment variables,
and `JSON.parse` results at the edge with a clear error. `console.assert` does not
throw; use an explicit `throw` for invariants.

```js
function checkPort(port) {
  if (!Number.isInteger(port) || port < 1 || port > 65535) {
    throw new RangeError(`port must be an integer 1-65535, got ${port}`);
  }
  return port;
}
```

**6. Scope.** `const` by default, `let` when reassigned, never `var`. No implicit
globals; no module-level object mutated from many places.

**7. Errors.** Every promise is awaited, returned, or given a `.catch`; discarding a
promise does not handle its rejection. No empty `catch`. Rethrow with `cause` when
you cannot handle it. `fetch` resolves on HTTP errors; check `response.ok`.

```js
try {
  return parse(x);
} catch (err) {
  throw new Error(`invalid config in ${path}`, { cause: err });
}
```

**8. Explicitness.** No `eval`, `new Function`, or string arguments to `setTimeout`.
No `child_process.exec` with a built string; use `execFile` with an argument array,
or the `fs` API when that is the real job. A plain object map beats `obj[name]()`
built from input.

**9. Checks.** Run the configured `eslint` without `--fix`. An `eslint-disable`
comment names the rule and the reason.

**10. Simplicity.** A function over a one-method class; `for...of` over a
`.map().filter().reduce()` chain that needs comments to follow.

**11. Comments.** Delete commented-out code. JSDoc stays where the project uses it
for types. A comment says why: `// Safari fires 'resize' twice; debounce`.

## Linter checks (ESLint)

| Rule | Checks |
|---|---|
| 1 | `no-labels`, `max-depth`, `complexity` |
| 2 | `no-unmodified-loop-condition` |
| 4 | `max-lines-per-function`, `max-params` |
| 6 | `no-var`, `prefer-const`, `no-implicit-globals`, `no-global-assign` |
| 7 | `no-empty`, `promise/catch-or-return` (eslint-plugin-promise) |
| 8 | `no-eval`, `no-implied-eval`, `no-new-func` |
| 9 | `eslint-comments/no-unlimited-disable` |
