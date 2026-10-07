# Power of 10 in TypeScript

Everything in `javascript.md` applies. This file adds what the type system changes.

**1. Control flow.** Discriminated unions with an exhaustive `switch` replace
nested `if` chains; add `default: assertNever(x)` so a new variant fails to compile.

**3. Resources.** `using` declarations give files and connections an explicit
release point where the project's runtime supports them.

**5. Validation.** Types are erased at runtime. A value typed `Config` is still
`unknown` when it came from `JSON.parse`, a request body, or `process.env`.
Validate it at the edge and let the type follow from the check. `as` casts and `!`
assertions silence the compiler without checking anything.

```ts
const cfg = parseConfig(JSON.parse(text)); // throws naming the failing field
```

**6. Scope.** Mark fields `readonly` when they are set once.

**7. Errors.** `void somePromise()` discards the rejection; it is not handling.
Catch variables are `unknown` under `useUnknownInCatchVariables`; narrow before
reading `err.message`.

**8. Explicitness.** Generics and union types over `any` and over runtime
reflection. Decorators are fine where the framework uses them.

**9. Checks.** Run `tsc` (see `tooling.md` for the safe form) and the configured
`eslint`. Do not loosen `tsconfig.json` to pass. `// @ts-ignore` hides an error
without a check; `// @ts-expect-error` with a reason is acceptable for a confirmed
compiler limitation.

**10. Simplicity.** A plain interface over a conditional mapped type; a string
union over an enum.

## Linter checks (typescript-eslint; "typed" needs type information)

| Rule | Checks |
|---|---|
| 1 | `switch-exhaustiveness-check` (typed) |
| 5 | `no-non-null-assertion`, `no-explicit-any`, `no-unsafe-*` (typed) |
| 7 | `no-floating-promises` (typed), `no-misused-promises` (typed) |
| 9 | `ban-ts-comment` |
