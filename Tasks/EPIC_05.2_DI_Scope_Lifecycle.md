# EPIC 05.2 — Scope Lifecycle and Disposal Semantics

## Status
**ACTIVE — scope lifecycle implementation in draft PR #22; Unity validation pending.** Prior PR #21 is merged: `f57cebb9e0dba11c3e6fbeefc8620627a9363428`.

## Current PR
- PR #22: https://github.com/Aizekhan/MythHunter/pull/22
- Head: `a48002f8e2e2cd7690f4f338796dcad2288c5a3f`.
- CI boundary/.NET passed; Minimal CI is a no-op. Unity is still required.
- Added tests for worker/main thread scope isolation, independent child scope, reverse disposal and resilient cleanup, and exactly-once cleanup of a shared reference.

## Goal
Make current-scope selection, nested scope behavior and resource ownership explicit. Preserve existing APIs while fixing only behavior established by regression tests.

## Source evidence
- `DIContainer.SetCurrentScope` currently compares `Thread.CurrentThread.ManagedThreadId` to itself, so the condition is always true and the thread-local branch is unreachable.
- The field named `_asyncLocalScope` is actually `ThreadLocal<DIScope>`; it does not express async-flow semantics.
- `CreateScope` stores the current scope as parent; `DIScope.ParentScope` is exposed but `GetOrCreateInstance<T>` does not search parents.
- Scope disposal iterates a `HashSet<IDisposable>`, so disposal order is unspecified; exceptions are swallowed after a debug write.
- Scope-managed values are disposed by the scope. Ownership/disposal policy for externally registered instances, singletons and transient instances is currently not consistently explicit.

## Decisions required before implementation
1. Current scope: do we intend a single Unity-main-thread active scope, per-thread scope, or async-flow scope? For current use, do not silently replace `ThreadLocal` with `AsyncLocal` until actual async DI use is mapped.
2. Child visibility: choose whether child scopes inherit resolution of parent-scoped instances, or whether each scope gets an independent instance for every scoped service.
3. Ownership: distinguish factory-created scope-owned instances from externally supplied instances; do not dispose an externally registered instance unless explicitly opted in.
4. Disposal: define deterministic order (recommended reverse creation order), idempotency, and best-effort behavior if one `Dispose` throws.
5. Transient/singleton disposal: do not automatically dispose transient/singleton results until ownership and container shutdown semantics are specified.

## Implemented and still-required tests
- Switching scope actually changes the current scope and scoped service instance.
- Returning/restoring the previous scope returns the expected prior scoped instance.
- Child scope follows the chosen parent-visibility rule and does not dispose resources owned by the parent.
- Scoped `IDisposable` values are disposed once; disposal order is deterministic.
- One throwing `Dispose` does not prevent other scope-owned disposables from being disposed.
- Disposing a scope twice is harmless; resolving/using a disposed scope fails deterministically.
- Registering an external instance does not silently transfer disposal ownership.
- Tests for thread/async context only after intended context semantics are written down.

## Acceptance criteria
- Add focused Unity EditMode tests and prove they fail on the relevant old behavior where practical.
- Fix the self-comparison error without introducing a false main-thread identity assumption.
- The chosen child-scope/ownership semantics are documented and tested.
- Full Unity compile, EditMode XML, and game bootstrap/scope smoke test pass.
- No API-wide rewrite and no Framework DI assembly created in this slice.
- Keep the PR draft until Unity validation passes.

## Rollback
Revert only this bounded scope lifecycle PR. Do not reset, clean, or discard local worktree changes.
