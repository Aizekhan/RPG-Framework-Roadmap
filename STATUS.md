# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-14.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `14bdf4f` — Latest merged: [PR #75 EPIC 11 Phase 5](https://github.com/Aizekhan/MythHunter/pull/75).

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

### EPIC 10 (RPG Foundation Domain Modules) — ✅ **COMPLETE** 🎉
All 5 phases successfully extracted RPG combat domain into universal Framework modules.

- **PR #65** merged as `8b5dc22` — Phase 1: Resource System extraction
- **PR #66** merged as `b9652f8` — Test Infrastructure Fixes
- **PR #67** merged as `999405c` — Phase 2: Stat System extraction
- **PR #68** merged as `2fd0b99` — Phase 3: Damage Calculation System
- **PR #69** merged as `f26bad5` — Phase 4: Status Effect System
- **PR #70** merged as `1475b2c` — Phase 5: Death/Life State System

**EPIC 10 Summary:**
- **Completed:** All 5 phases (Resources → Stats → Damage → Status Effects → Death/Life State)
- **Total Files:** 112 files (+9274 insertions, −248 deletions)
- **Total Tests:** 124 comprehensive tests (100% pass rate)
- **Assemblies Created:** `RPGFramework.Combat.Runtime`, `RPGFramework.Stats.Runtime`
- **Framework Quality:** Zero Unity dependencies, zero MythHunter coupling, pure .NET 8.0 C#

### EPIC 11 (Optional Gameplay Modules) — ✅ **5/6 PHASES COMPLETE** 🚀
Universal RPG gameplay systems extracted as standalone Framework modules.

- **PR #71** merged as `58fa490` — Phase 1: Inventory System ✅
  - Created `RPGFramework.Inventory.Runtime` assembly
  - `IInventory<TSlotType>` interface with generic slot types
  - `Inventory<TSlotType>` implementation with capacity management
  - `InventorySlot` struct for item storage with stack limits
  - Events: `ItemAddedEvent`, `ItemRemovedEvent`, `ItemMovedEvent`, `InventoryFullEvent`
  - 24 comprehensive NUnit tests (100% pass rate)
  - .NET 8.0 test harness + static boundary validator
  - GitHub Actions workflow: `rpg-framework-inventory.yml`
  - Stats: 17 files (+1194 −0)
  - Zero Unity dependencies, zero MythHunter coupling

- **PR #72** merged as `9842621` — Phase 2: Item System ✅
  - Created `RPGFramework.Items.Runtime` assembly
  - Item contracts: `IItem`, `IItemInstance`, `IEquipment`, `IConsumable`, `IItemRegistry`
  - Item metadata: `ItemType`, `ItemRarity`, `ItemQuality` enums
  - Implementations: `Item`, `ItemInstance`, `Equipment`, `Consumable`, `ItemRegistry`
  - Component: `ItemComponent` for ECS integration
  - Events: `ItemCreatedEvent`, `ItemDestroyedEvent`, `ItemUsedEvent`
  - 27 comprehensive NUnit tests (100% pass rate)
  - .NET 8.0 test harness + static boundary validator
  - GitHub Actions workflow: `rpg-framework-items.yml`
  - Stats: 23 files (+1597 −0)
  - Zero Unity dependencies, zero MythHunter coupling

- **PR #73** merged as `3e717b7` — Phase 3: Loot System ✅
  - Created `RPGFramework.Loot.Runtime` assembly
  - Loot contracts: `ILootEntry`, `ILootTable`, `ILootGenerator`, `ILootTableRegistry`
  - Data structures: `LootDrop`, `LootResult`
  - Implementations: `LootEntry`, `LootTable`, `LootGenerator`, `LootTableRegistry`
  - Weighted random selection with configurable RNG seed
  - Stack size ranges and quantity randomization
  - Drop chance calculation (0-100%)
  - Events: `LootGeneratedEvent`, `LootDroppedEvent`
  - 23 comprehensive NUnit tests (100% pass rate)
  - .NET 8.0 test harness + static boundary validator
  - GitHub Actions workflow: `rpg-framework-loot.yml`
  - Stats: 21 files (+1452 −0)
  - Zero Unity dependencies, zero MythHunter coupling

- **PR #74** merged as `a0851a8` — Phase 4: Ability System ✅
  - Created `RPGFramework.Abilities.Runtime` assembly
  - Ability types: `AbilityType` enum (Active, Passive, Triggered, Toggle, Channeled)
  - Target types: `TargetType` enum (Self, Enemy, Ally, Area, Ground, Any)
  - Ability contracts: `IAbility`, `IAbilityInstance`, `IAbilityRegistry`, `ICooldownTracker`, `ITargetValidator`, `IAbilityExecutor`
  - Data structures: `ResourceCost`, `CastInfo`, `CooldownState`, `AbilityMetadata`, `AbilityExecutionResult`
  - Implementations: `Ability`, `AbilityInstance`, `AbilityRegistry`, `CooldownTracker`, `TargetValidator`, `AbilityExecutor`
  - Callback pattern using Func<>/Action<> for game layer integration
  - Cooldown management with time-based tracking
  - Charge system for limited-use abilities
  - Cast time and channeling support
  - Events: `AbilityExecutedEvent`, `AbilityCanceledEvent`, `CooldownStartedEvent`, `CooldownExpiredEvent`
  - 27 comprehensive NUnit tests (100% pass rate)
  - .NET 8.0 test harness + static boundary validator
  - GitHub Actions workflow: `rpg-framework-abilities.yml`
  - Stats: 27 files (+1795 −0)
  - Zero Unity dependencies, zero MythHunter coupling

- **PR #75** merged as `14bdf4f` — Phase 5: Progression System ✅
  - Created `RPGFramework.Progression.Runtime` assembly
  - XP curve types: `LevelCurveType` enum (Linear, Exponential, Polynomial, Custom)
  - Stat growth types: `GrowthCurveType` enum (Linear, Exponential, Polynomial)
  - Progression contracts: `IProgressionTracker`, `ILevelTable`, `IStatGrowthFormula`, `IUnlockManager`
  - Data structures: `ExperienceData`, `LevelInfo`, `StatGrowthData`, `UnlockData`, `ProgressionResult`
  - Implementations: `ProgressionTracker`, `LevelTable`, `StatGrowthFormula`, `UnlockManager`
  - Factory pattern for level curves (CreateLinear, CreateExponential, CreatePolynomial, CreateCustom)
  - Binary search optimization for level calculation (O(log n))
  - Precalculated caching for performance
  - Multi-level progression support (gain multiple levels at once)
  - Level-gated unlock system for abilities/features
  - Events: `ExperienceGainedEvent`, `LevelUpEvent`, `StatIncreasedEvent`, `UnlockAchievedEvent`
  - 27 comprehensive NUnit tests (100% pass rate)
  - .NET 8.0 test harness + static boundary validator
  - GitHub Actions workflow: `rpg-framework-progression.yml`
  - Stats: 26 files (+1581 −0)
  - Zero Unity dependencies, zero MythHunter coupling

**EPIC 11 Summary (Phases 1-5):**
- **Completed:** 5/6 phases (Inventory → Items → Loot → Abilities → Progression)
- **Total Files:** 114 files (+7619 insertions)
- **Total Tests:** 24 + 27 + 23 + 27 + 27 = **128 comprehensive tests** (100% pass rate)
- **Assemblies Created:** `RPGFramework.Inventory.Runtime`, `RPGFramework.Items.Runtime`, `RPGFramework.Loot.Runtime`, `RPGFramework.Abilities.Runtime`, `RPGFramework.Progression.Runtime`
- **Framework Quality:** Zero Unity dependencies, zero MythHunter coupling, pure .NET 8.0 C#
- **Remaining:** Phase 6 (Integration with MythHunter game layer)

### Earlier milestones
- PR #50 merged as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls guarded by `#if UNITY_EDITOR`.
- PR #48 merged as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- **Architecture Enforcement Phase 1**: PR #57 merged as `6637cca` — Foundation layer with architecture tests.
- **JobHandle IL2CPP fix**: PR #56 merged as `d36b271` — Fixes IL2CPP compilation error in `JobHandleExtensions.IsValid()`.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- PR #53 merged as `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`: MovementSystem authoritative write-through.

## Active Work

**EPIC 11 Phase 6: Integration** — ⏳ NEXT

### Scope
Create MythHunter adapters to integrate extracted Framework modules (EPIC 10 + EPIC 11 Phases 1-5) with game layer.

### Goals
- Connect Framework systems to MythHunter ECS entities
- Implement game-specific callbacks and event handlers
- Create Unity MonoBehaviour bridges where needed
- E2E validation of all integrated systems
- Performance testing and optimization

### Deliverables (Planned)
- MythHunter adapters for all 10 Framework modules
- Integration tests validating cross-system behavior
- Performance benchmarks
- Documentation of integration patterns

**Status:** Ready to start Phase 6 planning.

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): residual diff only folder .meta files.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind dev.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind dev.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4): long-open dev → main integration PR.
