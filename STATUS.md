# Project Status

## Source project
- Game repository: `Aizekhan/MythHunter`
- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- EPIC 04 candidate audit: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`

## Current position
- **Active task: EPIC 05.1 — Logging and validation; first slice: neutral Logging contract + MythHunter adapter.**
- Implementation branch: `feature/epic-05-1-logging-contract`
- Draft PR #20: https://github.com/Aizekhan/MythHunter/pull/20
- Current head: `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`
- CI #41: https://github.com/Aizekhan/MythHunter/actions/runs/38075878947 — success.
- Static boundary/GUID validation: OK; 5 Framework runtime C# files and 2 test C# files; 13 Framework/test asset GUIDs checked.
- .NET tests: 9 passed, 0 failed, 0 skipped (8 ECS + 1 Logging contract). Latest run has no C# compiler warnings after nullable annotations were fixed.
- **Still required:** Unity Editor import/full-project compile, confirm `RPGFramework.Logging.Runtime.Tests.dll` is discovered and its EditMode test passes, then game bootstrap smoke check. PR #20 remains draft and must not be merged before these checks.

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

## Current EPIC 05.1 implementation
- Added `Assets/_Framework/Logging/Runtime/ILogger.cs`, `LogSeverity.cs`, and `RPGFramework.Logging.Runtime.asmdef` (no assembly refs, no Unity engine refs).
- Existing `IMythLogger` extends Framework `ILogger`; legacy API is preserved.
- `MythLogger` maps Framework severities/exceptions to existing MythHunter logging behavior.
- `GameBootstrapper` binds the exact same logger object as both `IMythLogger` and `RPGFramework.Logging.ILogger`.
- Added Editor-only Unity test assembly and a .NET harness test; extended static boundary/GUID checks and CI workflow paths.
- Task definition: `Tasks/EPIC_05.1_Logging_Validation.md`
- Scope is intentionally narrow: no mass call-site migration, no DI extraction, no validation-contract source change yet.

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
