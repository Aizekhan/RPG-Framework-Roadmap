# Audit — Core/DI

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/DI

## Name

Core/DI

## 1. Is it needed by any RPG?

Yes. Dependency injection is broadly reusable Framework infrastructure for modular RPG projects, though a specific RPG can choose another composition mechanism.

## 2. Does it depend on a specific game?

The DI concepts are game-agnostic, but the implementation currently depends on MythHunter logging. Several DI helper APIs also introduce framework-wide lifecycle and scope concepts that need tighter semantics.

## 3. Can it be reused without changes?

Not safely in its current state.

The basic registration/resolution model is reusable. However, scoped resolution, reflection-based construction and some factory paths appear incomplete or potentially incorrect.

## 4. Framework or Game Layer?

**Framework.**

DI should be part of the reusable infrastructure layer and must not depend on gameplay modules.

## 5. Are there architectural problems?

- DIContainer is very large and owns registration, resolution, lifetimes, scopes, reflection-based construction, injection and dependency analysis.
- DIContainer directly depends on IMythLogger, coupling the DI kernel to another Framework service.
- Scoped resolution is suspicious: ResolveByType creates `currentScope.GetOrCreateInstance<object>()` instead of requesting the actual service type, which can break scoped resolution semantics.
- ResolveLazy uses reflection to access the generic LazyDependency.Value instead of using a strongly typed path.
- Constructor/method/field/property injection is exposed through a broad InjectAttribute, increasing reflection complexity.
- Reflection and expression-based factory generation introduce startup/runtime complexity that should be isolated and measured.
- DIContainer automatically registers itself as IDIContainer, which is convenient but hardcodes container self-registration policy.
- DIScope stores a parent scope but does not visibly implement parent lookup/fallback semantics.
- Scoped disposal order is based on HashSet iteration rather than deterministic creation/dependency order.
- DILifecycleManager tracks scopes and shutdown callbacks but depends directly on UnityEngine without an evident use in the inspected implementation.
- There are overlapping factory concepts: DependencyFactory, DIFactory and ActivatorFactory. The parameterized factory methods are effectively placeholders and do not honor supplied parameters.
- DIInstaller contains logging behavior and therefore couples composition infrastructure to the logging implementation.
- There is no evidence in this top-level audit of circular dependency detection during resolution; AnalyzeDependencies exists but needs detailed inspection.

## 6. What does it depend on?

Observed:
- System reflection/expression/threading/collections
- MythHunter.Utils.Logging
- UnityEngine in DILifecycleManager

The DI interfaces themselves have minimal dependencies.

## 7. Who depends on it?

Large portions of MythHunter depend on Core.DI through constructors, [Inject] attributes, installers and service resolution. It is a foundational dependency for ECS, Systems, UI, Networking, Services and other modules.

## 8. Are there unnecessary dependencies?

Yes.

Primary candidates:
- DI kernel -> MythLogger
- DILifecycleManager -> UnityEngine
- DI installer -> logging implementation
- duplicate factory abstractions

These should be reduced or moved behind optional adapters.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split the container into focused responsibilities: registration, resolution, lifetime/scope management and injection.
- Fix and test scoped lifetime semantics before reuse.
- Define deterministic parent-scope behavior and disposal order.
- Consolidate factory abstractions into one coherent API.
- Make parameterized construction actually honor parameters or remove the API.
- Move logging out of the DI kernel or depend on a minimal diagnostics abstraction.
- Isolate reflection/expression compilation and add cycle/missing-dependency diagnostics.
- Remove Unity dependency from generic DI lifecycle management.
- Add tests for singleton/transient/scoped/lazy lifetimes, constructor/field/method injection and nested scopes.

## Conclusion

- [x] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: **Framework**. DI is a core universal subsystem, but the current implementation is overextended and contains at least one serious scoped-resolution design issue plus duplicated factory abstractions and infrastructure coupling.
