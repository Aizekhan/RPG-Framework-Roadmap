# EPIC 03.4 — Assembly Dependency Inventory

## Status
COMPLETE — static dependency map and first assembly cut are recorded. Unity-specific validation is tracked in EPIC 03.5.

## Existing compilation shape
- No .asmdef or .asmref currently exists under Assets/_MythHunter on dev.
- Most MythHunter source is compiled in Unity's predefined assembly layout.
- Unity predefined assemblies can reference user asmdef assemblies when Auto Referenced is enabled; the Framework module does not need a large MythHunter.Runtime.asmdef merely to be available.
- Framework asmdef assemblies must not reference types in predefined MythHunter assemblies.

## Confirmed consumer clusters
- `IComponent`: concrete component structs, ECS caches/storage/factories, archetype/template code, serialization contracts/implementations, generated code.
- `IEntityManager`: storage, component factories/cache, archetype/template code, gameplay systems, entity/hero factories, installers and ECS world.
- `EcsWorld` depends on `MythHunter.Systems.Core.ISystemRegistry`; it remains game-owned.
- The six UniTask asmdefs remain unchanged in this slice.
- UnityEngine-dependent components, UI, debug, settings and services remain outside Framework runtime.

## Selected first cut
- Runtime: `Assets/_Framework/ECS/Runtime/RPGFramework.ECS.Runtime.asmdef`
- Public surface: `RPGFramework.ECS.IComponent`, `RPGFramework.ECS.IEntityManager`
- Initial implementation: `RPGFramework.ECS.EntityManager`
- Unity test assembly: `Assets/_Framework/ECS/Tests/RPGFramework.ECS.Runtime.Tests.asmdef`
- Headless test harness: `Build/RPGFramework.ECS.Tests/RPGFramework.ECS.Tests.csproj`
- CI: `.github/workflows/rpg-framework-ecs.yml`

The Framework runtime is pure .NET, has no asmdef references, and sets `noEngineReferences: true`. It is intentionally independent of MythHunter and Unity. The current MythHunter contracts/runtime are preserved during this step; consumer migration and legacy removal are explicitly later work, so this interim duplication must not become the final state.

## Source cleanup in the draft
- Preserved legacy MythHunter interface asset GUIDs.
- Assigned unique GUIDs to new Framework script assets; static scan found no duplicate GUIDs among changed .meta files.
- Kept a few Editor/runtime portability changes in PR #18: guarded Editor-only references in preload/prefab tooling, removed the direct Editor import from EventBus, and corrected a preprocessor directive. Review their scope separately from the ECS runtime.
- Did not add a broad runtime asmdef to the existing MythHunter source.

## Test evidence
The GitHub Actions workflow compiled the pure .NET source and ran the NUnit suite. Two nullable warnings were corrected with an explicit default-return annotation and a setup-initialized test fixture field. A subsequent behavior review found that adding a component to an unknown or destroyed ID could create a phantom entity; the new Framework implementation now throws `ArgumentException`, and a regression test covers that case.

Latest run:
- Workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38049497156
- Commit: `be362f9df6910b3027f05f5b742b65bc3dc65182`
- Static boundary validator: OK; it checked the runtime asmdef, forbidden source references, metadata presence, and uniqueness of 7 Framework/test GUIDs.
- Result: 8 passed, 0 failed, 0 skipped; no C# compiler warnings detected in the latest job log.

This validates the pure .NET files linked by the harness only. It does not prove Unity asmdef import, full MythHunter compilation or Unity Test Runner results.

## Remaining EPIC 03.5 gate
- Import the feature branch in local Unity.
- Confirm the Framework runtime and test asmdefs are recognized correctly.
- Confirm the full project compiles without new errors.
- Run the eight focused tests in Unity's Test Runner.
- Preserve local user changes; do not reset the local worktree.
- Keep PR #18 draft until the Unity validation result is recorded.
