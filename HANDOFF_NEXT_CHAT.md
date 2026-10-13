# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 10: RPG Foundation Domain Modules — Phase 2 (Stat System)**

### Objective
Extract stat/attribute system from MythHunter into reusable Framework module with modifier calculations.

### Scope (Phase 2)
1. **Audit existing stat system:**
   - `StatType` enum (MythHunter game-specific stats)
   - `StatsComponent` (current/base/max values)
   - `StatRegistry` (global stat tracking)
   - Stat modifier patterns

2. **Design Framework contracts:**
   - `IStatContainer<TStatType, TValue>` (generic stat storage)
   - `IStatModifier` (additive/multiplicative modifiers)
   - Stat calculation formulas (base + modifiers)

3. **Extract to Framework:**
   - Create `RPGFramework.Stats.Runtime` assembly
   - Generic stat component (reusable across any RPG)
   - Modifier system (buffs/debuffs foundation)

4. **Create MythHunter adapter:**
   - Option A: Internal adapter (preserve existing API)
   - Option B: Direct migration (use Framework directly)

### Why now?
- EPIC 10 Phase 1 complete ✅ (Resource System)
- Resource extraction pattern validated
- Stat system depends on same EPIC 06 serialization foundation

### Prerequisites
- ✅ EPIC 06: Serialization foundation
- ✅ EPIC 07: Modular composition
- ✅ EPIC 08: Lifecycle stabilization
- ✅ EPIC 09 Phase 1: Generic pooling
- ✅ EPIC 10 Phase 1: Resource System

### Next immediate actions
1. **Search for stat systems:**
   - Grep: `StatType`, `StatsComponent`, `StatRegistry`
   - Check: `Assets/_MythHunter/Code/Stats/`, `Assets/_MythHunter/Code/Components/`

2. **Audit stat patterns:**
   - How are stats stored? (Current, Base, Max)
   - How are modifiers applied? (Additive, Multiplicative)
   - What's Framework-generic vs MythHunter-specific?

3. **Design Framework interface:**
   - `IStatContainer<TStatType, TValue>` — stat storage
   - `IStatModifier` — modifier application
   - Modifier calculation order (base + add → multiply)

4. **Create implementation plan:** `EPIC-10-Phase2-Implementation-Plan.md`

5. **Follow Phase 1 pattern:**
   - Assembly definition (noEngineReferences: true)
   - .NET test harness
   - Static boundary validator
   - GitHub Actions workflow

## Just completed (2026-10-13)

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

## Documentation Created (EPIC 10 Phase 1)
- `EPIC-10-RPG-Domain-Audit.md` (650+ lines)
- `EPIC-10-Phase1-Implementation-Plan.md` (570+ lines)
- `EPIC-10-Phase1-Summary.md`
- `EPIC-10-Phase1-Final-Recommendation.md`

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- Test results are valid only for the exact commit SHA tested.

## Key decisions carried forward
- EPIC 10 Phase 1 complete ✅ — Resource System validates extraction pattern
- Framework has zero game dependencies — universal RPG framework principle maintained
- Generic type parameters enable reusability (ResourceComponent<TResourceType>)
- .NET test harness provides fast CI feedback (8 sec vs 10+ min Unity)
- Static boundary validation prevents accidental Unity/MythHunter dependencies
- Adapter pattern preserves MythHunter API during migration
- Pre-existing test errors don't block Framework module validation
