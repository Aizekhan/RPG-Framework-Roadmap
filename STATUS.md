# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-13.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `8b5dc22` — Latest: [PR #65 EPIC 10 Phase 1 - Resource System extraction](https://github.com/Aizekhan/MythHunter/pull/65).

## Verified completed sequence

### EPIC 05 (Framework Runtime Modules)
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- EPIC 05.9: PR #54 merged as `e8f724f` — ComponentCache OnDemand positive-membership semantics.

### EPIC 06 (Serialization Foundation) — ✅ COMPLETE
- **PR #63** merged as `39af13b` — All 5 phases complete:
  - **Phase 1:** Core contracts (`ISerializable`, `ITypeRegistry`, `SerializableTypeAttribute`, `IMigrator`)
  - **Phase 2:** VersionedSerializer with migration support
  - **Phase 3:** Component serialization (Framework-agnostic registry)
  - **Phase 4:** Delta compression module (optional)
  - **Phase 5:** Network serialization refactored (stable type IDs)
- Files: 26 changed (+2016 −281), 20 new Framework files, 24 tests
- Assemblies: `RPGFramework.Serialization.Runtime`, `RPGFramework.DeltaCompression.Runtime`
- Compiler fix: `dece04d` — Fixed EventBus test struct definitions

### EPIC 07 (Modular Game Composition System) — ✅ COMPLETE
- **PR #58** merged as `0862793` — Phase 1: Module infrastructure, FrameworkCoreModule, GameLayerModule, InfrastructureModule, GameplayModule, NetworkingModule, LobbyModule, DebugToolsModule. Unity EditMode 58/60 (96.7%).
- **PR #59** merged as `92c9abe` — Phase 2: CoreInstaller split into FrameworkCoreInstaller and GameCoreInstaller, module migration, LazyDI deduplication, InstallerRegistry removal.

### EPIC 08 (Dependency Debt and Lifecycle Stabilization) — ✅ COMPLETE
- **PR #60** merged as `8faddb9` — Phases 1 & 2
- **PR #61** merged as `cad8439` — Phase 3
- **PR #62** merged as `e8bd55a` — Phase 4

### EPIC 09 (Optional Runtime Services and Adapters) — ✅ COMPLETE (Phase 1)
- **PR #64** merged as `9adfb80` — Phase 1: Generic Pooling Foundation
  - Created `RPGFramework.Pooling.Runtime` assembly
  - Extracted `IObjectPool<T>`, `ObjectPool<T>`, `PoolStatistics` (20 tests)
  - Refactored `GameObjectPool` to Unity adapter pattern
  - Audit: `EPIC-09-Services-Audit.md` (590 lines)
  - Hotfix: Domain events interface compliance
  - Stats: 72 files (+1988 −61)
- **Status:** Phase 1 complete ✅, Phases 2-3 deferred until proven need

### EPIC 10 (RPG Foundation Domain Modules) — Phase 1 ✅ COMPLETE
- **PR #65** merged as `8b5dc22` — Phase 1: Resource System extraction
  - Created `RPGFramework.Combat.Runtime` assembly
  - Generic resource pool: `ResourceComponent<TResourceType>` (HP, MP, Stamina, etc.)
  - Standard RPG resources: `ResourceType` enum (6 types)
  - 8 comprehensive NUnit tests
  - .NET test harness + static boundary validator
  - Audit: `EPIC-10-RPG-Domain-Audit.md` (650+ lines)
  - Implementation plan: `EPIC-10-Phase1-Implementation-Plan.md` (570+ lines)
  - Stats: 18 files (+1785 insertions)
  - Validation: .NET tests ✅, Static boundary ✅, GitHub Actions GREEN ✅
- **Status:** Phase 1 complete ✅, Phase 2 (Stat System) ready to begin
### Earlier milestones
- PR #50 merged as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls guarded by `#if UNITY_EDITOR`.
- PR #48 merged as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- **Architecture Enforcement Phase 1**: PR #57 merged as `6637cca` — Foundation layer with architecture tests.
- **JobHandle IL2CPP fix**: PR #56 merged as `d36b271` — Fixes IL2CPP compilation error in `JobHandleExtensions.IsValid()`.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- PR #53 merged as `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`: MovementSystem authoritative write-through.

## Exactly one active roadmap item
**ACTIVE — EPIC 10: RPG Foundation Domain Modules — Phase 2**

### Scope
Extract RPG domain logic from MythHunter into Framework modules:
- **Phase 1:** Resource System (Health, Mana, Stamina pools) ✅ COMPLETE
- **Phase 2:** Stat System (Stats, Attributes, Modifiers) — ACTIVE
- **Phase 3:** Damage Calculation System (Damage types, resistances, calculations)
- **Phase 4:** Status Effect System (Buffs, debuffs, duration, stacking)
- **Phase 5:** Death/Life State System (Death detection, respawn contracts)

### Prerequisites
- ✅ EPIC 06 complete (Serialization foundation)
- ✅ EPIC 07 complete (Modular composition)
- ✅ EPIC 08 complete (Lifecycle stabilization)
- ✅ EPIC 09 Phase 1 complete (Generic pooling)
- ✅ EPIC 10 Phase 1 complete (Resource System)

### Phase 2 Status: Stat System
- [ ] Audit existing stat system (StatType, StatsComponent, StatRegistry)
- [ ] Design `IStatContainer<TStatType, TValue>` interface
- [ ] Design `IStatModifier` (additive/multiplicative)
- [ ] Extract stat calculation logic to Framework
- [ ] Create MythHunter adapter
- [ ] Tests and validation

### Next action
Begin EPIC 10 Phase 2: Stat System extraction (audit existing MythHunter stat implementation).


### Next action
Begin Phase 1 implementation: Extract generic resource system from HealthComponent.

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): residual diff only folder .meta files.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind dev.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind dev.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4): long-open dev → main integration PR.
