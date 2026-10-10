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
- Status: ACTIVE — headless CI passes; Unity integration gate now PASSES for assembly discovery and focused tests. Full EditMode run on the exact PR head discovered `RPGFramework.ECS.Runtime.Tests.dll`: 8 ECS tests passed; total project tests 9 passed, 0 failed, 0 skipped. Full project compilation evidence is limited to this successful test invocation.
- PR: [#18 — Draft: standalone RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Current PR head: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- Current-head workflow: [RPGFramework ECS Runtime](https://github.com/Aizekhan/MythHunter/actions/runs/38067838447).
- Unity ECS test gate is passed; do not merge PR #18 until final changed-file and PR CI/review checks are complete.

## Implemented on the feature branch
- Standalone `RPGFramework.ECS.Runtime` assembly with no assembly references and `noEngineReferences: true`.
- Pure C# `IComponent`, `IEntityManager` and dictionary-backed `EntityManager`.
- Separate Unity EditMode test assembly with eight NUnit tests.
- .NET 8 test harness and GitHub Actions workflow.
- Static boundary validator for asmdef settings, forbidden runtime references, Unity metadata and GUID uniqueness.
- `AddComponent` rejects unknown or destroyed IDs with `ArgumentException`; regression coverage included.
- Legacy MythHunter ECS runtime remains active until a separate consumer migration is validated.
- Five unrelated Editor/runtime portability edits were removed from PR #18.

## Validation Evidence
- Current-head workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38067838447
- Tested PR head: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- GitHub Actions: success; static boundary validation passed; .NET compile/NUnit: 8 passed, 0 failed, 0 skipped.
- Earlier Unity CLI runs either discovered only an unrelated Addressables stub or returned zero tests with an overly specific filter.
- Latest Unity CLI full EditMode run: `FrameworkECS-TestResults.xml`; root suite Passed, total 9, passed 9, failed 0, skipped 0. `RPGFramework.ECS.Runtime.Tests.dll` was discovered and all 8 `RPGFramework.ECS.Tests.EntityManagerTests` passed.
- Unity CLI reported Unity `6000.0.45f1`; local HEAD was verified as `3917e6a48344d4469fd95a2049b1a2e60d983223` before the run.

## Rules
1. Do not reset, clean or discard user's local worktree changes.
2. Do not modify `dev` directly.
3. Keep Framework runtime independent of MythHunter, UnityEngine, UnityEditor, logging, DI and providers.
4. Do not leave duplicate legacy and Framework ECS implementations as the final architecture.
5. Do not return to broad diagnostics; validate only concrete source changes.
6. Track one active roadmap task at a time.

## Next Action
Unity CLI has now discovered and executed the test assembly successfully on the exact PR head: 8 ECS tests passed and the full EditMode run passed 9/9 tests. Next: verify the final changed-file list and PR CI/review state, then update the Unity gate evidence and prepare PR #18 for final review. Do not activate task 3.6 until PR #18 is reviewed/merged; then migrate MythHunter consumers and remove legacy ECS duplication coherently.

## Source of Truth
- Master Plan: ordering and checklist.
- STATUS.md: exact active task.
- Tasks: implementation scope and acceptance criteria.
- Audits: evidence and architectural rationale.
