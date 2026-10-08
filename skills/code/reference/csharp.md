# Power of 10 in C#

**1. Control flow.** No `goto`, including `goto case`. Recursion over user-supplied
trees can overflow the stack, and `StackOverflowException` cannot be caught;
iterate with an explicit stack. Pattern matching and early returns flatten nesting.

**2. Loops.** Polling loops take a `CancellationToken` and pass it to every awaited
call and delegate inside. When the caller gives none, create a
`CancellationTokenSource` with `CancelAfter` from a requirement-based timeout.
`Task.WaitAsync(timeout)` only bounds the caller's wait; the loop behind it keeps
running.

```csharp
while (!isDone(ct))
{
    await Task.Delay(1000, ct);
}
```

**3. Resources.** Every `IDisposable` has one owner: a `using`, a type that disposes
it, or the caller when a factory hands it over (dispose on failure before the
hand-over). `HttpClient` is shared, not created per request. Caches need a size
limit or eviction. Bound parallelism with `SemaphoreSlim` or
`MaxDegreeOfParallelism`.

**4. Functions.** About 60 lines is the signal.

**5. Validation.** `Debug.Assert` is compiled out in Release; it is for invariants
only. Validate arguments of externally callable APIs and all deserialized input with
`ArgumentNullException.ThrowIfNull`, `ArgumentOutOfRangeException.ThrowIf*`, or a
clear `throw`. Nullable reference types help inside your code; input from outside is
still unchecked.

**6. Scope.** No public mutable static fields; `static readonly` or `const` for
shared constants. `private` and `sealed` by default; get-only or `init` properties
where a value is set once.

**7. Errors.** No `catch (Exception)` that swallows; catch the specific type or
rethrow with `throw;`. Every `Task` has an owner that awaits it or otherwise
observes its failure. `TryParse`
results must be checked. At an isolation boundary, log the item and the exception,
then continue.

**8. Explicitness.** No `dynamic` where the type is known. Reflection only where
the framework needs it. No SQL or shell commands built from strings: parameterize
queries, use `ProcessStartInfo.ArgumentList`.

**9. Checks.** `dotnet build` runs the analyzers (see `tooling.md` for the safe
form). `#pragma warning disable` needs the rule id and a reason, scoped to the line.
Do not lower `AnalysisLevel` to pass.

**10. Simplicity.** A record over a class with equality boilerplate; a static method
over a one-method service unless it is injected; LINQ when it reads as a sentence.

**11. Comments.** XML doc comments stay where the project publishes them. Delete
commented-out code. A comment states why: `// the vendor API rejects ISO dates`.

## Analyzer checks (.NET analyzers; `S` codes need SonarAnalyzer)

| Rule | Checks |
|---|---|
| 1 | S907 (`goto`), S134 (nesting), CA1502 |
| 3 | CA2000, CA1001, CA2213 |
| 4 | S138 |
| 5 | CA1062 |
| 6 | CA2211, CA1051, IDE0044 |
| 7 | CA1031, CA2200, CS4014 (only some unawaited calls), S108 |
| 8 | CA2100 |
