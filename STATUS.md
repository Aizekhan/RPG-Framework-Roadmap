# Project Status

## Source project
- Game repository: `Aizekhan/MythHunter`
- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- EPIC 04 candidate audit: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`

## Current position
- **EPIC 04 is COMPLETE** — the candidate audit is recorded at `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`. We did not skip it; the decision was to avoid a speculative empty `RPGFramework.Contracts` assembly.
- **Active task: EPIC 05.2 — DI scope lifecycle and disposal semantics.** PR #21 is merged; PR #22 is open as a draft and awaits Unity EditMode plus full-project/bootstrap validation.
- Logging PR #20: https://github.com/Aizekhan/MythHunter/pull/20
- Merge commit to `dev`: `ec967a2944ba83db69655d4415d416f49a2ce122`.
- CI on pre-merge head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`: static boundary OK; 13 Framework/Test asset GUIDs checked; .NET tests 9 passed, 0 failed, 0 skipped; no C# compiler warnings.
- Unity CLI EditMode, Unity `6000.0.45f1`, XML `D:\\MythHunter-Git\\EPIC-05-1-Logging-EditMode.xml`: **10 passed, 0 failed, 0 skipped**. Logging test discovered and passed: `RPGFramework.Logging.Tests.LoggerContractTests.Log_PreservesSeverityMessageCategoryAndException`.
- User confirmed Unity full-project compile/Console and game bootstrap smoke are clean; source inspection verifies same logger instance bound under both `IMythLogger` and `RPGFramework.Logging.ILogger`.
- Validation API audit: no external production callers of `IValidator<T>`/`Validator<T>` found; no Framework validation extraction justified. See `Audits/EPIC_05.1_Validation_API_Candidate_Audit.md`.
- DI source audit: see `Audits/EPIC_05.2_DI_Candidate_Audit.md`. Characterization fixes are underway and must be backed by Unity validation.

## Completed roadmap work
### EPIC 03.5 — Standalone Framework ECS runtime
- PR #18 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/18
- Merge commit: `b6e046895938d2dd72026827c4f72fd44ced1fdf`
- Post-merge CI #36 and #237 passed.
- Unity EditMode on source commit `3917e6a48344d4469fd95a2049b1a2e60d983223`: all 8 ECS tests passed; full EditMode run 9/9 passed.

### EPIC 03.6 — ECS consumer migration
- PR #19 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/19
- Merge commit: `d3b81484f42d084cae150b5b60fc64185cfb70c2`
- Refreshed branch CI #37 and #238 passed; post-merge CI #38 and #239 passed.
- Unity EditMode on migration source `21d8d89938bc4c165aa485ae3abf96e003744cc1`, Unity `6000.0.45f1`: 9 passed, 0 failed, 0 skipped (8 Framework ECS + 1 Addressables test).
- User confirmed Unity Console/full-project compile and game bootstrap/ECS smoke were OK.
- Removed duplicate legacy `IComponent`, `IEntityManager`, and `EntityManager`; MythHunter consumers now use `RPGFramework.ECS`. Game-owned `EcsWorld` and system lifecycle remain in the Game Layer.

### EPIC 04 — Core contract candidate audit
- Audit complete: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`
- Decision: do not create a speculative empty `RPGFramework.Contracts` assembly. ECS contracts already have a single canonical owner; other candidates have game-specific coupling or lack demonstrated cross-module use.
- First implementation slice selected from evidence: neutral logging API with MythHunter compatibility facade, under EPIC 05.1.

## Completed EPIC 05.1 logging slice
- Added Framework `RPGFramework.Logging.Runtime` with `ILogger`, `LogSeverity`, and an assembly definition with no refs and `noEngineReferences: true`.
- `IMythLogger` extends the neutral contract while retaining legacy methods.
- `MythLogger` maps neutral severities/exceptions to existing MythHunter behavior.
- `GameBootstrapper` binds the same logger object under both interfaces.
- CI: static boundary/GUID validation passed; .NET 9/9. Unity EditMode: 10/10; Logging contract test passed. User confirmed compile and bootstrap smoke.
- PR #20 merged to `dev`: `ec967a2944ba83db69655d4415d416f49a2ce122`.

## EPIC 05.1 — complete
- Logging contract implemented, CI validated and PR #20 merged as `ec967a2944ba83db69655d4415d416f49a2ce122`.
- Unity EditMode: 10 passed, 0 failed, 0 skipped; Logging contract test passed.
- Validation candidate audit completed; no extraction justified without real caller demand. See `Audits/EPIC_05.1_Validation_API_Candidate_Audit.md`.

## EPIC 05.2 — active
- DI audit: `Audits/EPIC_05.2_DI_Candidate_Audit.md`.
- PR #21 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/21
- Merge commit: `f57cebb9e0dba11c3e6fbeefc8620627a9363428`.
- User confirmed the Unity tests and project work on PR #21; GitHub Actions #44 passed Framework boundary/.NET checks. Exact Unity XML totals were not provided.
- The merged slice added eight DI characterization tests and fixed scoped instance caching/injection, `Resolve(Type)`, and `IsRegistered(Type)`.
- **Current PR #22:** https://github.com/Aizekhan/MythHunter/pull/22
- Branch/head: `feature/epic-05-2-di-scope-lifecycle` / `a48002f8e2e2cd7690f4f338796dcad2288c5a3f`.
- Changes: isolate worker-thread scope from the container thread's scope; rename misleading `_asyncLocalScope` field to `_threadLocalScope`; deterministic reverse-order cleanup and deduplication of scope-owned disposables; 4 more EditMode tests.
- CI boundary/.NET run #38121408276 passed; Minimal CI #38121408235 passed as a no-op. Neither runs Unity tests.
- **Unity EditMode/full-project compile/bootstrap verification are still pending for PR #22. Keep it draft.**

## EPIC 04 clarification
- EPIC 04 was an audit/decision milestone, not a requirement to create a new assembly. The completed audit chose not to create `RPGFramework.Contracts` without demonstrated shared consumers.
- The actual source implementation sequence is EPIC 03.5/03.6 (ECS runtime and migration), EPIC 04 audit, then EPIC 05.1 logging and EPIC 05.2 DI. No EPIC 04 implementation task was silently skipped.

## Rules
1. One active roadmap item at a time.
2. Do not reset, clean, or discard local worktree changes.
3. Do not modify `dev` directly.
4. Framework runtime/contracts must not depend on MythHunter, UnityEngine, UnityEditor, DI, logging sinks, or providers.
5. Record actual validation evidence; successful .NET CI does not imply Unity assembly import/compile passed.
6. EPIC 04 and EPIC 05.1 are complete; EPIC 05.2 is active. PR #21 is merged; PR #22 remains draft pending Unity validation. Preserve all local worktree changes and use focused tests for the DI lifecycle slice.

## Next action
Run Unity EditMode on `feature/epic-05-2-di-scope-lifecycle`, verify all 12 DI tests are discovered and pass, check full-project compile/Console and game bootstrap/scope smoke. Then review PR #22 and decide merge. Leave async context propagation and transient/singleton disposal policy deferred.

## Source of truth
- Master plan: milestone ordering and checklist.
- `STATUS.md`: current active task.
- `Tasks/`: scope and acceptance criteria.
- `Audits/`: source evidence and architectural decisions.
