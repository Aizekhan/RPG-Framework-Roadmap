# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-13.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `ff9351d` — Latest: [PR #62 EPIC 08 Phase 4](https://github.com/Aizekhan/MythHunter/pull/62).

## Verified completed sequence

### EPIC 05 (Framework Runtime Modules)
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- EPIC 05.9: PR #54 merged as `e8f724f` — ComponentCache OnDemand positive-membership semantics.

### EPIC 07 (Modular Game Composition System) — ✅ COMPLETE
- **PR #58** merged as `0862793` — Phase 1: Module infrastructure, FrameworkCoreModule, GameLayerModule, InfrastructureModule, GameplayModule, NetworkingModule, LobbyModule, DebugToolsModule. Unity EditMode 58/60 (96.7%).
- **PR #59** merged as `92c9abe` — Phase 2: CoreInstaller split into FrameworkCoreInstaller and GameCoreInstaller, module migration, LazyDI deduplication, InstallerRegistry removal. Unity EditMode validated, clean compile.

### EPIC 08 (Dependency Debt and Lifecycle Stabilization) — ✅ COMPLETE
- **PR #60** merged as `8faddb9` — Phases 1 & 2: AsyncTaskTracker for cancellation token lifecycle, system decoupling (ArchetypeSystem, HeroSelectionService, TimerService), EventBus async cancellation, state machine async refactoring.
- **PR #61** merged as `cad8439` — Phase 3: ComponentCache thread safety (lock-free reads, write locks), EventBus duplicate subscription prevention, memory leak fixes (weak references, unsubscribe cleanup).
- **PR #62** merged as `e8bd55a` — Phase 4: DI reflection optimization (type cache, lazy metadata), InitializationTransaction for deterministic rollback. Performance benchmarks show 85-95% improvement.

### Earlier milestones
- PR #50 merged as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls guarded by `#if UNITY_EDITOR`.
- PR #48 merged as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- **Architecture Enforcement Phase 1**: PR #57 merged as `6637cca` — Foundation layer with architecture tests.
- **JobHandle IL2CPP fix**: PR #56 merged as `d36b271` — Fixes IL2CPP compilation error in `JobHandleExtensions.IsValid()`.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- PR #53 merged as `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`: MovementSystem authoritative write-through.

## Exactly one active roadmap item
**ACTIVE — EPIC 06: Serialization Foundation**

### Scope
Audit and extract serialization foundation:
- Audit component/entity/versioned/delta/network serialization separately
- Define stable schema/type identifiers when data crosses version/process boundaries
- Separate generic serialization from persistence policy and network wire formats
- Extract only proven-reusable contracts/runtime and test compatibility

### Status
- [ ] Audit current serialization implementations
- [ ] Define stable schema/type identifiers
- [ ] Separate generic serialization from persistence/network
- [ ] Extract reusable contracts and test

### Next action
Begin EPIC 06 audit: search and map current serialization implementations in MythHunter codebase, identify Framework vs Game boundaries, define extraction scope.

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): residual diff only folder .meta files.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind dev.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind dev.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4): long-open dev → main integration PR.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation. Test results are valid only for the exact commit SHA tested.
