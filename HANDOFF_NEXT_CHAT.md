# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 05.9 bounded implementation: OnDemand positive-membership semantics — PR #54.**

- PR: https://github.com/Aizekhan/MythHunter/pull/54
- Branch: `fix/epic-05-9-ondemand-positive-membership`
- Exact head: `aaaaee9f864c66a746d9c64a0c7dcfaa6b8249db`
- Change: OnDemand `ComponentCache<T>.Contains` now queries authoritative `IEntityManager.HasComponent<T>` even for IDs not previously cached; refreshes cache on positive membership; removes stale cache state on absence.
- Added test: `ComponentCache_OnDemandContainsFindsUncachedAuthoritativeMembership`.
- CI runs on exact head are in progress: RPGFramework ECS Runtime [38150268459](https://github.com/Aizekhan/MythHunter/actions/runs/38150268459), Minimal CI [38150268442](https://github.com/Aizekhan/MythHunter/actions/runs/38150268442).
- Keep draft until focused Unity EditMode execution is confirmed; do not report pass from CI alone.

## Just merged
- PR #53 merge commit: `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`. Persists movement/path/stamina struct changes from `MovementSystem.Update()` to the authoritative `IEntityManager`; avoids restoring stale values after `StopMovement`. CI passed on tested PR head `ab5b7c386163c8a1fcb7314840b8b223acd2868d`, but Unity validation was not run before the user's instruction to merge. This caveat must remain explicit.
- PR #50 merge commit: `0af6ab74f8236b5de7455349c795377c883b5295` — fixes editor-only GUI compile reference in `PreloadDebugTool`.
- PR #48 merge commit: `0d97060d45d978cafbe376cd43f00eb8fdf9f175` — ComponentCache consistency/reconciliation changes.

## EPIC 01 gates remain open
- Exact current-`dev` Unity EditMode report; historical 44/44 belongs to local SHA `40b6296`.
- Full-project compile and clean Console; Player Build; Movement/Visibility/Hero smoke.
- Complete reverse-reference checks for cache registries/extensions, asmdef references and Unity scene/prefab serialized references.
- Do not mark EPIC 01 complete or start broad ECS extraction.

## Next actions
1. Inspect the completion results for both PR #54 CI runs and verify they refer to exact head `aaaaee9f864c66a746d9c64a0c7dcfaa6b8249db`.
2. Arrange focused Unity EditMode for `ComponentCache_OnDemandContainsFindsUncachedAuthoritativeMembership` without repeating the prior blind 600-second loop.
3. If verified, update PR #54 with exact evidence and evaluate full-project compile/Player/smoke gates.
4. After #54, continue one bounded cache contract at a time; the next candidates are OnDemand enumeration semantics and cache-local mutation vs authoritative writes.

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- EPIC 05.9 remains bounded; EPIC 01 validation remains open.
