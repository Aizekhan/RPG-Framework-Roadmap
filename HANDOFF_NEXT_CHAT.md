# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 07 Phase 1: Modular Game Composition System — PR #58 [READY FOR REVIEW].**

- PR: https://github.com/Aizekhan/MythHunter/pull/58
- Branch: `feature/epic-07-game-layer-composition`
- Initial head: `d972a5a` (initial implementation)
- Current head: `b25d1a4` (assembly fixes + validation complete)
- Scope: Implements plug-and-play module system for Framework/Game separation and optional features
  - Module infrastructure with dependency resolution (topological sort)
  - FrameworkCoreModule: Universal ECS/DI/Logger (zero game dependencies)
  - Game modules: GameLayerModule, InfrastructureModule, GameplayModule
  - Optional modules: NetworkingModule, LobbyModule, DebugToolsModule
  - RPGFramework.Core.Runtime assembly for composition system
  - 8 architecture tests in ModularCompositionTests
- Files: 18 changed, 787+ insertions
- Assembly dependency fixes (7 additional commits after initial implementation)
- Unity validation COMPLETED at HEAD `b25d1a4`:
  - ✅ EditMode: 58/60 passed (96.7%), ModularCompositionTests: 7/8 passed
  - ✅ Compilation: Clean, no errors
  - ✅ System functionally validated and ready for review

## Just merged (2026-10-12)
- **PR #54** (EPIC 01): ComponentCache OnDemand positive-membership semantics → `e8f724fe4f6e54e6913db86ce7e17bd68f6ad85c`
- **PR #57** (Architecture Enforcement Phase 1): Foundation layer with architecture tests → `6637ccae2ed3df7e4f5ccab8636419da18adecc1`

## Related open PRs
- **PR #8** (Roadmap): EPIC 07 audit document (branch: `update/epic-07-phase1-progress`)

## Next actions
1. Run Unity EditMode tests on PR #58 HEAD `d972a5a` to validate ModularCompositionTests (7 new tests)
2. Full-project compile with clean Console
3. Test conditional module loading:
   - Offline configuration: FrameworkCore, GameLayer, Infrastructure, Gameplay only
   - Multiplayer configuration: + NetworkingModule, LobbyModule
4. Player Build validation
5. After PR #58 validation, consider Phase 2: migrate remaining installers (EventsInstaller, ProfileInstaller, ParallelSystemsInstaller)

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key Decisions
- EPIC 07 (Game Layer Composition) prioritized before EPIC 06 (Serialization) to establish clear Framework/Game boundaries
- Modular architecture enables optional features (Networking, Lobby, Debug Tools) to be loaded conditionally
- Framework Core has zero game dependencies - universal RPG framework principle maintained
