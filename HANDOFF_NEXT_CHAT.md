# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 01 Phase 1 reference-validation closeout.** PR #48 and the prerequisite compile fix PR #50 are now merged; phase exit gates are still open.

- PR #50 merge: `0af6ab74f8236b5de7455349c795377c883b5295` — editor-only GUI calls in `PreloadDebugTool.RenderGUI` guarded by `#if UNITY_EDITOR`; `UnityEditor.EditorStyles.boldLabel` qualified.
- PR #48 merge: `0d97060d45d978cafbe376cd43f00eb8fdf9f175` — ComponentCache membership reconciliation/rotating batches/OnDemand live reads and characterization tests.
- Source runtime merge head was `0d97060d45d978cafbe376cd43f00eb8fdf9f175`; documentation-only PR #52 is now merged and current `dev` is `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- Validation caveat: local Unity EditMode reports 44/44 for local HEAD `40b6296`, not exact PR head `6631edf`; workflows pass but do not compile Unity. Full-project compile/Console, Player Build, and Movement/Visibility/Hero smoke remain unconfirmed at current dev head.
- Unity CLI EditMode attempt on isolated PR #50 worktree timed out after 600 seconds with no NUnit XML. This is inconclusive. Diagnostics found no project-specific Unity test process and an unchanged `Editor.log` for 15 seconds; do not kill CLI server PID 18080 or delete caches.
- Correct CLI usage verified from `unity --help`: `unity recompile --project-path <path>`; for tests use positional project argument.

## Confirmed ComponentCache consumer map
- `MovementSystem`: directly constructs caches for `PositionComponent`, `MovementComponent`, `PathComponent` and updates them per active frame.
- `VisibilitySystem`: directly constructs caches for `PositionComponent`, `VisibilityComponent` and updates them in each update.
- `HeroSystem`: obtains `StatsComponent` and `HealthComponent` caches from `IComponentCacheRegistry` and calls `Update()` on each system update.
- `EntitiesInstaller` binds `IComponentCacheRegistry` to `ComponentCacheRegistry`.
- PR #52 merged audit reconciliation docs at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- These are remote source-inspection facts, not proof of Unity scene/prefab references or runtime behavior.

## First actions
1. Preserve local state in `D:\\MythHunter-Git`; do not reset/clean/discard or switch branches unsafely.
2. On exact current `dev` head `0d97060d45d978cafbe376cd43f00eb8fdf9f175`, verify full Unity compilation/clean Console, EditMode XML, Player Build, and Movement/Visibility/Hero smoke. Do not reuse the old 44/44 result as exact-head evidence.
3. Complete remaining reverse-reference coverage using local `rg`/IDE Find All References, asmdef references, and Unity scene/prefab serialized-reference review for `IComponentCacheRegistry`, `GetCache<T>`, `FindEntities<T>`, `ForEach<T>`, and cache extension methods.
4. Update `MythHunter/ARCHITECTURE_AUDIT.md` / related audit via PR after confirming source/scene reference evidence. Keep findings distinct from assumptions.
5. Only when EPIC 01 Phase 1 exit gates are met, activate EPIC 05.9. Scope begins with the ownership/lifecycle/storage/query model and the now-merged ComponentCache behavior; do not broad-refactor or remove caches without consumer/performance evidence.

## Hard guardrails
- Exactly one ACTIVE roadmap item.
- `STATUS.md` is operational pointer; master plan is canonical scope/history.
- Never write directly to `dev`; all subsequent source/doc changes go through PRs.
- Never reset, clean, or discard local user files.
- A CI pass, docs change, timeout, or test report from another SHA is not Unity validation for the current SHA.
- EPIC 05.9 remains blocked until EPIC 01 Phase 1 is closed.
