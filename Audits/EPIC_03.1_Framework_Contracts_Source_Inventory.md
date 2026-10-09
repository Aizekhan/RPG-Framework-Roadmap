# EPIC 03.1 — Framework Contracts: Source-Grounded Extraction Plan

## Status
DONE (contract inventory and decision plan); implementation of contract extraction is not yet complete.

## Scope
Review the existing public contracts in `Aizekhan/MythHunter` branch `dev` and define the first safe Framework contract slice. This document distinguishes source evidence from target design. It does not claim that the Framework contracts have already been moved or that the Unity project compiles.

## Source snapshot inspected
- Repository: `Aizekhan/MythHunter`
- Branch: `dev`
- Git tree SHA: `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`
- Unity Editor version declared by `ProjectSettings/ProjectVersion.txt`: `6000.0.45f1`
- The source tree contains 358 C# files under `Assets/_MythHunter/Code/`.
- The only discovered `.asmdef` files in this tree are under `Assets/Plugins/UniTask/`; no `.asmdef` was found under `Assets/_MythHunter/Code/`.

These observations come from the repository tree; they do not establish whether the project compiles in a local Unity Editor session.

## Existing contract evidence

### ECS
File: `Assets/_MythHunter/Code/Core/ECS/IComponent.cs`
- Namespace: `MythHunter.Core.ECS`
- It is a marker interface with no members.
- No Unity type is directly used in the file.
- Ownership candidate: Framework ECS contract.

File: `Assets/_MythHunter/Code/Core/ECS/IEntityManager.cs`
- Namespace: `MythHunter.Core.ECS`
- Defines create/destroy entity, add/get/try-get/remove component, and query methods.
- Uses `int` entity IDs and returns allocated arrays for entity/query enumeration.
- Generic component methods constrain against the existing `IComponent`.
- The public method `CreateScope` is not relevant here; the ECS contract should not depend on DI types.
- Ownership candidate: Framework ECS contract, but entity identity and query semantics must be decided before claiming the API is stable.

File: `Assets/_MythHunter/Code/Core/ECS/IEcsWorld.cs`
- Namespace: `MythHunter.Core.ECS`
- Exposes `IEntityManager`, `Initialize()`, `Update(float)`, and `Dispose()`.
- Ownership candidate: Framework ECS/world lifecycle contract.
- Potential overlap with system scheduler lifecycle must be checked.

File: `Assets/_MythHunter/Code/Core/ECS/ISystem.cs`
- Namespace: `MythHunter.Core.ECS`
- Exposes `Initialize()`, `Update(float)`, and `Dispose()`.
- Ownership candidate: Framework system contract.
- The current naming/placement under ECS should be reconsidered; system lifecycle and scheduling must be separate from game phase policy.

### Dependency injection
File: `Assets/_MythHunter/Code/Core/DI/IDIContainer.cs`
- Namespace: `MythHunter.Core.DI`
- Combines registration, singleton/transient/scoped lifetimes, type-based resolution, dependency analysis, property/member injection, lazy resolution and concrete `DIScope` return types.
- It also returns or accepts implementation-oriented DI types such as `LazyDependency<T>` and `DIScope`.
- Ownership candidate: Framework DI abstraction, but not yet a clean minimum contract.
- Required design action: split container registration/resolution from scope/lifecycle details where that removes concrete implementation coupling; resolve whether injection and dependency analysis belong in the stable public API.
- Do not duplicate this interface under a new namespace while existing consumers still compile against it.

### Events
File: `Assets/_MythHunter/Code/Events/IEvent.cs`
- Namespace: `MythHunter.Events`
- Requires each event to expose `GetEventId()` and `GetPriority()`.
- This embeds identity and priority policy into the event contract.
- Ownership candidate: Framework events contract only after deciding whether all events need explicit IDs and priority.
- Do not assume every general-purpose event system requires both members.

File: `Assets/_MythHunter/Code/Events/IEventBus.cs`
- Namespace: `MythHunter.Events`
- Exposes sync and async subscriptions/publication plus `Clear()`.
- The public API directly depends on `Cysharp.Threading.Tasks.UniTask`.
- It requires struct events and embeds `EventPriority`.
- Ownership candidate: Framework event dispatch abstraction, but the contract currently couples to UniTask and a priority policy.
- Required design action: decide whether async dispatch belongs in the core event contract, an optional async extension, or an adapter; decide ownership and semantics of priority and clearing subscriptions.
- No signature change until consumer call sites and dispatch semantics are inventoried.

### Serialization
File: `Assets/_MythHunter/Code/Data/Serialization/ISerializable.cs`
- Namespace: `MythHunter.Data.Serialization`
- Exposes `byte[] Serialize()` and `Deserialize(byte[] data)`.
- This forces implementation objects to own byte-format details and mutating deserialization.
- Ownership candidate: not accepted as the generic Framework serialization contract without redesign.
- Consider a codec/serializer abstraction that operates on data objects and explicit schema/version information rather than requiring every component/domain type to implement raw byte operations.
- Persistence and network protocol formats remain separate concerns.

## Current contract gaps / risks
1. There are already interfaces for most major areas; the task is not to invent a second parallel API surface.
2. All examined interfaces use MythHunter namespaces and therefore are not yet a platform-neutral public Framework API boundary.
3. ECS uses `int` IDs and array-returning query methods; stable identity and query semantics are not frozen.
4. `IEcsWorld` and `ISystem` expose overlapping lifecycle-shaped methods; exact ownership of update scheduling is unresolved.
5. `IDIContainer` exposes concrete support types and a broad API that may be difficult to implement independently.
6. `IEventBus` exposes UniTask, event priority and synchronous/asynchronous behavior in one contract.
7. `IEvent` requires ID and priority even though those could be dispatch policy rather than the occurrence's essential data.
8. `ISerializable` couples serializable objects to byte-level persistence details and lacks explicit schema/version policy.
9. No project-owned assembly definitions were found under `Assets/_MythHunter/Code/`; a real Framework compile-time boundary does not yet exist there.
10. The audit does not prove compilation or test pass status. A local Unity compile/test baseline remains required before modifying source.

## Target contract slice (provisional)
These are responsibilities to establish; names are provisional and should be aligned to the assembly decision before implementation:

- **ECS contracts:** component marker/value expectations, entity identity contract, entity-manager/world/query contracts.
- **System contracts:** system lifecycle and update/scheduler contracts with no MythHunter phase semantics.
- **DI contracts:** registration/resolution contract and carefully owned lifetime/scope abstractions; no concrete container implementation types leaking through interfaces.
- **Event contracts:** event occurrence abstraction and dispatch/subscription contract; async and priority policies explicitly decided instead of silently inherited.
- **Serialization contracts:** serializer/codec interface, stable schema identification/version metadata; avoid raw byte methods on every domain object unless the use case requires them.
- **Logging abstraction:** only after locating the actual logger interfaces and all dependents; do not invent a competing duplicate logger.
- **Validation primitives:** pure runtime validation only; Editor scanning/code generation is not a Framework Core contract.

## Recommended extraction sequence
1. Preserve the existing source and establish the local Unity compile/test baseline before any code modification.
2. Inspect all usages/implementations of each selected existing interface and identify its concrete dependent types.
3. Decide the minimum stable contract for one concept at a time, starting with the smallest truly neutral candidate.
4. Add the Framework assembly boundary only after checking which existing files can compile inside it without references to MythHunter Game, Unity APIs or unrelated packages.
5. Move or adapt existing contract definitions; do not copy them to a new namespace and leave duplicate competing interfaces.
6. Update consumers in small cohesive groups, maintaining a compilable checkpoint after each group.
7. Add tests for contract behavior and architecture-boundary enforcement.
8. Mark the extraction item complete only after actual source code changes, project compile/test validation, and dependency checks.

## Decisions deliberately not made
- Final public namespaces and exact contract assembly split.
- Entity ID representation or lifecycle invariants.
- Whether queries return arrays, enumerators, spans or dedicated query types.
- Whether event priority/async behavior are core or optional.
- Exact DI scope API.
- Concrete serialization wire/save formats.
- Any new assembly definitions.

These require call-site/dependency evidence and local Unity validation; choosing them from the interface declarations alone would be premature.

## Conclusion
The source already contains candidate contracts, but they are not yet clean independent Framework contracts. The correct next action is a usage/dependency audit and a local compile/test baseline, followed by one controlled extraction—not duplicating interfaces or rewriting signatures blindly.

This is an architecture inventory/plan only. No MythHunter source code was changed.
