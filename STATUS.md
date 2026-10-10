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
- Next ECS consumer migration task: `Tasks/EPIC_03.6_ECS_Consumer_Migration.md`

## Current Position
- Epic: 03 — First Reusable Framework Slice
- **Active task: 3.5 — Implement and validate first Framework ECS runtime**
- Status: ACTIVE — current-head headless CI passes; Unity integration remains the required validation gate before merge.
- PR: [#18 — Draft: standalone RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Current PR head: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- Current-head workflow: [RPGFramework ECS Runtime](https://github.com/Aizekhan/MythHunter/actions/runs/38067838447).
- Required next: import/compile current PR head in Unity 6000.0.45f1 and run the Framework EditMode test assembly.
- Do not merge PR #18 until that validation is recorded.

## Implemented on the feature branch
- Standalone `RPGFramework.ECS.Runtime` assembly with no assembly references and `noEngineReferences: true`.
- Pure C# `IComponent`, `IEntityManager` and dictionary-backed `EntityManager`.
- Separate Unity EditMode test assembly with eight NUnit tests.
- Headless .NET 8 test harness and GitHub Actions workflow.
- Static boundary validator for asmdef settings, forbidden runtime references, Unity metadata and GUID uniqueness.
- `AddComponent` rejects unknown or destroyed IDs with `ArgumentException`; regression coverage included.
- Legacy MythHunter ECS runtime remains active until a separate consumer migration is validated.
- Five unrelated Editor/runtime portability edits were removed from PR #18.

## Validation evidence
- Current-head workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38067838447
- Tested PR head: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- Workflow: success; static boundary validation passed; .NET compile/NUnit: 8 passed, 0 failed, 0 skipped.
- This does not prove Unity asmdef import, Unity Test Runner discovery or full MythHunter project compilation.
- Unity integration remains unverified.

## Rules
1. Do not reset, clean or discard user's local worktree changes.
2. Do not modify `dev` directly.
3. Keep Framework runtime independent of MythHunter, UnityEngine, UnityEditor, logging, DI and providers.
4. Do not leave duplicate legacy and Framework ECS implementations as the final architecture.
5. Do not return to broad diagnostics; validate only concrete source changes.
6. Track one active roadmap task at a time.

## Next action
Run Unity import/compile/EditMode validation against PR #18's current head. Resolve only concrete failures. If it passes, complete final review and merge. Then activate task 3.6 in `Tasks/EPIC_03.6_ECS_Consumer_Migration.md`: migrate the existing MythHunter ECS consumers to Framework contracts/runtime and remove the legacy ECS implementation coherently.

## Source of truth
- Master Plan: ordering and checklist.
- STATUS.md: exact active task.
- Tasks: implementation scope and acceptance criteria.
- Audits: evidence and architectural rationale.
