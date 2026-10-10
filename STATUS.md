# Project Status

## Source project
- Game repository: `Aizekhan/MythHunter`
- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- EPIC 04 candidate audit: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`

## Current position
- **Active task: EPIC 05.1 — validation API candidate audit.** Logging slice is merged and validated.
- Logging PR #20: https://github.com/Aizekhan/MythHunter/pull/20
- Merge commit to `dev`: `ec967a2944ba83db69655d4415d416f49a2ce122`.
- CI on pre-merge head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`: static boundary OK; 13 Framework/Test asset GUIDs checked; .NET tests 9 passed, 0 failed, 0 skipped; no C# compiler warnings.
- Unity CLI EditMode, Unity `6000.0.45f1`, XML `D:\\MythHunter-Git\\EPIC-05-1-Logging-EditMode.xml`: **10 passed, 0 failed, 0 skipped**. Logging test discovered and passed: `RPGFramework.Logging.Tests.LoggerContractTests.Log_PreservesSeverityMessageCategoryAndException`.
- User confirmed Unity full-project compile/Console and game bootstrap smoke are clean; source inspection verifies same logger instance bound under both `IMythLogger` and `RPGFramework.Logging.ILogger`.
- Validation source audit: `IValidator<T>` has only its declaration found in search; concrete `Validator<T>` and mutable `ValidationResult` currently live in MythHunter Utils. No Framework-neutral extraction until callers/contracts have defined value semantics.

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

## Validation contract audit (current focus)
- `IValidator<T>` is declared in `Assets/_MythHunter/Code/Utils/Validation/IValidator.cs` and returns `ValidationResult`.
- `Validator<T>` is a concrete builder that aggregates rule errors and short-circuits on `IsCritical`; `ValidationResult.Errors` is a mutable `List<string>`.
- Repository search found no explicit users of `IValidator<T>` outside its declaration; avoid extracting the interface alone from its current result type.
- Direct validation-style methods exist in game/resource code (e.g. `StaticData.Validate()`, `PreloadSceneConfig.ValidateConfig()`); these are separate APIs and do not yet establish that generic `IValidator<T>` needs Framework ownership.
- Next: enumerate concrete `Validator<T>` instantiations and `ValidationResult` references, determine whether this generic API is used at all, and record decision before any source migration.

## Rules
1. One active roadmap item at a time.
2. Do not reset, clean, or discard local worktree changes.
3. Do not modify `dev` directly.
4. Framework runtime/contracts must not depend on MythHunter, UnityEngine, UnityEditor, DI, logging sinks, or providers.
5. Record actual validation evidence; successful .NET CI does not imply Unity assembly import/compile passed.
6. After Unity validation passes, merge PR #20, finish the validation-candidate audit for EPIC 05.1, and keep moving along the master plan.

## Next action
Run Unity EditMode on branch `feature/epic-05-1-logging-contract` and inspect the new logging test assembly. If Unity compile and EditMode pass, check bootstrap smoke, update this status with the XML evidence, then mark PR #20 ready and merge. Continue the validation API only after logging integration is verified.

## Source of truth
- Master plan: milestone ordering and checklist.
- `STATUS.md`: current active task.
- `Tasks/`: scope and acceptance criteria.
- `Audits/`: source evidence and architectural decisions.
