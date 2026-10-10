# EPIC 05.1 — Neutral Logging Contract and MythHunter Adapter

## Status
**EPIC COMPLETE — logging merged and validated; validation candidate audit concluded no extraction is justified by current evidence.** PR #20: https://github.com/Aizekhan/MythHunter/pull/20; merge commit `ec967a2944ba83db69655d4415d416f49a2ce122`. CI on source head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`: static boundary/GUID checks passed, 9 .NET tests passed, 0 failed/skipped, no compiler warnings. Unity `6000.0.45f1` EditMode XML `D:\\MythHunter-Git\\EPIC-05-1-Logging-EditMode.xml`: 10 passed, 0 failed, 0 skipped; the Logging contract test was discovered and passed. User confirmed full-project compile/Console and game bootstrap smoke clean.

## Parent
- EPIC 05 — Framework Runtime Modules
- Follow-up to EPIC 04 candidate audit: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`.

## Source evidence
- `Assets/_MythHunter/Code/Utils/Logging/IMythLogger.cs` defines MythHunter-specific `LogLevel` and a broader logger API.
- `Assets/_MythHunter/Code/Utils/Logging/MythLogger.cs` owns Unity Console output, color/category formatting, context, enrichers and file rotation. These are adapter/game-layer concerns, not Framework runtime.
- `Assets/_MythHunter/Code/Core/Game/GameBootstrapper.cs` creates the logger and binds it to the DI container as `IMythLogger`.
- Logger consumers span game systems, DI, ECS optimizers/factories, events, UI, debug tools, resource pools and scene loading. A compatibility facade is required; do not rename all call sites in this first slice.
- The current `IValidator<T>` / `ValidationResult` surface has not been demonstrated as a used cross-module contract. Defer its redesign until actual callers and semantics are mapped.

## Scope
1. Add `RPGFramework.Logging.Runtime` with only `ILogger` and `LogSeverity`, no assembly references and `noEngineReferences: true`.
2. Make existing `IMythLogger` extend `RPGFramework.Logging.ILogger`, preserving all existing methods and its current MythHunter `LogLevel`.
3. Implement severity/exception adaptation in `MythLogger`. Keep UnityEngine, file paths, color formatting, file rotation and context enrichers out of Framework.
4. Register the same logger object under both `IMythLogger` and Framework `ILogger` in `GameBootstrapper`.
5. Add an Editor-only Unity test assembly and a .NET harness test for the neutral contract; extend the static boundary/GUID validator and workflow path filters to include Logging.
6. Do not migrate all consumers, extract DI, or invent a logging provider registry in this slice.

## Acceptance criteria
- The Framework Logging runtime contains no Unity, MythHunter, DI or provider references.
- Exactly one `LogSeverity` to MythHunter `LogLevel` mapping exists in the adapter: Trace→Trace, Debug→Debug, Information→Info, Warning→Warning, Error→Error, Critical→Fatal.
- Existing `IMythLogger` members remain source-compatible.
- DI resolves the same `MythLogger` instance as both `IMythLogger` and `RPGFramework.Logging.ILogger`.
- Static boundary/GUID validation and .NET ECS+Logging tests pass in CI.
- Unity imports/compiles, discovers the logging test assembly and runs EditMode tests.
- Preserve all existing Unity .meta identities; new asset GUIDs are unique.
- Update this task, `STATUS.md`, and the master plan with actual test evidence before closing the item.

## Rollback
Revert only this feature branch/PR. The MythHunter-facing API remains intact, so rollback must not require mass caller edits. Do not reset/clean the user's local worktree.

## Validation API audit — decision
See `Audits/EPIC_05.1_Validation_API_Candidate_Audit.md`.
- Repository search found no actual production consumers of the generic `IValidator<T>` / `Validator<T>` builder outside its own implementation.
- `ValidationResult` is mutable and includes game-specific critical short-circuit behavior; moving the interface alone would be incomplete.
- Decision: keep existing validation files untouched. Do not introduce a neutral Framework validation assembly until real consumer demand and requirements justify it.

## Outcome
EPIC 05.1 is complete. The next active item is EPIC 05.2 — Dependency Injection audit/design.
