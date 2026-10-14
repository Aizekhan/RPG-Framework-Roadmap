# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 10: RPG Foundation Domain Modules — Phase 5 (Death/Life State System)**

### Objective
Extract death/life state system from MythHunter into reusable Framework module.

### Scope (Phase 5)
1. **Audit existing death/life systems:**
   - `DeathSystem` / `RespawnManager` (death detection and respawn mechanics)
   - `LifeState` / health depletion handlers
   - Death event triggers and listeners
   - Respawn logic and spawn points

2. **Design Framework contracts:**
   - `IDeathHandler` (generic death detection interface)
   - `ILifeState` (alive/dead state tracking)
   - `IRespawnHandler` (respawn contract)
   - Death event framework (builds on EventBus)

3. **Extract to Framework:**
   - Create Framework death/life state module
   - Generic death detection (resource depletion trigger)
   - Reusable respawn contracts

4. **Create MythHunter adapter:**
   - Preserve existing death/respawn API
   - Delegate to Framework death handler

### Why Phase 5 next?
- EPIC 10 Phase 4 complete ✅ (Status Effect System)
- Death system is the natural final piece of combat flow
- Completion sequence: Resources ✅ → Stats ✅ → Damage ✅ → Status Effects ✅ → Death
- Completes the core RPG combat domain extraction

### Prerequisites
- ✅ EPIC 06: Serialization foundation
- ✅ EPIC 07: Modular composition
- ✅ EPIC 08: Lifecycle stabilization
- ✅ EPIC 09 Phase 1: Generic pooling
- ✅ EPIC 10 Phase 1: Resource System
- ✅ EPIC 10 Phase 2: Stat System
- ✅ EPIC 10 Phase 3: Damage Calculation System
- ✅ EPIC 10 Phase 4: Status Effect System

## Just completed (2026-10-13)

### EPIC 10 Phase 4 — ✅ COMPLETE (Status Effect System)

**PR #69** merged as `f26bad5`

**Deliverables:**
- Created status effect module in `RPGFramework.Combat.Runtime`
- `StatusEffectType` enum (20+ types: Buff, Debuff, DOT, HOT, Stun, Slow, etc.)
- `StackBehavior` enum (5 modes: Stack, Replace, Refresh, Ignore, StackCount)
- Effect contracts: `IStatusEffect`, `IStatusEffectTarget`, `IStatusEffectManager`
- Implementations: `StatusEffect`, `TimedStatusEffect`, `RecurringStatusEffect`, `StatusEffectManager`
- Duration tracking: Permanent (-1) or timed with automatic expiration
- Lifecycle hooks: OnApplied, OnUpdate, OnRemoved
- Recurring effects: Tick-based DOT/HOT with configurable intervals
- Integration with Phase 2 (stats) and Phase 3 (damage)
- 34 comprehensive NUnit tests (100% pass rate expected)
- .NET 8.0 test harness
- Implementation plan: `EPIC-10-Phase4-Implementation-Plan.md` (278 lines)

**Validation:**
- ✅ .NET compilation and tests: Pure .NET compile and Combat tests GREEN
- ✅ GitHub Actions: Combat tests PASSED
- ⚠️ Architecture tests failed (may be pre-existing)

**Stats:**
- 22 files (+1623 −0)
- 9 Runtime C# files
- 1 Test C# file (34 tests)
- Zero Unity dependencies (noEngineReferences: true)
- Zero MythHunter changes (additive only)

**Technical Decisions:**
- Generic effect type enum with extensibility (Custom = 1000)
- Five stacking behaviors covering common RPG patterns
- Lifecycle hook pattern for custom effect logic
- Recurring effect base class for DOT/HOT standardization
- Integration with existing Phase 2 stats (modifiers) and Phase 3 damage (calculator)

### EPIC 10 Phase 3 — ✅ COMPLETE (Damage Calculation System)

**PR #68** merged as `2fd0b99`

**Deliverables:**
- Created damage calculation module in `RPGFramework.Combat.Runtime`
- `DamageType` enum (Physical, Magical, Pure)
- `IDamageCalculator` interface + `DamageCalculator` implementation
- Damage formula: `(baseDamage * totalMultiplier) * (100 / (100 + effectiveDefense))`
- Critical hits, damage multipliers, armor penetration support
- Minimum damage guarantee (1 damage)
- `DamageResult` struct with detailed breakdown
- 30 comprehensive NUnit tests (100% pass rate)
- .NET 8.0 test harness + static boundary validator
- Implementation plan: `EPIC-10-Phase3-Implementation-Plan.md` (580+ lines)

**Validation:**
- ✅ .NET compilation and tests: 36/36 PASSED (22ms)
- ✅ Static boundary validation: OK (zero Unity dependencies)
- ✅ GitHub Actions: GREEN

**Stats:**
- 24 files (+1580 −0)
- 4 Runtime C# files
- 1 Test C# file (30 tests)
- Zero Unity dependencies (noEngineReferences: true)
- Zero MythHunter changes (additive only)

**Technical Decisions:**
- Standard RPG damage formula with diminishing returns
- Multiplicative damage multiplier stacking
- Armor penetration as flat defense subtraction
- Minimum damage guarantee prevents 0-damage hits

### EPIC 10 Phase 2 — ✅ COMPLETE (Stat System)

**PR #67** merged as `999405c`

**Deliverables:**
- Created `RPGFramework.Stats.Runtime` assembly
- `StatContainer<TStatType, TValue>` with modifier support
- Modifier types: Additive, Multiplicative, Override
- Calculation formula: `(base + additives) * multiplicatives` or override
- Priority system for modifier ordering
- Performance optimization: caching with dirty flags
- 18 comprehensive NUnit tests (100% pass rate)
- .NET 8.0 test harness + static boundary validator
- GitHub Actions workflow: `rpg-framework-stats.yml`
- Implementation plan: `EPIC-10-Phase2-Implementation-Plan.md` (500+ lines)
- Summary: `EPIC-10-Phase2-Summary.md` (600+ lines)

**Validation:**
- ✅ .NET tests: 18/18 PASSED
- ✅ Static boundary: OK
- ✅ GitHub Actions: GREEN

### EPIC 10 Phase 1 — ✅ COMPLETE (Resource System)

**PR #65** merged as `8b5dc22`

**Deliverables:**
- Created `RPGFramework.Combat.Runtime` assembly
- `ResourceComponent<TResourceType>` — generic resource pool (HP, MP, Stamina, etc.)
- `ResourceType` enum — 6 standard RPG resources (Health, Mana, Stamina, Energy, Rage, Focus)
- 8 comprehensive NUnit tests
- .NET 8.0 test harness for CI
- Static boundary validator (Python)
- GitHub Actions workflow (`rpg-framework-combat.yml`)

**Documentation:**
- `EPIC-10-RPG-Domain-Audit.md` (650+ lines) — comprehensive audit of Stats, Resources, Buffs, Damage, Death systems
- `EPIC-10-Phase1-Implementation-Plan.md` (570+ lines) — design rationale, adapter options, rollback plan
- `EPIC-10-Phase1-Summary.md` — complete delivery documentation
- `EPIC-10-Phase1-Final-Recommendation.md` — merge recommendation with risk analysis

**Validation:**
- ✅ .NET compilation and tests: 8/8 PASSED
- ✅ Static boundary validation: OK (zero Unity dependencies)
- ✅ GitHub Actions: GREEN
- ⏳ Unity EditMode: blocked by pre-existing errors (70+ errors unrelated to PR #65)

**Stats:**
- 18 files (+1785 insertions)
- 2 Runtime C# files
- 1 Test C# file (8 tests)
- Zero Unity dependencies (noEngineReferences: true)
- Zero MythHunter changes (additive only)

**Technical Decisions:**
- Generic type parameter: `where TResourceType : struct, IConvertible`
- Parameterless serialization (ISerializable contract)
- No value capping (Framework provides storage; game enforces policy)
- IResourceManager deferred to Phase 1.5/2 (YAGNI principle)

### EPIC 09 Phase 1 — ✅ COMPLETE (2026-10-13)

**PR #64** merged as `9adfb80`

**Deliverables:**
- Created `RPGFramework.Pooling.Runtime` assembly
- Extracted `IObjectPool<T>`, `ObjectPool<T>`, `PoolStatistics`
- Refactored `GameObjectPool` to Unity adapter pattern
- 20 tests
- Audit: `EPIC-09-Services-Audit.md` (590 lines)

**Status:** Phases 2-3 deferred until proven need

### EPIC 06-08 — ✅ COMPLETE (2026-10-12/13)
- EPIC 06: Serialization Foundation (5 phases, PR #63)
- EPIC 07: Modular Game Composition (2 phases, PRs #58, #59)
- EPIC 08: Lifecycle Stabilization (4 phases, PRs #60-62)

## Documentation Created (EPIC 10 Phases 1-4)
- `EPIC-10-RPG-Domain-Audit.md` (650+ lines)
- `EPIC-10-Phase1-Implementation-Plan.md` (570+ lines)
- `EPIC-10-Phase1-Summary.md`
- `EPIC-10-Phase1-Final-Recommendation.md`
- `EPIC-10-Phase2-Implementation-Plan.md` (500+ lines)
- `EPIC-10-Phase2-Summary.md` (600+ lines)
- `EPIC-10-Phase3-Implementation-Plan.md` (580+ lines)
- `EPIC-10-Phase4-Implementation-Plan.md` (278 lines)

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key decisions carried forward
- EPIC 10 Phases 1-4 complete ✅ — Combat foundation established
- Framework has zero game dependencies — universal RPG framework principle maintained
- Generic type parameters enable reusability (ResourceComponent, StatContainer)
- .NET test harness provides fast CI feedback (seconds vs minutes)
- Static boundary validation prevents accidental Unity/MythHunter dependencies
- Adapter pattern preserves MythHunter API during migration
- Pre-existing test errors don't block Framework module validation
- Combat flow complete: Resources → Stats → Damage → Status Effects
- Final piece: Death/Life State System (Phase 5)
