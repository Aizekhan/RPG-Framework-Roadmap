# EPIC 05.2 — DI Behavior Characterization and Safe Extraction Plan

## Status
**ACTIVE — source audit complete; next slice is focused DI regression tests.** Audit: `Audits/EPIC_05.2_DI_Candidate_Audit.md`.

## Goal
Make DI behavior explicit and testable before introducing a platform-neutral Framework DI runtime. Preserve current MythHunter-facing APIs until characterization proves a safe migration path.

## Scope for first implementation PR
Add focused tests and fix only verified DI correctness defects in the existing Game Layer implementation. Do not create a Framework DI assembly in the first PR, do not change all consumers, and do not move Unity/installer policy.

## Current implementation
- Draft PR #21: https://github.com/Aizekhan/MythHunter/pull/21
- Branch/head: `feature/epic-05-2-di-characterization` / `d5ab5f5fe4101f0e3d6b111b347649d8e07f4272`.
- Eight EditMode tests were added under `Assets/_MythHunter/Tests/DI`.
- Bounded fixes change scoped cache key to the requested service type, make `Resolve(Type)` return an instance, and align `IsRegistered(Type)` with the generic API for registered instances/lazies.
- CI #44 passes Framework/static/.NET checks; #242 is a successful no-op. These do not validate the Unity DI assembly.
- **Next gate:** run Unity EditMode on this branch, inspect each named test in XML, verify full project compile/Console and bootstrap smoke before marking PR ready/merging.

## Test matrix
1. Generic singleton registration: repeated resolution returns the same reference.
2. Transient registration: each resolution returns a distinct reference.
3. Scoped registration: repeated resolutions inside one scope return the same requested service; different scopes return distinct instances.
4. Nested scopes: explicitly decide whether parent-scoped instances are visible in a child scope and test that rule.
5. Instance registration: both generic and Type-based APIs report the service as registered and return the same instance.
6. Type-based `Resolve(Type)`: returns a service instance, not internal registration metadata.
7. Lazy singleton: factory runs only on first value access; subsequent reads return the same object; failure/retry semantics are intentional.
8. Missing registration and constructor failures: errors are deterministic and retain useful root exception context.
9. Disposal: scoped `IDisposable` instances are disposed exactly once; define explicit transient/singleton ownership rather than assuming current behavior.
10. Scope disposal: accessing a disposed scope fails predictably; child-scope disposal does not accidentally dispose parent-owned values.
11. Thread/current-scope selection: document and test scope isolation; do not assume `ThreadLocal<T>` flows safely across async continuations.

## Acceptance criteria
- Regression tests fail against any confirmed broken current behavior and pass after the bounded fix.
- Existing public interface and the existing consumer call sites stay source-compatible in this slice.
- No speculative architecture rewrite or Framework assembly creation.
- Unity full project compilation, EditMode tests and game bootstrap smoke are reported with actual evidence.
- The audit documents any intentionally deferred semantics and a rollback plan.

## Rollback
Revert only the bounded DI behavior-fix PR. Do not reset, clean, or discard local worktree changes. Avoid source edits until the characterization cases and their expected semantics are clear.
