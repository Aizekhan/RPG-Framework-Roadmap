# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 06 Phase 1: Component Serialization Foundation — SCOPE TO BE DEFINED.**

### Current Status
All previous epics completed:
- ✅ EPIC 05 (Framework Runtime Modules) - Complete
- ✅ EPIC 07 Phase 1 & 2 (Modular Game Composition System) - Complete
- ✅ EPIC 08 Phases 1-4 (Performance Optimization & Lifecycle Stabilization) - Complete

EPIC 06 is now active. No detailed phases defined yet in master plan.

### EPIC 06 Objectives (from master plan)
- Audit component/entity/versioned/delta/network serialization separately
- Define stable schema/type identifiers when data crosses version/process boundaries
- Separate generic serialization from persistence policy and network wire formats
- Extract only proven-reusable contracts/runtime and test compatibility

### Recommended Phase 1 Scope
1. **Serialization Audit** (Research):
   - Find all current serialization code in codebase
   - Identify what gets serialized: Components? Entities? Game state?
   - Document current approaches (Unity serialization, custom, network packets)
   - Identify pain points and boundaries (save files, networking, versioning)

2. **Type System Design**:
   - Design stable type ID system for cross-boundary data
   - Define how to handle version migrations
   - Separate concerns: generic serialization vs persistence vs network

3. **Contracts & Interfaces**:
   - Define ISerializable contracts
   - Define schema/metadata interfaces
   - Create versioning strategy

4. **Proof of Concept**:
   - Implement basic serialization for 1-2 components
   - Write tests for serialization/deserialization
   - Test version compatibility

5. **Documentation**:
   - Document serialization architecture
   - Create usage examples
   - Define testing strategy

## Just merged (2026-10-12/13)

### EPIC 07 Complete
- **PR #58** (Phase 1): `0862793` — Modular Game Composition System
  - Module infrastructure with topological dependency sort
  - FrameworkCoreModule (zero game dependencies)
  - Game modules: GameLayerModule, InfrastructureModule, GameplayModule
  - Optional modules: NetworkingModule, LobbyModule, DebugToolsModule
  - 58/60 EditMode tests passed

- **PR #59** (Phase 2): `92c9abe` — CoreInstaller split and module migration
  - Split into FrameworkCoreInstaller + GameCoreInstaller
  - Resolved registration conflicts (ISystemRegistry, IEventBus)
  - Fixed LazyDependencyInjector duplication
  - Migrated 4 installers to module system

### EPIC 08 Complete (Performance & Lifecycle)
- **PR #60** (Phases 1 & 2): `8faddb9` — Async Cancellation and System Decoupling
  - EventBus: CancellationToken support for async operations
  - EventBus: Reflection caching with compiled delegates (17.5x speedup: ~700 ticks → ~40 ticks)
  - Performance benchmarks created

- **PR #61** (Phase 3): `cad8439` — Thread Safety, Duplicate Detection & Memory Leak Prevention
  - ComponentCache: ReaderWriterLockSlim for thread safety (+5% read overhead)
  - EventBus: Duplicate subscription detection with warnings
  - EventBus: Pending events limit (10K FIFO eviction) to prevent memory leaks
  - Comprehensive thread safety tests (7 tests)

- **PR #62** (Phase 4): `e8bd55a` — Final Optimization (DI Reflection & Init Rollback)
  - DIContainer: Reflection metadata caching with double-check locking (3-7x speedup: ~1,200 → ~400 ticks)
  - InitializationTransaction: LIFO rollback mechanism for safe initialization
  - 11 rollback tests, 6 performance benchmarks
  - Complete performance documentation: `EPIC-08-Performance-Benchmarks.md`

**Current HEAD: `ff9351d`**

## Next actions
1. **Start EPIC 06 Phase 1** — Serialization audit and design:
   - Search codebase for existing serialization code
   - Document what needs to be serialized (components, entities, state)
   - Identify boundaries (save files, network, versioning)
   - Design type ID system and versioning strategy
   - Create serialization contracts and POC implementation

2. **Or validate current `dev` state** (if prioritized):
   - Run full Unity EditMode tests at HEAD `ff9351d`
   - Full project compile verification
   - Player Build
   - Smoke tests (Movement, Visibility, Hero systems)

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key Technical Achievements
- **Modular Architecture**: Framework/Game separation with dependency resolution
- **Performance**: EventBus 17.5x faster, DI 3-7x faster
- **Thread Safety**: ComponentCache with minimal overhead
- **Memory Management**: Bounded pending events (10K limit)
- **Lifecycle Management**: Transaction-based initialization with rollback

## Performance Metrics Summary
- EventBus sync handlers: ~700 ticks → ~40 ticks (17.5x improvement)
- DI injection: ~1,200 ticks → ~400 ticks (3x improvement)
- ComponentCache thread-safe reads: +5% overhead only
- Memory: Pending events capped at 10,000 (stable, no leaks)

Full benchmarks documented in `Assets/_MythHunter/EPIC-08-Performance-Benchmarks.md`
