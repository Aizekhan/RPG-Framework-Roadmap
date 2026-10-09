# EPIC 03.1C — Contract Usage Map and First Extraction Boundary

## Status
REVIEW REQUIRED — EPIC 03.1 remains ACTIVE

## Purpose
Record concrete source references to existing contract interfaces and determine which areas are consumers, implementations, or contract-coupled integrations. This supports the first extraction without changing production source code.

## Source basis
- `Aizekhan/MythHunter`, branch `dev`, tree SHA `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`.
- GitHub code search on that tree was used to locate references.
- Results are a practical call-site sample, not a formal compiler-generated dependency graph and not proof every usage has been enumerated.

## Usage map

### IComponent / IEntityManager
Relevant files found:
- Contract/implementation: `Core/ECS/IComponent.cs`, `Core/ECS/IEntityManager.cs`, `Core/ECS/EntityManager.cs`.
- ECS helpers: `Core/ECS/IComponentFactory.cs`, `ComponentFactory.cs`, `ComponentCache.cs`, `ComponentCacheRegistry.cs`.
- Archetype/template areas: `Entities/Archetypes/EntityArchetypeBase.cs`, `ArchetypeTemplateBuilder.cs`, `IArchetypeTemplateBuilder.cs`, `IArchetypeTemplateRegistry.cs`.
- Gameplay consumers include `Systems/Gameplay/Combat/CombatDetectionSystem.cs`, `Systems/Lobby/HeroSelectionSystem.cs`, `Systems/Gameplay/Movement/MovementSystem.cs`, `Systems/Gameplay/Combat/CombatAbilitySystem.cs`, and `Systems/Gameplay/Combat/CombatSystem.cs`.
- Serialization registry also uses `IComponent`: `Entities/IComponentSerializerRegistry.cs` and `Data/Serialization/IComponentSerializer.cs`.

Implication:
- `IComponent` has broad reach across core ECS, generic entity template helpers, concrete game components and serializers.
- A namespace/type move must be coordinated as a single source-wide dependency change; copying a second `IComponent` would split the type identity and break generic constraints.
- `IEntityManager` exposes behaviour currently relied on by several gameplay systems; its semantics are compatibility-sensitive.

First-boundary recommendation:
- Prioritize making the existing `IComponent` type available from a neutral Framework assembly without creating a duplicate type.
- Before doing so, enumerate all dependencies of its containing directory and all `using MythHunter.Core.ECS`/fully qualified references; a new assembly should contain only the minimum files that compile independently.
- Do not attempt to move all of `Core/ECS` as a folder.

### IDIContainer
Search results show usage in:
- Implementation: `Core/DI/DIContainer.cs`.
- Installer infrastructure: `DIInstaller.cs`, `InstallerRegistry.cs`, `CoreServices/*Installer.cs`, `SystemsGroups/*Installer.cs`.
- System registry: `Systems/Core/ISystemRegistry.cs`, `Systems/Core/SystemRegistry.cs`, `ParallelSystemRegistry.cs`.
- Application/state flow: `Core/Game/GameBootstrapper.cs`, `Core/StateMachine/GameStateMachine.cs`, `BaseState.cs`, concrete states.
- UI/runtime integrations: `UI/Core/UISystem.cs`, `UIViewFactory.cs`, `Core/DI/DIContainerExtensions.cs`, Unity MonoBehaviours.
- Resource and debug systems: `Resources/PreloadManager.cs`, `Debug/DebugService.cs`.

Implication:
- This interface is a high-fan-out contract, and its consumers also resolve concrete game services and Unity-bound objects through the same container.
- The first Framework contract split cannot rename/change `IDIContainer` in isolation without a broad migration and explicit adapter/composition plan.

First-boundary recommendation:
- Defer the public DI API move until the minimal contract, scope/lazy ownership and explicit registration pattern are settled.
- Keep the existing type canonical during initial extraction; no parallel container interfaces.

### IEventBus
Search results show usage in:
- Implementation: `Events/EventBus.cs`; a separate `SimpleEventBus.cs` also exists.
- System base: `Core/ECS/SystemBase.cs`.
- Application/game flow: `Core/Game/GameFlowManager.cs`, state classes, `Systems/Phase/PhaseSystem.cs`.
- Game systems include timer, preload, lobby, entity spawn, archetype, movement, combat and combat detection/ability.
- UI/presentation includes presenters and navigation services.
- Event infrastructure includes batching, throttling, handler base and related components.

Implication:
- The event bus is highly coupled across application, gameplay, infrastructure and UI.
- The existing contract combines sync/async dispatch, event priority and a dependency on UniTask; extracting it unchanged preserves the coupling rather than creating a neutral seam.
- Having both EventBus and SimpleEventBus reinforces the need for semantic parity analysis before replacement.

First-boundary recommendation:
- Do not move `IEventBus` unchanged in the first contract slice.
- Audit behaviour and call-site signatures before selecting a minimum event contract or async extension.
- In the first slice, event work is documentation/testing/design only unless a fully compatible extraction can be proven.

### ISerializable
Search results show usage in:
- `Data/Serialization/VersionedSerializer.cs`.
- `Networking/Messages/NetworkEventMessage.cs`.
- `Networking/Serialization/DeltaSerializer.cs`.
- `Events/NetworkEvents/INetworkEvent.cs` (combines event and serializable concerns).
- `Networking/Messages/INetworkMessage.cs`.
- Several components implement `ISerializableComponent`; some concrete components also implement `IComponent`.

Implication:
- The current byte-oriented `ISerializable` crosses persistence, ECS component and network-message domains.
- Moving it to Framework Serialization as-is would perpetuate the coupling between save and network contracts.

First-boundary recommendation:
- Do not extract `ISerializable` as the universal serializer contract yet.
- Keep serialization contract redesign for its dedicated task, after save/network schemas are separated.

## First extraction boundary decision

### Proposed smallest initial slice
`IComponent` is the only current candidate likely to be a low-surface first move because it is a marker interface without members or direct external dependencies. However, it is referenced by components, storage, factories, templates, and serialization contracts across multiple folders, so the change is not zero-risk.

The first source patch should be limited to:
1. one canonical Framework-owned `IComponent` definition;
2. the minimum assembly/folder setup needed for that definition to compile;
3. coordinated namespace/reference changes only if required;
4. no EntityManager storage rewrite, no ComponentCache removal, no query API change, no gameplay behavior change.

If the local compile cannot validate this patch, do not mark it complete; first establish or restore the compile/test baseline.

### Explicitly deferred
- `IEntityManager` API redesign.
- `IEcsWorld` extraction while it owns system registry lifecycle.
- `ISystem` public placement before scheduler dependencies are resolved.
- `IDIContainer` extraction until scope/lazy/lifecycle semantics are settled.
- `IEventBus` extraction before sync/async/priority contract and behavior tests.
- `ISerializable` extraction before persistence/network ownership is disentangled.

## Additional source-search validation

A second code search for `IComponent` and `using MythHunter.Core.ECS` confirmed that the marker is referenced from ECS helpers, concrete game components, archetype/template builders and serialization registries. Examples include:
- `Core/ECS/IComponentFactory.cs`
- `Core/ECS/IComponentCacheRegistry.cs`
- `Entities/IComponentSerializerRegistry.cs`
- `Data/Serialization/IComponentSerializer.cs`
- `Entities/Archetypes/EntityArchetypeBase.cs`
- `Entities/Archetypes/ArchetypeTemplateBuilder.cs`
- `Components/Combat/HealthComponent.cs`
- `Components/Combat/TeamComponent.cs`
- `Components/Movement/PathComponent.cs`
- `Components/Character/StatsComponent.cs`
- `Editor/Wizards/MythHunterCodeGenerator.cs`

This confirms the type is a cross-cutting generic constraint, not just an implementation detail local to the ECS folder. Moving it requires a canonical type identity and coordinated source-wide references, including authoring/code-generation templates. The source search does not provide a complete formal graph, so local checkout analysis remains necessary.

## Verification and limitations
- Source declaration and example consumers were verified by file retrieval and code-search results.
- No Unity compile or automated test was executed through the GitHub connector.
- No production source changes were made for this usage map.
- A local compile/test baseline and complete dependency check are mandatory before the first source patch.

## Conclusion
The usage map supports a cautious first extraction of the marker contract `IComponent`, subject to local baseline validation and a complete reference check. Higher-fan-out contracts (`IDIContainer`, `IEventBus`, `ISerializable`) should not be moved unchanged because their current APIs expose implementation/platform or cross-domain policy coupling. EPIC 03.1 remains ACTIVE until the first contract is actually extracted and validated.
