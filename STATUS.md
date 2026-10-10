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
- Status: EPIC 03.5 merged to `dev` as `b6e046895938d2dd72026827c4f72fd44ced1fdf` ([PR #18](https://github.com/Aizekhan/MythHunter/pull/18)). Unity CLI EditMode on migration source `21d8d89938bc4c165aa485ae3abf96e003744cc1`: 9 passed, 0 failed, 0 skipped (8 Framework ECS tests + 1 Addressables stub). User confirmed Unity Console/full project compilation and game bootstrap/ECS smoke are OK. PR #19 now targets `dev`; merge-history alignment commit `fde1beba9940dc6647ac18244ee78fa973c8d31c` changed ancestry only, not the source tree. Refreshed CI is running; do not merge PR #19 until both workflows pass.
- PR: [#18 — EPIC 03.5: standalone RPGFramework ECS runtime and tests](https://github.com/Aizekhan/MythHunter/pull/18)
- Merged PR #18 head: `4385e0f23729f37749344c7ad33d2861e2003125`; merge commit: `b6e046895938d2dd72026827c4f72fd44ced1fdf`.
- Current-head workflows: [RPGFramework ECS Runtime #19](https://github.com/Aizekhan/MythHunter/actions/runs/38070722424) and [Minimal CI #236](https://github.com/Aizekhan/MythHunter/actions/runs/38070722498), both success.
- PR #18 is merged. Post-merge Minimal CI #237 passed; RPGFramework ECS Runtime #36 was still running at last check.
- Migration branch: `feature/epic-03-6-ecs-consumer-migration`; draft PR [#19](https://github.com/Aizekhan/MythHunter/pull/19) now targets `dev` and isolates 50 migration files (87 additions, 244 deletions). Alignment commit `fde1beba9940dc6647ac18244ee78fa973c8d31c` records `dev` as a second parent after PR #18's squash merge; source tree unchanged.
- EPIC 03.6 source migration removes duplicate legacy ECS types, binds Framework `EntityManager` in the composition root, migrates consumers, and updates Editor generation. Original CI [#35](https://github.com/Aizekhan/MythHunter/actions/runs/38073513821) passed; refreshed checks on aligned head `fde1beba9940dc6647ac18244ee78fa973c8d31c`: [RPGFramework ECS Runtime #37](https://github.com/Aizekhan/MythHunter/actions/runs/38075216261) and [Minimal CI #238](https://github.com/Aizekhan/MythHunter/actions/runs/38075216339).

## Implemented on the feature branch
- Standalone `RPGFramework.ECS.Runtime` assembly with no assembly references and `noEngineReferences: true`.
- Pure C# `IComponent`, `IEntityManager` and dictionary-backed `EntityManager`.
- Separate Unity EditMode test assembly with eight NUnit tests.
- .NET 8 test harness and GitHub Actions workflow.
- Static boundary validator for asmdef settings, forbidden runtime references, Unity metadata and GUID uniqueness.
- `AddComponent` rejects unknown or destroyed IDs with `ArgumentException`; regression coverage included.
- Legacy MythHunter ECS contract/manager duplicates are removed in migration PR #19; awaiting refreshed CI before merge.
- Five unrelated Editor/runtime portability edits were removed from PR #18.

## Validation Evidence
- Tested source commit: `3917e6a48344d4469fd95a2049b1a2e60d983223`.
- GitHub Actions on that source commit: success; static boundary validation passed; .NET compile/NUnit 8 passed, 0 failed, 0 skipped.
- Unity CLI on that source commit, Unity `6000.0.45f1`: `RPGFramework.ECS.Runtime.Tests.dll` discovered; all 8 `RPGFramework.ECS.Tests.EntityManagerTests` passed; total EditMode run 9 passed, 0 failed, 0 skipped.
- PR #18 merge commit: `b6e046895938d2dd72026827c4f72fd44ced1fdf`; post-merge Minimal CI #237 passed. Latest migration-head Unity EditMode: `EPIC-03-6-EditMode-latest.xml`, 9 passed, 0 failed, 0 skipped.

## Rules
1. Do not reset, clean or discard user's local worktree changes.
2. Do not modify `dev` directly.
3. Keep Framework runtime independent of MythHunter, UnityEngine, UnityEditor, logging, DI and providers.
4. Do not leave duplicate legacy and Framework ECS implementations as the final architecture.
5. Do not return to broad diagnostics; validate only concrete source changes.
6. Track one active roadmap task at a time.

## Next Action
Wait for refreshed CI on PR #19 aligned head `fde1beba9940dc6647ac18244ee78fa973c8d31c`; boundary validation passed and .NET ECS tests were running at last check. If both workflows pass, mark PR #19 ready and merge (user authorized progression and confirmed Unity compile/bootstrap smoke). Then record EPIC 03.6 complete and activate EPIC 04 — Framework Core Contracts and Shared Primitives.

## Source of Truth
- Master Plan: ordering and checklist.
- STATUS.md: exact active task.
- Tasks: implementation scope and acceptance criteria.
- Audits: evidence and architectural rationale.
