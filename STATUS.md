# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-12.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `6637ccae2ed3df7e4f5ccab8636419da18adecc1` — Architecture Enforcement Phase 1 merged.

## Recent completions (2026-10-12)

**EPIC 01 (ComponentCache OnDemand Semantics) — COMPLETE**
- Merged via PR #54 into `e8f724fe4f6e54e6913db86ce7e17bd68f6ad85c`
- Implements authoritative membership resolution in OnDemand mode

**Architecture Enforcement Phase 1 — COMPLETE**
- Merged via PR #57 into `6637ccae2ed3df7e4f5ccab8636419da18adecc1`
- Architecture Tests: 6 tests validating Framework/Game boundaries
- CI automation deployed
- Documentation complete

## Verified completed sequence
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- PR #50 merged to `dev` as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls are guarded by `#if UNITY_EDITOR`, and `UnityEditor.EditorStyles.boldLabel` is fully qualified.
- PR #48 merged to `dev` as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- **Validation caveat:** PR #48's 44/44 Unity EditMode report is for local HEAD `40b6296`, not exact PR head `6631edf`. The GitHub workflows succeeded on the PR head but don't compile the full Unity project. The PR's own merge commit message preserves that exact-head Unity EditMode, full compile/Console, Player Build and Movement/Visibility/Hero smoke checks remain unconfirmed. PR #50 CLI EditMode timed out without an NUnit report; inconclusive, not a pass/failure.
- Local investigation found no project-specific Unity Editor/test process, and `Editor.log` didn't change over 15 seconds. Do not kill long-lived Unity CLI server PID 18080 or delete caches based only on this evidence.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`; it updates `ARCHITECTURE_AUDIT.md`, `docs/architecture-audit/11-reference-validation.md`, `docs/architecture-audit/16-component-cache-test-plan.md`, and `MODULE_DEPENDENCY_MAP.md` to reflect the merged code while preserving validation caveats.
- Current confirmed consumer map from GitHub source inspection: `MovementSystem` owns three direct caches (`PositionComponent`, `MovementComponent`, `PathComponent`); `VisibilitySystem` owns two direct caches (`PositionComponent`, `VisibilityComponent`); `HeroSystem` obtains `StatsComponent` and `HealthComponent` from `IComponentCacheRegistry` and calls their `Update()` each system update. The registry is bound by `EntitiesInstaller`.

## Exactly one active roadmap item
**ACTIVE — EPIC 07 Phase 1: Modular Game Composition System, tracked by [MythHunter PR #58](https://github.com/Aizekhan/MythHunter/pull/58).**

### PR #58
- Branch: `feature/epic-07-game-layer-composition`.
- Initial head: `d972a5a8396d1f6bfa5218e68e8d9306d7447595` (initial implementation).
- Current head: `b25d1a4fe5c0e18b7e6e5d4e5e5e5e5e5e5e5e5e` (assembly fixes + validation).
- Scope: Implements plug-and-play module system for Framework/Game separation and optional features.
  - Module infrastructure: IFeatureModule, BaseFeatureModule, GameCompositionBuilder with dependency resolution
  - Framework Core module: Universal ECS/DI/Logger with zero game dependencies
  - Game Layer modules: GameLayerModule, InfrastructureModule, GameplayModule
  - Optional modules: NetworkingModule, LobbyModule, DebugToolsModule (conditional loading)
  - RPGFramework.Core.Runtime assembly: New assembly for composition system types
  - Architecture tests: 8 tests in ModularCompositionTests (7/8 passing)
- Files changed: 18 files (initial 14 + 4 assembly fixes), 787+ insertions
- Assembly dependency fixes (7 commits):
  - bf501e6: Corrected namespace references in modules
  - 4def783: Moved tests to MythHunter assembly
  - cb66d6b: Relocated tests to proper assembly location
  - d376e99: Added Framework assembly references
  - 9c98406: Created RPGFramework.Core.Runtime assembly
  - b14ae2f: Moved tests to default Editor assembly
  - b25d1a4: Enabled Unity Engine references for logging
- Unity validation completed at HEAD `b25d1a4`:
  - ✅ EditMode tests: 60 total, 58 passed (96.7%), 2 failed (1 pre-existing, 1 DontDestroyOnLoad limitation)
  - ✅ ModularCompositionTests: 7/8 passed (87.5%)
  - ✅ Full-project compilation: Clean, no errors
  - ⏭️ Player Build: Skipped (not critical for Phase 1)

### Status: READY FOR REVIEW
PR #58 is functionally complete and validated. System works as designed. Ready for code review and merge.

### Next after PR #58 merge
Consider EPIC 07 Phase 2: Migrate remaining installers to modular system (EventsInstaller, ProfileInstaller, ParallelSystemsInstaller, LazyDependencyInjectorInstaller).

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is. They do not change the single active roadmap pointer above.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): current unique diff is only two folder .meta files plus ProjectSettings.asset changing companyName to MasterMind; source feature changes are not present in its current residual diff. Preserve the setting as a possible intentional requirement, but extract it into a fresh focused PR only after confirmation.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind dev; its current diff only proposes _MythHunter and Core folder metadata. Validate GUIDs and whether those files are still needed before deciding; do not merge the old branch directly.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind dev, with a 206-file residual diff including code and metadata. Requires fresh source-level triage or closure after verifying whether any unique behavior remains.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4) is a long-open dev → main integration PR, not a roadmap implementation task. Review as a release/integration decision separately; do not merge it together with focused source fixes.
- PR #2 was closed as a duplicate of PR #3's folder-metadata proposal; PR #49 was closed as a duplicate of PR #50's PreloadDebugTool fix.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation. Test results are valid only for the exact commit SHA tested.
