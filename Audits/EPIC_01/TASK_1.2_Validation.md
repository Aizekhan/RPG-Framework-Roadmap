# Audit — Core/Validation

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/Validation

## Name

Core/Validation

## 1. Is it needed by any RPG?

Validation is broadly useful in RPG Framework infrastructure for configuration, dependency graphs, data integrity and runtime contracts.

## 2. Does it depend on a specific game?

The validation concept is reusable, but the current implementation is strongly tied to MythHunter, Unity and the custom DI system.

DIValidator specifically depends on GameBootstrapper and scene MonoBehaviours.
DependencyValidator scans assemblies whose names contain "MythHunter".

## 3. Can it be reused without changes?

No.

The useful idea is reusable, but current validators contain project-specific discovery and Unity runtime assumptions.

## 4. Framework or Game Layer?

**Mixed, with a Framework candidate underneath.**

Generic validation primitives can belong to Framework. These concrete DI validators should be treated as development/application tooling.

## 5. Are there architectural problems?

- DIValidator is a static RuntimeInitializeOnLoadMethod utility that scans the scene after load.
- DIValidator's HasInjectAttributes currently returns true unconditionally, so it is effectively a placeholder and can produce false warnings.
- DIValidator depends on GameBootstrapper, making validation dependent on a specific application composition root.
- DependencyValidator scans only assemblies containing "MythHunter", making it impossible to reuse as a generic validator.
- DependencyValidator eagerly reflects over every matching type and all constructors/fields/properties/methods, which can be expensive and fragile.
- It treats every class implementing an interface as a validation target, which may include types that should never be container-managed.
- Constructor fallback to the first constructor can produce false dependency errors for types intentionally constructed through another mechanism.
- IsRegistered is invoked through reflection even though IDIContainer already exposes a Type-based overload.
- Validation relies on current container registrations, so results can depend on installer execution order and partially built application state.
- ValidationIssue is mutable and nested inside DependencyValidator.
- There is no clear distinction between Framework validation, editor-time validation and runtime diagnostics.

## 6. What does it depend on?

Observed:
- System.Reflection / LINQ
- MythHunter.Core.DI
- MythHunter.Utils.Logging
- UnityEngine
- MythHunter.Core.Game
- MythHunter.Core.MonoBehaviours
- MythHunter.Utils.UnityApiUtils

The generic validation concept itself could have minimal dependencies.

## 7. Who depends on it?

- DIExtensionsInstaller invokes DependencyValidator during editor validation.
- The runtime DI validator is invoked automatically after scene load.
- DIContainer provides the registrations being inspected.

## 8. Are there unnecessary dependencies?

Yes.

- Validation -> GameBootstrapper
- Validation -> MythHunter assembly naming
- Validation -> Unity scene scanning
- Reflection through IDIContainer instead of direct type-based API

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split validation into generic validation primitives, DI graph validation and Unity/editor diagnostics.
- Make assembly/type discovery injectable instead of hardcoded to "MythHunter".
- Validate only explicitly registered or annotated services.
- Use IDIContainer.IsRegistered(Type) directly.
- Remove placeholder logic such as HasInjectAttributes always returning true.
- Separate editor-time validation from runtime validation.
- Produce structured immutable validation results.
- Add explicit cycle, missing registration and lifetime validation once DI semantics are stabilized.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Mixed**. Validation is valuable Framework infrastructure, but the current Core/Validation implementation is primarily MythHunter-specific DI diagnostics and development tooling with several placeholder and reflection-heavy behaviors.
