# EPIC 02.9 — Post-Architecture Roadmap Reassessment

## Status
COMPLETE — roadmap restructured and aligned to source-backed execution.

## Decision
The roadmap has been shortened into a dependency-ordered path and the next useful engineering unit is the first reusable ECS contract slice. The plan must not remain stuck in baseline diagnostics when the actual limitation is understood and can be handled by compiling/testing the first bounded change.

The Master Plan now:
- records that local validation was attempted but is not a clean-compile certification;
- preserves all local setup changes rather than requiring a reset;
- avoids pretending package stub tests are MythHunter coverage;
- selects entity identity and minimal ECS contracts as the first source-backed slice;
- defers `EcsWorld` because it currently depends on MythHunter's system registry;
- makes usage mapping and explicit API decisions the active task before source edits.

## Execution order
1. EPIC 03.1 — record the known local baseline evidence and limitations (documented; not a clean compile claim).
2. EPIC 03.2 — map current ECS contract usages and settle identity/component behavior.
3. EPIC 03.3 — create the first bounded and enforceable assembly boundary, add focused tests, compile, and preserve rollback.
4. EPIC 04–05 — extract only required reusable contracts and runtimes.
5. EPIC 06–08 — serialization and Game Layer integration, then residual dependency cleanup.
6. EPIC 09–14 — optional adapters, RPG/gameplay modules, persistence/networking and release/sample work.

## Guardrails
- No broad rewrite or speculative empty modules.
- No source edit until the affected callers and API semantics are understood.
- Preserve existing local changes; never reset the user's worktree to manufacture a clean baseline.
- Source code changes require compile and relevant focused tests. Report the actual outcome, including failures.
- Keep Framework independent from MythHunter; the Game Layer may depend on Framework, never vice versa.
- A documentation update is not implementation proof.

## Conclusion
The roadmap is aligned. EPIC 03.2 is the active task: map entity identity and ECS contract use, decide the minimal neutral API, and prepare one bounded implementation change. Unity diagnostics should be repeated only when they directly block validating that change.
