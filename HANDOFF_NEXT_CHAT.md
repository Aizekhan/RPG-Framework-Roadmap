# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 09: Optional Runtime Services and Adapters — Phase 1 Complete ✅**

### Current State
- ✅ Phase 1: Generic Pooling Foundation (PR #64, commit `9adfb80`)
- ⏸️ Phase 2: Resource Loading Contracts (deferred until multi-engine need)
- ⏸️ Phase 3: Replay/Cloud Interfaces (deferred until implementations exist)

### What was delivered (Phase 1)
**Created Framework Pooling:**
- `RPGFramework.Pooling.Runtime` assembly (no Unity dependencies)
- `IObjectPool<T>` interface with lifecycle callbacks
- `ObjectPool<T>` implementation (pre-warming, statistics, max size)
- 20 comprehensive tests in `ObjectPoolTests.cs`

**Refactored Game Layer:**
- `GameObjectPool` → Unity adapter pattern using Framework pool
- Logger adapter bridges `IMythLogger` → Framework `ILogger`
- Maintained existing public API, enhanced with Framework pool internals

**Documentation:**
- `EPIC-09-Services-Audit.md` (590 lines) — comprehensive analysis of 4 service subsystems

**Stats:** 72 files (+1988 −61)

### Why Phases 2-3 deferred?
- **Phase 2 (Resources):** Current `IResourceManager` is Unity-only. No multi-engine need proven yet. Extraction would create unused Framework contracts.
- **Phase 3 (Replay/Cloud):** `IReplaySystem` and `IDataService` are interfaces with no implementations. Cannot extract patterns from non-existent code.

### Next action
**Ready for next EPIC assignment.** EPIC 09 extraction complete per audit recommendations.

## Just completed (2026-10-13)

### EPIC 09 Phase 1 — ✅ COMPLETE

**PR #64** merged as `9adfb80`

**Deliverables:**
- `RPGFramework.Pooling.Runtime` assembly with generic pooling foundation
- `IObjectPool<T>`, `ObjectPool<T>`, `PoolStatistics` (20 tests)
- `GameObjectPool` refactored to Unity adapter pattern
- `EPIC-09-Services-Audit.md` (590 lines)
- Hotfix: Domain events interface compliance (IEvent.GetEventId, GetPriority)

**Stats:** 72 files (+1988 −61)

**Phases 2-3 deferred** until multi-engine need proven (resources) and implementations exist (replay/cloud).

### EPIC 06 — ✅ COMPLETE (All 5 Phases)

**PR #63** merged as `39af13b`

**Phase 1: Core Contracts**
- Created `RPGFramework.Serialization.Runtime` assembly
- Contracts: `ISerializable`, `ISerializer<T>`, `IVersionedData`, `ITypeRegistry`, `IMigrator`
- `SerializableTypeAttribute` for stable type IDs (replaces `Type.FullName`)
- `TypeRegistry` implementation with 16 tests

**Phase 2: Versioned Serialization**
- Refactored `VersionedSerializer` to use `ITypeRegistry`
- Implemented `IMigrationRegistry` + `MigrationRegistry`
- Version mismatch detection and rejection
- Migration tests (v1 → v2 transformation, 8 tests)

**Phase 3: Component Serialization**
- Extracted `IComponentSerializer<T>` to Framework
- Extracted `ComponentSerializerBase<T>` to Framework
- Made `ComponentSerializerRegistry` game-agnostic (no hardcoded components)
- Serializers registered via `SerializationInstaller`

**Phase 4: Delta Compression**
- Created `RPGFramework.DeltaCompression.Runtime` optional module
- `IDeltaSerializer` + `DeltaSerializer` (thread-safe, configurable threshold)
- Memory cleanup (`ClearState`, `ClearAllStates`)

**Phase 5: Network Serialization**
- Refactored `BinaryNetworkSerializer` to use `ITypeRegistry`
- Added `SerializableTypeAttribute` to `NetworkEventMessage`
- `NetworkingInstaller` registers message types
- Wire format: [TypeId][Version][Length][Payload]

**Stats:**
- 26 files changed (+2016 −281)
- 20 new Framework files (17 runtime, 3 tests)
- 2 new assemblies (Serialization, DeltaCompression)
- 24 tests (TypeRegistry, VersionedSerializer)

**Compiler Fix:**
- `dece04d` — Fixed EventBus test struct definitions (EPIC 08 residual)

### EPIC 07 — ✅ COMPLETE (2026-10-12)
- PR #58: Phase 1 — Module infrastructure (`0862793`)
- PR #59: Phase 2 — CoreInstaller split (`92c9abe`)

### EPIC 08 — ✅ COMPLETE (2026-10-12/13)
- PR #60: Phases 1-2 — Async cancellation (`8faddb9`)
- PR #61: Phase 3 — Thread safety (`cad8439`)
- PR #62: Phase 4 — DI optimization (`e8bd55a`)

## Documentation Created
- `EPIC-09-Services-Audit.md` — 590 lines, services subsystems analysis
- `EPIC-06-Serialization-Audit.md` — 590 lines, comprehensive analysis
- Session summaries for 2026-10-12, 2026-10-13

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key decisions carried forward
- EPIC 09 Phase 1 complete ✅ — Generic pooling extracted to Framework
- EPIC 06 complete ✅ — Serialization foundation unblocks EPIC 09, 12, 13
- Framework has zero game dependencies — universal RPG framework principle maintained
- Type identification uses stable IDs (`SerializableTypeAttribute`) — survives refactoring
- Delta compression is optional module — not mandatory Framework dependency
- Pooling is optional module — Framework provides generic pool, Unity provides adapter
