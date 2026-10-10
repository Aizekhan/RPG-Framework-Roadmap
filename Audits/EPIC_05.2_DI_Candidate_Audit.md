# EPIC 05.2 — Dependency Injection Candidate Audit

## Status
**SOURCE AUDIT COMPLETE — do not extract the current DI interfaces as-is.** Audited MythHunter source at `dev` commit `d3b81484f42d084cae150b5b60fc64185cfb70c2`. Logging has already been merged separately in PR #20.

## Current public surface and dependencies
- `IDIContainer` exposes registration (transient/scoped/singleton/lazy), instance binding, generic and Type-based resolution, registration checks, dependency injection/analysis, and direct access to `DIScope` and `LazyDependency<T>`.
- `IDIContainer` depends on MythHunter-owned concrete concepts: `DIScope`, `LazyDependency<T>`, reflection injection attributes and behavior. It is not a clean Framework contract.
- `DIContainer` constructor and fields depend on `IMythLogger`; factory construction uses reflection/expression trees and `InjectAttribute`.
- `IDIInstaller.InstallBindings(IDIContainer)`, `DIInstaller`, installer registries and multiple Game Layer installers bind concrete game services. `DIContainerExtensions` also imports UnityEngine and `IMythLogger`.
- `DILifecycleManager` depends on the current `DIScope` and `IMythLogger`, tracks scopes/shutdown handlers and is not independent of the current container.

## Confirmed usage and compatibility
Search shows `IDIContainer` consumed by GameBootstrapper, installers, states, UI, system registry, factories, lifecycle manager, DI extensions and diagnostics. `DIScope` / `LazyDependency<T>` are in the public container API and have direct current usage. This is a broadly coupled Game Layer service; do not rename or migrate all callers as part of the first extraction.

## Correctness risks to address before a Framework extraction
1. **Scoped resolution looks incorrect:** `DIContainer.ResolveByType` calls `currentScope.GetOrCreateInstance<object>()` for a scoped registration. That keys the scope by `object`, and `DIScope.GetOrCreateInstance<T>` calls back into `container.Resolve<T>()`; this risks recursion / resolving an unregistered `object` instead of the requested service.
2. **Type-based resolve returns the registration record:** `IDIContainer.Resolve(Type)` in `DIContainer` returns `_registrations[type]`, which appears to be the internal `Registration` object rather than the resolved instance.
3. **Type-based registration check is inconsistent:** `IsRegistered(Type)` checks only `_registrations`, while generic `IsRegistered<T>()` also counts singleton and lazy maps.
4. **Scope hierarchy semantics are unclear:** `DIScope.ParentScope` is stored, but lookup does not consult the parent. Scope disposal captures `IDisposable` values but there is no explicit container policy for singleton/transient disposal.
5. **Concurrency and current scope need a design:** `DIContainer` has both `_currentScope` and `ThreadLocal<DIScope> _asyncLocalScope`; the inspected implementation does not show a clear unified selection policy. The name async-local is also not equivalent to thread-local for asynchronous flows.
6. **Game policy is mixed into container:** constructor injection selection, field/property/method injection, logging, Unity component helpers, GameObject creation, installer order and boot validation are all coupled to MythHunter/Unity.

These findings are static source inspection. They are not yet confirmed by failing tests. Treat them as high-priority cases for focused characterization/regression tests before changing behavior.

## Decision
Do not move `IDIContainer`, `DIScope`, `LazyDependency<T>` or the whole DI implementation directly into Framework. First establish behavior with focused tests, beginning with registration/resolution and scoped lifetimes. Separate a platform-neutral registration/resolution kernel from MythHunter adapters only after the core semantics (lifetime, scopes, disposal, injection policy, errors and concurrency) are explicit. Keep Unity `MonoBehaviour` binding and scene injection in the Game Layer. Keep installer orchestration outside the DI kernel.

## Proposed execution order
1. Characterization tests for singleton, transient, scoped, lazy, Type-based resolution and disposal.
2. Fix proven correctness bugs in a bounded MythHunter DI PR, without changing public caller APIs.
3. Define the minimal framework DI contract based on tested behavior, deliberately excluding Unity/game installer concerns.
4. Add the pure runtime assembly and tests, then an adapter/compatibility facade and migrate consumers in a separate PR.
5. Validate Unity EditMode, game bootstrap smoke, CI, and rollback path at every migration boundary.

## Acceptance gates
- No Framework DI runtime references Unity, MythHunter, `IMythLogger`, or Unity injection helpers.
- Lifetime and scope semantics are documented and tested, including failure cases and nested scopes.
- Type-based resolve/check APIs behave consistently with generic API.
- Disposal ownership for transient/scoped/singleton instances is explicit.
- Existing Game Layer consumers remain compatible until the separate migration gate passes.
- Do not create a Framework DI assembly until tests and minimal contract design are accepted.
