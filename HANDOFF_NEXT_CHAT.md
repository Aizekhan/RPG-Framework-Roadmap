# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 10: RPG Foundation Domain Modules — Phase 3 (Damage Calculation System)**

### Objective
Extract damage calculation system from MythHunter into reusable Framework module.

### Scope (Phase 3)
1. **Audit existing damage systems:**
   - `DamageCalculationService` (damage formulas)
   - `AttackSystem` / `DefenseSystem` (combat mechanics)
   - Damage type enums and resistance calculations
   - Critical hit, dodge, miss mechanics

2. **Design Framework contracts:**
   - `IDamageCalculator` (generic damage calculation interface)
   - Damage types framework (Physical, Magical, True damage, etc.)
   - Resistance calculation formulas
   - Damage modifier system (builds on Phase 2 stat modifiers)

3. **Extract to Framework:**
   - Create Framework damage calculation module
   - Generic damage types and resistance framework
   - Reusable damage formulas

4. **Create MythHunter adapter:**
   - Preserve existing combat API
   - Delegate to Framework damage calculator

### Why now?
- EPIC 10 Phase 2 complete ✅ (Stat System with modifiers)
- Damage calculations depend on stats and modifiers
- Natural progression: Resources → Stats → Damage → Status Effects → Death

### Prerequisites
- ✅ EPIC 06: Serialization foundation
- ✅ EPIC 07: Modular composition
- ✅ EPIC 08: Lifecycle stabilization
- ✅ EPIC 09 Phase 1: Generic pooling
- ✅ EPIC 10 Phase 1: Resource System
- ✅ EPIC 10 Phase 2: Stat System (ready to merge)

### Next immediate actions
1. **Merge PR #67** (Phase 2 Stat System) — CI passed, ready to merge
2. **Search for damage systems:**
   - Grep: `DamageCalculationService`, `AttackSystem`, `DefenseSystem`
   - Check: `Assets/_MythHunter/Code/Services/`, `Assets/_MythHunter/Code/Systems/`
3. **Audit damage patterns:**
   - How is damage calculated? (base damage + modifiers − defense)
   - What damage types exist? (Physical, Magical, etc.)
   - How do resistances work?
   - What's Framework-generic vs MythHunter-specific?
4. **Create implementation plan:** `EPIC-10-Phase3-Implementation-Plan.md`
5. **Follow Phase 1-2 pattern:**
   - Assembly definition (noEngineReferences: true)
   - .NET test harness
   - Static boundary validator
   - GitHub Actions workflow
## Just completed (2026-10-13)
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
- EPIC 10 Phase 2 complete ✅ — Stat System validates modifier pattern
- Framework has zero game dependencies — universal RPG framework principle maintained
- Generic type parameters enable reusability (ResourceComponent<TResourceType>)
- .NET test harness provides fast CI feedback (8 sec vs 10+ min Unity)
- Static boundary validation prevents accidental Unity/MythHunter dependencies
- Adapter pattern preserves MythHunter API during migration
- Pre-existing test errors don't block Framework module validation
- Modifier system foundation ready for Status Effect System (Phase 4)
