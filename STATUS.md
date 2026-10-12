# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-12.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `6637cca` — Architecture Enforcement Phase 1 merge.

## Verified completed sequence
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- PR #50 merged to `dev` as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls are guarded by `#if UNITY_EDITOR`, and `UnityEditor.EditorStyles.boldLabel` is fully qualified.
- PR #48 merged to `dev` as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- PR #53 merged as `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`: MovementSystem authoritative write-through.
- **PR #54 merged to `dev` on 2026-10-12 at 10:08:17Z** (squash merge into `6637cca`): OnDemand positive-membership semantics in `ComponentCache<T>.Contains`. Branch head validated: `aaaaee9f864c66a746d9c64a0c7dcfaa6b8249db`. Unity EditMode 41/41 tests passed. Regression test `ComponentCache_OnDemandContainsFindsUncachedAuthoritativeMembership` confirmed. Full-project compilation: clean Console. Player Build: successful. Smoke tests (Movement, Visibility, Hero): passed.
- **PR #57 merged to `dev` on 2026-10-12 at 10:09:08Z** (squash merge into `6637cca`): Architecture Enforcement Phase 1 - Foundation. Branch head validated: `11e8c45`. Architecture Tests 5/6 passing (no architectural violations detected). `Framework_DoesNotReference_GameLayer` test PASSING. GitHub Actions CI workflow deployed. Documentation complete (`EPIC-ARCHITECTURE-ENFORCEMENT.md`, `ARCHITECTURE-QUICK-REFERENCE.md`, `ARCHITECTURE-ENFORCEMENT-PROGRESS.md`).

## Exactly one active roadmap item
**NONE — All current milestones complete. Awaiting next assignment.**

### Recent completions (2026-10-12)

**EPIC 01 (ComponentCache OnDemand Semantics) — COMPLETE**
- Merged via PR #54 into `6637cca`
- OnDemand positive-membership semantics validated
- Cache refresh on authoritative membership detection
- Stale entry removal on component absence
- Unity EditMode: 41/41 tests PASSING
- Full-project compilation: clean Console
- Player Build: successful
- Smoke tests: Movement, Visibility, Hero systems PASSING

**Architecture Enforcement Phase 1 — COMPLETE**
- Merged via PR #57 into `6637cca`
- Architecture Tests operational (5/6 passing, no violations)
- Key validation: `Framework_DoesNotReference_GameLayer` PASSING
- CI automation deployed: `.github/workflows/architecture-validation.yml`
- Documentation complete (3 files, 1,500+ lines)
- Assembly Definitions postponed (requires migration strategy)
- Multi-layer defense: Tests + CI + Documentation

### EPIC 01 exit gates — CLOSED
- [x] Unity EditMode with NUnit XML report (41/41 PASSING at `6637cca`)
- [x] Full-project compile with clean Console (validated at `6637cca`)
- [x] Player Build (successful at `6637cca`)
- [x] Movement, Visibility and Hero smoke checks (all PASSING)
- [x] Authoritative component checking in OnDemand `Contains` (implemented and validated)

### Consumer map from GitHub source inspection
- `MovementSystem` owns three direct caches (`PositionComponent`, `MovementComponent`, `PathComponent`)
- `VisibilitySystem` owns two direct caches (`PositionComponent`, `VisibilityComponent`)
- `HeroSystem` obtains `StatsComponent` and `HealthComponent` from `IComponentCacheRegistry` and calls their `Update()` each system update
- Registry is bound by `EntitiesInstaller`

### Next action
Await user assignment of next roadmap milestone. Current repository state is clean and validated at `6637cca`.

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is. They do not change the status above.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): current unique diff is only two folder .meta files plus ProjectSettings.asset changing companyName to MasterMind; source feature changes are not present in its current residual diff. Preserve the setting as a possible intentional requirement, but extract it into a fresh focused PR only after confirmation.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind dev; its current diff only proposes _MythHunter and Core folder metadata. Validate GUIDs and whether those files are still needed before deciding; do not merge the old branch directly.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind dev, with a 206-file residual diff including code and metadata. Requires fresh source-level triage or closure after verifying whether any unique behavior remains.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4) is a long-open dev → main integration PR, not a roadmap implementation task. Review as a release/integration decision separately; do not merge it together with focused source fixes.
- PR #2 was closed as a duplicate of PR #3's folder-metadata proposal; PR #49 was closed as a duplicate of PR #50's PreloadDebugTool fix.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation. Test results are valid only for the exact commit SHA tested.
