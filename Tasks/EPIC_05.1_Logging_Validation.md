# EPIC 05.1 — Neutral Logging Contract and MythHunter Adapter

## Status
**SOURCE SLICE IMPLEMENTED — .NET/static CI PASS; Unity validation pending.** Branch: `feature/epic-05-1-logging-contract`; current head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`. Draft PR #20: https://github.com/Aizekhan/MythHunter/pull/20. CI #41 succeeded: static boundary OK, 13 Framework/Test asset GUIDs checked, 9 .NET tests passed (8 ECS + 1 Logging), 0 failed, 0 skipped; nullable compiler warnings were fixed and the latest run has no C# compiler warnings. Do not merge until Unity import/compile, logging EditMode test, and bootstrap smoke are verified.

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

## Next step
Run Unity EditMode on `feature/epic-05-1-logging-contract`, inspect `RPGFramework.Logging.Runtime.Tests.dll` and confirm the one logger contract test passes. Check Unity Console/full project compile and confirm bootstrap registers the same logger under both interfaces. Then update this record with the XML evidence and merge PR #20. Afterward, audit actual callers of `IValidator<T>` / `ValidationResult` before choosing whether to extract a neutral validation contract.
