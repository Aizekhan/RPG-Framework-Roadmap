# EPIC 03.4 — Assembly Dependency Inventory

## Status
COMPLETE — first pure Framework runtime and test boundary selected and statically validated.

## Existing compilation shape
- No `.asmdef` or `.asmref` exists under `Assets/_MythHunter` on `dev`.
- MythHunter code currently uses Unity's predefined assemblies.
- Predefined assemblies can reference an auto-referenced project asmdef; the Framework asmdef must not reference predefined MythHunter types.
- Six existing UniTask asmdefs remain unchanged.

## Selected independent slice
- Runtime: `Assets/_Framework/ECS/Runtime/RPGFramework.ECS.Runtime.asmdef`
- Namespace: `RPGFramework.ECS`
- Public API: `IComponent`, `IEntityManager`
- Implementation: dictionary-backed `EntityManager`
- Unity tests: `Assets/_Framework/ECS/Tests/EntityManagerTests.cs`
- Test asmdef: `Assets/_Framework/ECS/Tests/RPGFramework.ECS.Runtime.Tests.asmdef`
- Headless test harness: `Build/RPGFramework.ECS.Tests/RPGFramework.ECS.Tests.csproj`
- Static boundary validator: `Build/validate_framework_ecs_boundary.py`
- CI workflow: `.github/workflows/rpg-framework-ecs.yml`

## Transitional policy
The new pure Framework runtime is canonical for future Framework consumers. Current MythHunter consumers still use `MythHunter.Core.ECS`; those old contracts/runtime remain temporarily to avoid switching the existing game without validation. The duplicate implementations must not remain as the final architecture. A later consumer migration needs to switch all call sites coherently and remove the old runtime, with compile/test evidence.

## Important semantic decision
The first implementation rejects `AddComponent` for unknown/destroyed entity IDs with `ArgumentException`. This prevents phantom entities, but differs from the old implementation's behavior that could create storage for an arbitrary ID. This is an intentional correctness change in the new Framework implementation and is covered by a regression test; do not silently assume it preserves all legacy semantics.

Other initial semantics:
- IDs start at 1 and increase during manager lifetime.
- Destroying an unknown ID is a no-op.
- `GetComponent` returns `default` when missing.
- The API keeps integer IDs; a value-type `Entity` handle is out of scope here.

## Scope review
The draft also changes these legacy files for Editor/runtime portability:
- `Assets/_MythHunter/Code/Debug/Core/PreloadDebugTool.cs`
- `Assets/_MythHunter/Code/Events/EventBus.cs`
- `Assets/_MythHunter/Code/Resources/Pool/PooledObjectLifetimeTracker.cs`
- `Assets/_MythHunter/Code/Services/Prefabs/IPrefabProvider.cs`
- `Assets/_MythHunter/Code/Services/Prefabs/PrefabProvider.cs`

These are not required to compile the pure Framework runtime through the .NET harness. Review separately and split from the core extraction if they are not needed for this slice. Preserve unrelated local files and all existing Unity GUIDs.

## Validation evidence
GitHub Actions workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38049497156
Tested commit: `be362f9df6910b3027f05f5b742b65bc3dc65182`
- Static boundary validation: pass.
- .NET compile/NUnit: 8 passed, 0 failed, 0 skipped.
- No warnings reported in the latest job log according to the PR's test record.
- This harness links the runtime C# files directly and does not prove Unity asmdef import or full MythHunter compilation.

## Remaining EPIC 03.5 gate
- Import the feature branch into Unity and verify asmdef recognition.
- Compile the full project and record any new errors against the baseline.
- Run the 8 tests through Unity Test Runner.
- Review/split Editor/runtime portability edits before merge.
- Keep PR #18 a draft until these criteria are met.
