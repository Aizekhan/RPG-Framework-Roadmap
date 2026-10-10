# EPIC 03.4 — Assembly Dependency Inventory

## Status
PARTIAL INVENTORY — sufficient to reject a premature contracts-only assembly; full file-by-file map remains open.

## Current compilation shape
No `.asmdef` or `.asmref` exists under `Assets/_MythHunter`. Most project source therefore participates in Unity's predefined assembly layout, apart from package/plugin asmdefs. Creating an asmdef within the current ECS folder establishes a separate assembly and stops that folder from belonging to the predefined assembly; the rest of the project cannot simply reference it without being moved under compatible named assemblies.

## Confirmed boundary constraints
- `IComponent` is referenced by a wide set of components, storage/cache, factories, archetype/template code, and serializers.
- `IEntityManager` is referenced by ECS runtime, EntityFactory/HeroFactory, archetypes/templates, gameplay systems, installers, serializers' adjacent systems and caches.
- `EcsWorld` directly depends on `MythHunter.Systems.Core.ISystemRegistry`; it is not a neutral contract.
- `Assets/_MythHunter/Code` is not uniformly engine-independent: UI, debug, settings and utility files reference UnityEngine or Unity packages.
- Direct/editor-sensitive references are present in or beneath runtime-looking directories:
  - `Assets/_MythHunter/Code/Debug/Core/PreloadDebugTool.cs`: direct `using UnityEditor`
  - `Assets/_MythHunter/Code/Events/EventBus.cs`: direct `using UnityEditor`
  - `Assets/_MythHunter/Code/Services/Prefabs/PrefabProvider.cs`: conditional AssetDatabase use
  - `Assets/_MythHunter/Code/Resources/Pool/PooledObjectLifetimeTracker.cs`: guarded UnityEditor use
  - `Assets/_MythHunter/Code/Services/Prefabs/IPrefabProvider.cs`: UnityEngine and conditional UnityEditor
- Editor directory files explicitly reference UnityEditor and game types; once named assemblies are introduced, their reference direction must be Game/Runtime -> Framework and Editor -> Game/Runtime. Runtime must not depend on Editor.
- `Assets/Plugins/UniTask` has six assembly definitions and its own Editor/Runtime split. Do not move or mutate UniTask for this milestone.

## Immediate conclusion
A giant `MythHunter.Runtime.asmdef` covering the entire existing code tree is not yet justified. A two-interface contracts asmdef is also incomplete until consumers can reference it. The next implementation must use a dependency map and make one coherent assembly cut.

## Next bounded action
Review dependency clusters around ECS and UnityEditor references, then choose one of:
- **Option A:** move all existing MythHunter code into a runtime assembly and explicitly create an editor assembly. This may be too broad given package/API dependencies.
- **Option B:** first make ECS-related source a coherent named assembly slice including its direct internal dependencies, while moving dependent callers together. This may be smaller but must prove dependency closure.
- **Option C:** keep `IComponent` inside current predefined assembly temporarily and extract only truly independent code into a standalone package with no current-type dependencies. This is safest for a framework-first prototype but may require adapting existing runtime code later.

Do not choose solely for minimal diff count. Choose the option that produces a useful reusable module and a verifiable dependency graph.

## Validation rule
This document is source inspection, not compile evidence. The current CI workflow does not compile. PR #18 remains draft; no merge is allowed until the assembly layout is complete, focused tests exist, and Unity compilation succeeds.
