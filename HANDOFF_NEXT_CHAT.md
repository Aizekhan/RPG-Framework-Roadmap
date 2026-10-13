# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 06: Serialization Foundation — AUDIT PHASE**

### Objective
Establish a reusable serialization foundation before EPIC 12 (Persistence) and EPIC 13 (Networking).

### Scope
1. **Audit current serialization** — Search codebase for:
   - Component/entity serialization
   - Versioned data structures
   - Delta/snapshot serialization
   - Network protocol serialization
   - Save/load mechanisms

2. **Define stable identifiers** — Type IDs, schema versions, compatibility contracts

3. **Separate concerns**:
   - Generic serialization contracts (Framework)
   - Persistence policies (Game Layer)
   - Network wire formats (Networking Module)

4. **Extract reusable contracts** — Only proven-reusable with compatibility tests

### Why now?
EPIC 07 (Game Layer Composition) and EPIC 08 (Lifecycle Stabilization) are complete. Serialization foundation is needed before:
- EPIC 09 (Save/Load, Resources)
- EPIC 12 (Persistence)
- EPIC 13 (Networking)

### Prerequisites ✅
- ✅ EPIC 07 complete: Modular composition system established
- ✅ EPIC 08 complete: Lifecycle, threading, DI optimized
- ✅ Clean separation between Framework and Game Layer

### Next immediate actions
1. **Search for serialization code**:
   - Grep for: `Serialize`, `Deserialize`, `ISerializ`, `JsonConvert`, `BinaryFormatter`
   - Check: Components, Entities, Save/Load, Network, Resources

2. **Map serialization patterns**:
   - What gets serialized? (entities, components, game state)
   - Where? (save files, network packets, resources)
   - How? (JSON, binary, custom formats)
   - Versioning strategy?

3. **Identify Framework vs Game**:
   - Generic serialization contracts → Framework
   - Game-specific data → Game Layer
   - Network protocol → Networking Module

4. **Create audit document**: `EPIC-06-Serialization-Audit.md`

## Just completed (2026-10-13)

### EPIC 07 Phase 2 — ✅ MERGED
- **PR #59** merged as `92c9abe`
- Split CoreInstaller → FrameworkCoreInstaller + GameCoreInstaller
- Migrated 4 installers to modules
- Removed InstallerRegistry dead code
- Fixed LazyDI duplication
- Resolved hidden registration conflicts

### EPIC 08 — ✅ COMPLETE (All 4 Phases)

**Phase 1-2: Async Cancellation & System Decoupling** — PR #60 (`8faddb9`)
- AsyncTaskTracker for proper cancellation token lifecycle
- Decoupled systems: ArchetypeSystem, HeroSelectionService, TimerService
- EventBus async cancellation support
- State machine async refactoring (all 7 states)
- 37 files, 3,398+ insertions

**Phase 3: Thread Safety & Memory Leaks** — PR #61 (`cad8439`)
- ComponentCache thread safety (lock-free reads, write locks)
- EventBus duplicate subscription prevention
- Memory leak fixes (weak references, cleanup)
- 5 files, 1,265+ insertions
- Comprehensive thread safety and leak tests

**Phase 4: Performance Optimization** — PR #62 (`e8bd55a`)
- DI reflection optimization: 85-95% performance improvement
- Type metadata caching
- InitializationTransaction for deterministic rollback
- Performance benchmarks and characterization tests
- 6 files, 1,090+ insertions

### EPIC 07 Phase 1 — ✅ MERGED (2026-10-12)
- **PR #58** merged as `0862793`
- Module infrastructure with topological dependency resolution
- FrameworkCoreModule (zero game dependencies)
- Game modules: GameLayerModule, InfrastructureModule, GameplayModule
- Optional modules: NetworkingModule, LobbyModule, DebugToolsModule
- 18 files, 787+ insertions

## Documentation created (2026-10-13)
- `EPIC-01-Reverse-Reference-Audit.md` — Component cache usage patterns
- `EPIC-06-Serialization-Audit.md` — Serialization patterns and boundaries
- `EPIC-08-Dependency-Lifecycle-Audit.md` — Async, threading, DI issues
- `EPIC-06-08-Implementation-Plan.md` — Complete implementation strategy
- `SESSION-2026-10-13-Summary.md` — Full session summary
- `EPIC-08-Performance-Benchmarks.md` — DI performance analysis
- `SMOKE-TESTS.md` — Smoke test procedures
- `ENVIRONMENT-SETUP.md` — Development environment setup

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key decisions carried forward
- EPIC 07 prioritized before EPIC 06 to establish Framework/Game boundaries ✅
- EPIC 08 completed before EPIC 06 to stabilize foundation ✅
- Now EPIC 06 can proceed with clean module boundaries and stable lifecycle
- Framework Core has zero game dependencies — universal RPG framework principle maintained
