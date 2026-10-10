# EPIC 04 — Core Contract Candidate Audit

## Status
AUDIT COMPLETE — no new shared `RPGFramework.Contracts` assembly justified by current source evidence. The canonical ECS contract/runtime is already established in `RPGFramework.ECS.Runtime`; remaining candidates either carry MythHunter policy/dependencies or do not yet have demonstrated cross-module consumers. Do not create a placeholder assembly.

## Source baseline
Reviewed current `dev` after PR #18 merge (`b6e046895938d2dd72026827c4f72fd44ced1fdf`) and after EPIC 03.6 was merged (`d3b81484f42d084cae150b5b60fc64185cfb70c2`). The only Framework-owned runtime files currently are:

- `Assets/_Framework/ECS/Runtime/IComponent.cs`
- `Assets/_Framework/ECS/Runtime/IEntityManager.cs`
- `Assets/_Framework/ECS/Runtime/EntityManager.cs`

The Framework ECS assembly has no references and sets `noEngineReferences: true`. Migration CI now verifies the boundary and absence of duplicate legacy ECS contracts.

## Candidate matrix

| Candidate | Observed source and coupling | Decision |
|---|---|---|
| ECS `IComponent`, `IEntityManager` | `Assets/_Framework/ECS/Runtime/`; consumers migrated in EPIC 03.6 | Canonical owner already established. Keep there; do not duplicate into a generic Contracts assembly. |
| `IDIContainer`, `IDIInstaller`, `IDILifecycleManager` | `Assets/_MythHunter/Code/Core/DI/`; container API exposes MythHunter `DIScope`, `LazyDependency<T>` and registration/injection/lifecycle policy | Not neutral as-is. Keep Game Layer until a bounded DI design/extraction in EPIC 05.2. |
| `IEvent`, `IEventBus`, `IEventPool` | `Assets/_MythHunter/Code/Events/`; `IEvent` uses MythHunter `EventPriority`; `IEventBus` imports Cysharp `UniTask` and requires struct events | Not neutral as-is. Event semantics, async abstraction and scheduling policy need an explicit EPIC 05.3 decision. |
| `IComponentSerializer<T>`, `ISerializable` | `Assets/_MythHunter/Code/Data/Serialization/`; component serializer is constrained to Framework ECS components; registry/implementations are game/content-specific | Keep serializers in the current layer until EPIC 06 maps schema/versioning and actual use. `ISerializable` has not been demonstrated as a shared cross-module contract. |
| `IValidator<T>` / `ValidationResult` | `Assets/_MythHunter/Code/Utils/Validation/IValidator.cs` and `Validator.cs`; result type is local to MythHunter validation namespace and returns mutable `List<string>` | Candidate for a later neutral validation API, but first verify callers and define result semantics under EPIC 05.1. Do not move only the interface and leave its result behind. |
| `IMythLogger` / `LogLevel` | `Assets/_MythHunter/Code/Utils/Logging/IMythLogger.cs`; explicitly MythHunter-named and includes category/context/file logging controls | Not a Framework contract as-is. EPIC 05.1 should define the minimal neutral logging API, then retain an adapter/compatibility facade for MythHunter. |
| `IPrioritizable` | `Assets/_MythHunter/Code/Resources/Core/IPrioritizable.cs`; only an integer `Priority` property is visible in the contract | Too small to justify shared assembly without proven consumers outside Resources. Keep local until a concrete second consumer exists. |

## Decisions / architecture rules
1. Do not create `RPGFramework.Contracts` as an empty or speculative assembly. Add one only when at least two real modules need the same neutral contract and the dependency graph requires a shared owner.
2. Keep `RPGFramework.ECS.Runtime` the sole owner of the ECS contracts. Do not bring Game Layer lifecycle, `ISystemRegistry`, `EcsWorld`, logger, DI, events or serializers into the ECS assembly.
3. Do not move dormant interfaces simply because their names begin with `I`. Confirm concrete callers/implementations and lifecycle/error semantics first.
4. Exclude `UnityEditor`, Unity/game phases, MythHunter namespaces, providers and presentation from platform-neutral contract modules.
5. Keep module extraction separate from policy/API redesign; compatibility facades must not create duplicate CLR contract identities.

## Next source task
Proceed to EPIC 05.1 — Logging and validation. Start with a caller inventory for `IMythLogger`, its implementation(s), `LogLevel`, `IValidator<T>`, and `ValidationResult`; design a small neutral contract plus MythHunter adapter/facade. Do not edit call sites until the boundary and API compatibility plan are recorded. Add focused tests with the first implementation change.
