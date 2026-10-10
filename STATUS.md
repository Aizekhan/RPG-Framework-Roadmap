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
- **Active task: 3.6 — Migrate MythHunter ECS consumers to Framework runtime (validation checkpoint)**
- Status: 3.5 implementation/validation complete; PR #18 awaits formal review/merge. EPIC 03.6 migration code is prepared on a stacked branch; local Unity CLI EditMode result is pending XML inspection. Unity compile and game bootstrap/smoke status are not yet confirmed. The original 3.5 Unity test gate passed on source commit `3917e6a48344d4469fd95a2049b1a2e60d983223`: test assembly discovered, 8 ECS tests passed; total EditMode run 9 passed, 0 failed, 0 skipped. README/PR description was updated in documentation-only commit `4385e0f23729f37749344c7ad33d2861e2003125`; both CI workflows on this latest PR head now passed.
- PR: [#18 — EPIC 03.5: standalone RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Current PR head: `4385e0f23729f37749344c7ad33d2861e2003125`.
- Current-head workflows: [RPGFramework ECS Runtime #19](https://github.com/Aizekhan/MythHunter/actions/runs/38070722424) and [Minimal CI #236](https://github.com/Aizekhan/MythHunter/actions/runs/38070722498), both success.
- PR #18 title is aligned with its ready-for-review state; it has no submitted reviews/inline threads and remains unmerged.
- Stacked preparation branch: `feature/epic-03-6-ecs-consumer-migration`; draft PR [#19](https://github.com/Aizekhan/MythHunter/pull/19) is temporarily based on `feature/framework-ecs-contracts` so its diff isolates the consumer migration. Latest branch CI [#21](https://github.com/Aizekhan/MythHunter/actions/runs/38071622972) passed the static boundary/import checks and .NET ECS tests.
- EPIC 03.6 branch has targeted namespace cleanup across the composition root, component factory contracts, serializers, archetype APIs, and entity factories; the Editor generator now emits plain components against `RPGFramework.ECS`. Latest migration head `f5ed3325a930c61eeab90e287898eb87355d886f`; CI run [#34](https://github.com/Aizekhan/MythHunter/actions/runs/38073434986) passed static boundary checks and .NET ECS tests. This does not establish Unity compilation. The local EditMode XML and full project compile outcome still need inspection, and game bootstrap/ECS smoke validation remains unconfirmed.

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
- Tested source commit: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- GitHub Actions on that source commit: success; static boundary validation passed; .NET compile/NUnit 8 passed, 0 failed, 0 skipped.
- Unity CLI on that source commit, Unity `6000.0.45f1`: `RPGFramework.ECS.Runtime.Tests.dll` discovered; all 8 `RPGFramework.ECS.Tests.EntityManagerTests` passed; total EditMode run 9 passed, 0 failed, 0 skipped.
- Latest PR-head documentation-only commit: `4385e0f23729f37749344c7ad33d2861e2003125`; [RPGFramework ECS Runtime #19](https://github.com/Aizekhan/MythHunter/actions/runs/38070722424) and [Minimal CI #236](https://github.com/Aizekhan/MythHunter/actions/runs/38070722498) are complete and successful.

## Rules
1. Do not reset, clean or discard user's local worktree changes.
2. Do not modify `dev` directly.
3. Keep Framework runtime independent of MythHunter, UnityEngine, UnityEditor, logging, DI and providers.
4. Do not leave duplicate legacy and Framework ECS implementations as the final architecture.
5. Do not return to broad diagnostics; validate only concrete source changes.
6. Track one active roadmap task at a time.

## Next Action
CI and Unity validation gates pass. Finish formal PR review and merge decision for PR #18. Do not activate task 3.6 until PR #18 is reviewed/merged; then migrate MythHunter consumers and remove legacy ECS duplication coherently.

## Source of Truth
- Master Plan: ordering and checklist.
- STATUS.md: exact active task.
- Tasks: implementation scope and acceptance criteria.
- Audits: evidence and architectural rationale.
