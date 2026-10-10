# Project Status

## Source Project
This roadmap controls refactoring of **Aizekhan/MythHunter**, Unity project on branch `dev`.

- Roadmap: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`
- Source baseline: `Audits/EPIC_03.1_Source_Baseline.md`
- ECS contract usage map: `Audits/EPIC_03.2_ECS_Contract_Usage_Map.md`
- Assembly boundary reassessment: `Audits/EPIC_03.3_Assembly_Boundary_Reassessment.md`
- Assembly dependency inventory: `Audits/EPIC_03.4_Assembly_Dependency_Inventory.md`

## Current Position
- Epic: 03 — First Reusable Framework Slice
- **Active task: 3.5 — Implement and validate first Framework ECS runtime**
- Status: ACTIVE — static boundary and headless .NET tests passed for commit `be362f9df6910b3027f05f5b742b65bc3dc65182`; Unity integration is still unverified.
- PR: [#18 — Draft: isolated RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Current PR head reported: `be362f9df6910b3027f05f5b742b65bc3dc65182`.
- Required next: Unity import/compile/Test Runner on the current PR head and review/split unrelated portability edits.
- Do not merge PR #18 until that validation is recorded.

## Implemented on the feature branch
- Standalone `RPGFramework.ECS.Runtime` assembly, no references, `noEngineReferences: true`.
- Pure C# `IComponent`, `IEntityManager` and dictionary-backed `EntityManager`.
- Separate Unity EditMode test assembly with eight NUnit tests.
- Headless .NET 8 test harness and GitHub Actions workflow.
- Static boundary validator for asmdef settings, forbidden runtime references, Unity metadata and GUID uniqueness.
- `AddComponent` rejects unknown or destroyed IDs with `ArgumentException` to prevent phantom entities; regression test included.
- Legacy `MythHunter.Core.ECS` runtime remains intact temporarily. Consumer migration and removal of duplicate legacy runtime remain future tasks.
- Additional Editor/runtime portability changes are included and need scope review.

## Validation evidence
- Workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38049497156
- Tested commit: `be362f9df6910b3027f05f5b742b65bc3dc65182`.
- Workflow result: successful; static boundary validation passed; .NET compile/NUnit reported 8 passed, 0 failed, 0 skipped.
- The latest PR head differs from the tested commit; do not claim this result validates later commits.
- Headless .NET harness does not prove Unity asmdef import, Unity Test Runner discovery, or full MythHunter project compilation.

## Scope concern
The PR includes portability edits in these legacy files which are outside the isolated Framework runtime:
- `Assets/_MythHunter/Code/Debug/Core/PreloadDebugTool.cs`
- `Assets/_MythHunter/Code/Events/EventBus.cs`
- `Assets/_MythHunter/Code/Resources/Pool/PooledObjectLifetimeTracker.cs`
- `Assets/_MythHunter/Code/Services/Prefabs/IPrefabProvider.cs`
- `Assets/_MythHunter/Code/Services/Prefabs/PrefabProvider.cs`

Review whether these are necessary for the Framework slice. Prefer splitting them into a separate PR if not required.

## Rules
1. Do not reset, clean or discard user's local worktree changes.
2. Do not modify `dev` directly.
3. Keep the Framework runtime independent of MythHunter, UnityEngine, UnityEditor, logging, DI and providers.
4. Do not leave duplicate legacy and Framework ECS implementations as the final architecture.
5. Do not return to broad diagnostics; validate only concrete source changes.

## Next action
Validate the current PR head in Unity once, then resolve only concrete failures. Review/split unrelated portability edits. After this first runtime slice is validated and merged, the next planned task is moving existing MythHunter ECS consumers to `RPGFramework.ECS` and removing the legacy ECS runtime coherently.

## Source of truth
- Master Plan: ordering and checklist.
- STATUS.md: exact active task.
- Audits: evidence and architectural rationale.
