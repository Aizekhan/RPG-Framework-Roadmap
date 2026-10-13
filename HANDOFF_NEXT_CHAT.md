# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 10: RPG Foundation Domain Modules — Audit Complete**

### Objective
Extract universal RPG domain modules that define core game mechanics without MythHunter-specific content.

### Audit Complete (2026-10-13)
**Document:** `EPIC-10-RPG-Domain-Audit.md` (MythHunter repository)

**Key Findings:**
1. **Framework-Ready (Extract As-Is):**
   - HealthComponent → ResourceComponent<TResourceType>
   - DamageAppliedEvent, HealingAppliedEvent, EntityDeathEvent
   - DamageType enum (Physical, Magical, Pure)

2. **Requires Refactoring (Split Framework/Game):**
   - StatType enum → CoreStatType (Framework) + MythHunterStatType (Game)
   - StatsComponent → IStatContainer<TStatType, TValue>
   - CombatStatsComponent → Basic stats (Framework) + Rage/Concentration (Game)

3. **Design New System (Missing Implementation):**
   - Generic modifier system (no add/remove/recalculate API)
   - Status effect system (no duration/stack tracking)
   - Damage calculator (no exposed formula engine)

### Phased Extraction Plan

**Phase 1: Resource System (High Priority — Nearly Complete)**
- Extract ResourceComponent<TResourceType> from HealthComponent
- Create RPGFramework.Combat.Runtime assembly
- MythHunter adapter: HealthComponent → ResourceComponent<Health>
- Tests: Resource creation, depletion, regeneration

**Phase 2: Stat System (Medium Priority — Requires Refactoring)**
- IStatContainer<TStatType, TValue> storage
- IStatModifier (additive/multiplicative)
- IStatModifierRegistry (add/remove/recalculate)
- Split StatType enum (Framework vs Game)

**Phase 3: Damage Calculation (Medium Priority — Design New)**
- Extract event structures (DamageAppliedEvent, etc.)
- IDamageCalculator interface
- MythHunter implementation with Rage/Concentration

**Phase 4: Status Effect System (High Priority — Design New)**
- StatusEffectComponent (Duration, StackCount)
- IStatusEffectDefinition, IStatusEffectRegistry
- StatusEffectSystem (apply/tick/expire)

**Phase 5: Life State Management (Deferred)**
- No resurrection system exists in MythHunter
- EntityDeathEvent already extracted in Phase 3

### Prerequisites
- ✅ EPIC 06: Serialization foundation (supports component serialization)
- ✅ EPIC 07: Modular composition
- ✅ EPIC 08: Lifecycle stabilization
- ✅ EPIC 09 Phase 1: Pooling (available if needed)

### Next immediate actions
1. **Create Phase 1 implementation plan:**
   - Read HealthComponent source (Assets/_MythHunter/Code/Components/Combat/HealthComponent.cs)
   - Design ResourceComponent<TResourceType> generic structure
   - Define acceptance criteria (Unity EditMode tests, game bootstrap smoke)

2. **Create feature branch:**
   - `feature/epic-10-phase-1-resource-system`
   - Based on current dev (9adfb80)

3. **Implementation steps:**
   - Create Assets/_Framework/Combat/Runtime/ directory
   - Add RPGFramework.Combat.Runtime.asmdef
   - Extract ResourceComponent<TResourceType> (CurrentValue, MaxValue, RegenRate)
   - Add ResourceComponentTests.cs (EditMode tests)
   - Update .NET test harness and CI paths

4. **Validation gates:**
   - Static boundary/GUID validation
   - .NET tests pass (resource lifecycle)
   - Unity EditMode tests pass
   - MythHunter game bootstrap smoke test clean
   - No regression in existing HealthComponent usage

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
- `EPIC-10-RPG-Domain-Audit.md` — Comprehensive audit of Stats, Resources, Buffs, Damage, Death systems
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
- EPIC 10 audit complete ✅ — HealthComponent ready for Framework extraction
- EPIC 09 Phase 1 complete ✅ — Generic pooling extracted to Framework
- EPIC 06 complete ✅ — Serialization foundation unblocks EPIC 10 component serialization
- Framework has zero game dependencies — universal RPG framework principle maintained
- Type identification uses stable IDs (`SerializableTypeAttribute`) — survives refactoring
- Optional modules pattern established — Pooling, DeltaCompression, Combat (upcoming)
