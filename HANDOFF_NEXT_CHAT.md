# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 05.9 bounded implementation: MovementSystem authoritative write-through — PR #53.** The user explicitly asked to merge the preceding fixes and continue EPIC 5. This narrow source-backed slice proceeds without declaring EPIC 01 Phase 1 complete; its full Unity/player/consumer validation gates remain open.

- PR #50 merge: `0af6ab74f8236b5de7455349c795377c883b5295` — editor-only GUI calls in `PreloadDebugTool.RenderGUI` guarded by `#if UNITY_EDITOR`; `UnityEditor.EditorStyles.boldLabel` qualified.
- PR #48 merge: `0d97060d45d978cafbe376cd43f00eb8fdf9f175` — ComponentCache membership reconciliation/rotating batches/OnDemand live reads and characterization tests.
- Source runtime merge head was `0d97060d45d978cafbe376cd43f00eb8fdf9f175`; documentation-only PR #52 is merged and current `dev` was `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- Validation caveat: local Unity EditMode reports 44/44 for local HEAD `40b6296`, not exact PR head `6631edf`; workflows pass but do not compile Unity. Full-project compile/Console, Player Build, and Movement/Visibility/Hero smoke remain unconfirmed at current dev head.
- Unity CLI EditMode attempt on isolated PR #50 worktree timed out after 600 seconds with no NUnit XML. This is inconclusive. Diagnostics found no project-specific Unity test process and an unchanged `Editor.log` for 15 seconds; do not kill CLI server PID 18080 or delete caches.
- Correct CLI usage verified from `unity --help`: `unity recompile --project-path <path>`; for tests use positional project argument.

## Active PR #53 — MovementSystem authoritative persistence

- PR: https://github.com/Aizekhan/MythHunter/pull/53
- Branch: `fix/epic-05-9-movement-cache-write-through`
- Exact head: `ab5b7c386163c8a1fcb7314840b8b223acd2868d`
- CI: Minimal CI and RPGFramework ECS Runtime both succeeded on this head; neither executes the Unity Editor test suite.
- Source finding: `MovementSystem.Update()` refreshed its default-Full caches from `IEntityManager` each active tick, but after mutating local struct copies wrote them only to cache-local `Add`; the next tick therefore reloaded stale authoritative values.
- Fix: persist `PathComponent`, `MovementComponent`, and `PositionComponent` to `IEntityManager`, then refresh local caches. If movement points are exhausted, reload components after `StopMovement` writes the authoritative stopped state.
- Regression test: `MovementSystemUpdate_PersistsMutatedComponentsToAuthoritativeEntityManager` checks stored position X=1, movement points=9, and path time=1 after one second of movement.
- **Do not merge #53 until this test executes and passes on its exact head.** Full Unity compilation, Player Build, and Movement/Visibility/Hero smoke tests are not yet confirmed. The old 44/44 report belongs to a different SHA.

## Confirmed ComponentCache consumer map
- `MovementSystem`: directly constructs caches for `PositionComponent`, `MovementComponent`, `PathComponent` and updates them per active frame.
- `VisibilitySystem`: directly constructs caches for `PositionComponent`, `VisibilityComponent` and updates them in each update.
- `HeroSystem`: obtains `StatsComponent` and `HealthComponent` caches from `IComponentCacheRegistry` and calls `Update()` on each system update.
- `EntitiesInstaller` binds `IComponentCacheRegistry` to `ComponentCacheRegistry`.
- PR #52 merged audit reconciliation docs at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- These are remote source-inspection facts, not proof of Unity scene/prefab references or runtime behavior.

## First actions
1. Preserve local state in `D:\\MythHunter-Git`; do not reset/clean/discard or switch branches unsafely.
2. Validate PR #53's focused EditMode regression test on exact head `ab5b7c386163c8a1fcb7314840b8b223acd2868d`; then run full compile/clean Console, Player Build and Movement/Visibility/Hero smoke before merge.
3. Keep EPIC 01 Phase 1's reverse-reference, asmdef and Unity scene/prefab evidence open.
4. After #53 is validated, continue the next bounded EPIC 05.9 item: settle OnDemand positive-membership semantics and registry/cache lifecycle with tests, not a broad rewrite.

## Hard guardrails
- Exactly one ACTIVE roadmap item.
- `STATUS.md` is operational pointer; master plan is canonical scope/history.
- Never write directly to `dev`; all subsequent source/doc changes go through PRs.
- Never reset, clean, or discard local user files.
- A CI pass, docs change, timeout, or test report from another SHA is not Unity validation for the current SHA.
- EPIC 05.9 remains blocked until EPIC 01 Phase 1 is closed.
