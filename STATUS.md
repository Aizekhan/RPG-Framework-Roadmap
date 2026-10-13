# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-13.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `39af13b` — Latest: [PR #63 EPIC 06 Serialization Foundation](https://github.com/Aizekhan/MythHunter/pull/63).

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
  - **Phase 1:** Core contracts (ISerializable, ITypeRegistry, SerializableTypeAttribute, IMigrator)
  - **Phase 2:** VersionedSerializer with migration support
  - **Phase 3:** Component serialization (Framework-agnostic registry)
  - **Phase 4:** Delta compression module (optional)
  - **Phase 5:** Network serialization refactored (stable type IDs)
- Files: 26 changed (+2016 −281), 20 new Framework files, 24 tests
- Assemblies: RPGFramework.Serialization.Runtime, RPGFramework.DeltaCompression.Runtime
- Compiler fix: `dece04d` — Fixed EventBus test struct definitions

### EPIC 07 (Modular Game Composition System) — ✅ COMPLETE
- **PR #58** merged as `0862793` — Phase 1
- **PR #59** merged as `92c9abe` — Phase 2

### EPIC 08 (Dependency Debt and Lifecycle Stabilization) — ✅ COMPLETE
- **PR #60** merged as `8faddb9` — Phases 1 & 2
- **PR #61** merged as `cad8439` — Phase 3
- **PR #62** merged as `e8bd55a` — Phase 4

## Exactly one active roadmap item
**ACTIVE — EPIC 09: Optional Runtime Services and Adapters**

### Scope
Extract optional runtime services (resources, pooling, replay, cloud) with clear Framework/Game boundaries.

### Status
- [ ] Audit resource/preload/pool systems
- [ ] Define framework contracts vs Unity adapters
- [ ] Extract reusable patterns

### Next action
Begin EPIC 09 audit: map resource management and pooling systems.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation.
