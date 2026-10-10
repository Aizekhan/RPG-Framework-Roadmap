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
- Status: ACTIVE — pure .NET and static boundary checks passed; Unity validation pending.
- PR: [#18 — Draft: add isolated RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Next required validation: import the branch into Unity, confirm compilation, and execute the Unity test assembly. Do not merge before that check.

## What is implemented on the feature branch
- Pure .NET `RPGFramework.ECS.Runtime` assembly with no references and `noEngineReferences: true`.
- `IComponent`, `IEntityManager`, and an initial dictionary-backed `EntityManager`.
- Eight NUnit tests in a separate Unity test assembly.
- .NET 8 test harness, static boundary validator, and GitHub Actions workflow.
- `AddComponent` rejects unknown/destroyed IDs with `ArgumentException`; regression coverage confirms no phantom entity is created.
- Legacy `MythHunter.Core.ECS` interface files remain intact; MythHunter consumers have not yet been migrated to the new runtime.
- Legacy interface GUIDs are preserved; seven new Framework/test asset GUIDs were checked and no duplicates were found.
- A small set of Editor/runtime portability changes is included in PR #18 and must be reviewed as part of its diff.

## Latest CI evidence
- Workflow: [RPGFramework ECS Runtime](https://github.com/Aizekhan/MythHunter/actions/runs/38049497156)
- Tested commit: `be362f9df6910b3027f05f5b742b65bc3dc65182`.
- Static assembly boundary check: **OK**.
- Static scan: 3 runtime C# files, 1 test C# file, 7 Framework/test asset GUIDs checked.
- .NET compile and NUnit result: **8 passed, 0 failed, 0 skipped**.
- No C# compiler warnings were found in the latest job log.
- This validates the pure .NET sources linked into the harness and the checked metadata, not Unity asmdef import or full MythHunter compilation.

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
