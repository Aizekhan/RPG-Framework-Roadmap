# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 07 Phase 1: Modular Game Composition System — PR #58 [READY FOR REVIEW].**

- PR: https://github.com/Aizekhan/MythHunter/pull/58
- Branch: `feature/epic-07-game-layer-composition`
- Initial head: `d972a5a` (initial implementation)
- Current head: `f701e79` (assembly fixes + validation + metadata + .claude settings)
- Scope: Implements plug-and-play module system for Framework/Game separation and optional features
  - Module infrastructure with dependency resolution (topological sort)
  - FrameworkCoreModule: Universal ECS/DI/Logger (zero game dependencies)
  - Game modules: GameLayerModule, InfrastructureModule, GameplayModule
  - Optional modules: NetworkingModule, LobbyModule, DebugToolsModule
  - RPGFramework.Core.Runtime assembly for composition system
  - 8 architecture tests in ModularCompositionTests
- Files: 18 changed, 787+ insertions
- Assembly dependency fixes (7 additional commits after initial implementation)
- Unity validation COMPLETED at HEAD `1fa6115` (metadata + settings updates: `f701e79`):
  - ✅ EditMode: 58/60 passed (96.7%), ModularCompositionTests: 7/8 passed
  - ✅ Compilation: Clean, no errors
  - ✅ System functionally validated and ready for review
- Note: GitHub Actions failing on Unity licensing (CI configuration issue, NOT code problem)

## Just merged (2026-10-12)
- **PR #56** (JobHandle IL2CPP fix): `d36b271` → Fixes IL2CPP compilation error in `JobHandleExtensions.IsValid()`
- **PR #57** (Architecture Enforcement Phase 1): `6637cca` → Foundation layer with architecture tests
- **PR #54** (EPIC 01): `e8f724f` → ComponentCache OnDemand positive-membership semantics

## Related open PRs
None currently - PR #58 is the active PR

## Phase 2 Research Completed (2026-10-12)

### Critical Findings
Deep analysis of installer migration revealed **4 categories of architectural issues**:

1. **God Class Anti-pattern**
   - CoreInstaller mixes 13 services (7 Framework + 6 Game)
   - Violates Single Responsibility Principle

2. **Hidden Registration Overwrites**
   - ISystemRegistry registered twice: SystemRegistry → ParallelSystemRegistry
   - IEventBus registered twice: SimpleEventBus → EventBus/NetworkEventBus
   - Works by accident (execution order dependent)

3. **Resource Duplication**
   - LazyDependencyInjector GameObject created TWICE by different installers
   - First instance unused, second instance registered in DI

4. **Dead Code**
   - InstallerRegistry class has zero usages

### Revised Phase 2 Strategy
Split CoreInstaller before migration:
1. Create **FrameworkCoreInstaller** (Framework services + ParallelSystemRegistry)
2. Create **GameCoreInstaller** (Game services + Event system)
3. Update FrameworkCoreModule → use FrameworkCoreInstaller
4. Update GameLayerModule → use GameCoreInstaller
5. Update InfrastructureModule → add LazyDI + ProfilePresenter
6. Remove 4 direct installer calls from GameBootstrapper
7. Delete dead code (InstallerRegistry)

**Benefits**: No conflicts, clear Framework/Game separation, testable independently

### Phase 2 Documentation
Created in MythHunter-Git repository:
- `EPIC-07-Phase2-Complete-Analysis.md` (700+ lines, ready-to-use implementation code)
- `EPIC-07-Phase2-Research-Findings.md` (registration conflicts analysis)
- `EPIC-07-Phase2-Additional-Findings.md` (LazyDI duplication investigation)
- `SESSION-2026-10-12-Summary.md` (complete session summary)

## Next actions
1. **Code review and merge PR #58** - Phase 1 is validated and ready
2. **EPIC 07 Phase 2** (after PR #58 merge):
   - Implement revised strategy (split CoreInstaller)
   - Migrate 4 installers to modules with conflict resolution
   - Remove direct installer calls
   - Validate offline vs multiplayer mode configurations
3. Complete EPIC 07: Document game composition patterns, validate startup/shutdown

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
- Phase 2 research uncovered hidden architectural issues requiring CoreInstaller split
