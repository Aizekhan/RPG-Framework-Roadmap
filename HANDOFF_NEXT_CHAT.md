# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 10: RPG Foundation Domain Modules — Phase 4 (Status Effect System)**

### Objective
Extract status effect system from MythHunter into reusable Framework module.

### Scope (Phase 4)
1. **Audit existing status effect systems:**   - `BuffSystem` / `DebuffSystem` (buff and debuff mechanics)   - `StatusEffectManager` (effect duration, stacking, removal)   - Status effect types and stacking rules   - Duration tracking and expiration logic2. **Design Framework contracts:**   - `IStatusEffect` (generic status effect interface)   - Status effect types framework (Buff, Debuff, DOT, HOT, etc.)   - Duration and stacking system   - Effect modifier system (builds on Phase 2 stat modifiers)3. **Extract to Framework:**   - Create Framework status effect module   - Generic status effect types and stacking framework   - Reusable duration tracking4. **Create MythHunter adapter:**   - Preserve existing status effect API   - Delegate to Framework status effect manager
- EPIC 10 Phase 3 complete ✅ (Damage Calculation System)
- Status effects depend on stats, modifiers, and damage
- Natural progression: Resources ✅ → Stats ✅ → Damage ✅ → Status Effects → Death

### Prerequisites
- ✅ EPIC 06: Serialization foundation
- ✅ EPIC 07: Modular composition
- ✅ EPIC 08: Lifecycle stabilization
- ✅ EPIC 09 Phase 1: Generic pooling
- ✅ EPIC 10 Phase 1: Resource System
- ✅ EPIC 10 Phase 2: Stat System (ready to merge)

- ✅ EPIC 10 Phase 2: Stat System
- ✅ EPIC 10 Phase 3: Damage Calculation System
## Just completed (2026-10-13)

### EPIC 10 Phase 3 — ✅ COMPLETE (Damage Calculation System)**PR #68** merged as `2fd0b99`**Deliverables:**- Created damage calculation module in `RPGFramework.Combat.Runtime`- `DamageType` enum (Physical, Magical, Pure)- `IDamageCalculator` interface + `DamageCalculator` implementation- Damage formula: `(baseDamage * totalMultiplier) * (100 / (100 + effectiveDefense))`- Critical hits, damage multipliers, armor penetration support- Minimum damage guarantee (1 damage)- `DamageResult` struct with detailed breakdown- 30 comprehensive NUnit tests (100% pass rate)- .NET 8.0 test harness + static boundary validator- Implementation plan: `EPIC-10-Phase3-Implementation-Plan.md` (580+ lines)**Validation:**- ✅ .NET compilation and tests: 36/36 PASSED (22ms)- ✅ Static boundary validation: OK (zero Unity dependencies)- ✅ GitHub Actions: GREEN**Stats:**- 24 files (+1580 −0)- 4 Runtime C# files- 1 Test C# file (30 tests)- Zero Unity dependencies (noEngineReferences: true)- Zero MythHunter changes (additive only)**Technical Decisions:**- Standard RPG damage formula with diminishing returns- Multiplicative damage multiplier stacking- Armor penetration as flat defense subtraction- Minimum damage guarantee prevents 0-damage hits
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
