# EPIC 03.4 — Assembly Dependency Inventory

## Status
PARTIAL INVENTORY — enough to keep the initial Framework contract slice narrow; detailed mapping continues only if a specific dependency requires it.

## Current compilation shape
No `.asmdef` or `.asmref` exists under `Assets/_MythHunter`. Most MythHunter source participates in Unity's predefined assembly layout, apart from package/plugin asmdefs.

Unity's documented default behavior is that predefined assemblies reference all project asmdef assemblies with `Auto Referenced` enabled. The proposed `Framework.ECS.Contracts.asmdef` sets `autoReferenced: true`. Therefore existing predefined-assembly consumers can reference the contract assembly by default. The opposite dependency must be forbidden: the contracts assembly cannot reference types from predefined MythHunter assemblies.

## Confirmed boundary constraints
- `IComponent` is referenced by components, storage/cache, factories, archetype/template code and serializers.
- `IEntityManager` is referenced by ECS runtime, EntityFactory/HeroFactory, archetypes/templates, gameplay systems, installers and caches.
- `EcsWorld` directly depends on `MythHunter.Systems.Core.ISystemRegistry`; it is not a neutral contract and must stay outside the contracts assembly.
- UnityEngine-dependent UI, debug, settings and utility files exist. They do not need to move to make the auto-referenced contracts assembly available.
- Editor-sensitive files under runtime-looking paths deserve targeted cleanup, but should not expand this first contract extraction into a broad assembly migration.
- Keep the six existing UniTask asmdefs untouched for this task.

## Immediate conclusion
Do not create one large `MythHunter.Runtime.asmdef` for this work. The initial two-interface contracts assembly is a reasonable small boundary if it remains dependency-free and passes the consumer compile/test validation.

## Next bounded action
1. Preserve asset GUIDs while moving `IComponent` and `IEntityManager`.
2. Confirm existing files compile against the canonical types without duplicate declarations.
3. Add focused tests for the existing `EntityManager` behavior or at the appropriate test seam.
4. Validate the Unity assembly import/compile on the feature branch.

## Validation rule
The current CI workflow does not run Unity compilation. PR #18 remains draft until the integration is locally compiled and relevant tests pass.
