# Project Status

## Source Project
This roadmap controls refactoring of **Aizekhan/MythHunter**, Unity project on branch `dev`.

- Roadmap: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`
- Source baseline: `Audits/EPIC_03.1_Source_Baseline.md`
- ECS contract usage map: `Audits/EPIC_03.2_ECS_Contract_Usage_Map.md`
- Boundary reassessment: `Audits/EPIC_03.3_Assembly_Boundary_Reassessment.md`
- Dependency inventory: `Audits/EPIC_03.4_Assembly_Dependency_Inventory.md`

## Current Position
- Epic: 03 — First Reusable Framework Slice
- **Active task: 3.5 — Implement and validate first Framework ECS runtime**
- Status: ACTIVE — pure .NET validated; Unity validation pending.
- PR: [#18 — Draft: add isolated RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Next required validation: import the branch into Unity, confirm compilation, and execute the Unity test assembly. Do not merge before that check.

## What is implemented on the feature branch
- Pure .NET `RPGFramework.ECS.Runtime` assembly with no references and `noEngineReferences: true`.
- `IComponent`, `IEntityManager`, and an initial dictionary-backed `EntityManager`.
- Eight NUnit tests in a separate Unity test assembly.
- .NET 8 test harness and GitHub Actions workflow so the pure runtime compiles/tests without loading Unity.
- `AddComponent` now rejects unknown/destroyed IDs with `ArgumentException`; it cannot silently create a phantom entity.
- Legacy `MythHunter.Core.ECS` interface files remain intact; MythHunter consumers have not yet been migrated to the new runtime.
- New Unity script GUIDs were checked for uniqueness across changed asset meta files; legacy interface GUIDs are preserved.
- A small set of Editor/runtime portability changes is included in PR #18 and must be reviewed as part of its diff.

## CI evidence
- Latest GitHub Actions run: [RPGFramework ECS Runtime](https://github.com/Aizekhan/MythHunter/actions/runs/38049093863)
- Tested commit: `cd3d34633ba8d656347cfbaf8f85848b3810a790`.
- Result: **8 passed, 0 failed, 0 skipped**.
- The previous nullable warnings were addressed; no C# compiler warnings were found in the latest job log.
- This validates only the pure .NET sources linked into the test project. It does not validate Unity .asmdef import or the whole MythHunter project.

## Architectural constraints
1. Keep `RPGFramework.ECS.Runtime` independent of MythHunter, UnityEngine, UnityEditor, logging, DI, and providers.
2. Do not add a broad MythHunter runtime asmdef merely to consume this module; predefined Unity assemblies can reference auto-referenced user assemblies. Keep the dependency direction one-way.
3. The legacy MythHunter ECS and the new Framework ECS temporarily coexist. This is an intentional migration phase, not the final architecture.
4. The follow-up migration must move consumers to Framework APIs and remove the duplicate legacy runtime; do not leave two ECS runtimes as the finished design.
5. Do not reset or discard the user's local working-tree changes.
6. Keep PR #18 as draft until the Unity import/compile/test gate passes.

## Next action
Validate PR #18 once in local Unity. If it compiles and the tests run, record the result and plan the next small migration slice for moving existing MythHunter consumers onto the Framework runtime. If it fails, fix only the concrete integration failure without starting another broad diagnostic loop.

## Source of truth
- Master plan: task ordering and checkboxes.
- STATUS.md: exact active task.
- Audits: source-backed rationale and validation record.
