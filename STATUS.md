# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-13.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `ff9351d` — [Post EPIC 08 Phase 4 completion](https://github.com/Aizekhan/MythHunter/commit/ff9351d).

## Verified completed sequence
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- PR #50 merged to `dev` as `0af6ab74f8236b5de7455349c795377c883b5295`: `PreloadDebugTool.RenderGUI` GUI calls are guarded by `#if UNITY_EDITOR`, and `UnityEditor.EditorStyles.boldLabel` is fully qualified.
- PR #48 merged to `dev` as `0d97060d45d978cafbe376cd43f00eb8fdf9f175`: Incremental membership reconciliation, rotating refresh batches, OnDemand live reads and ComponentCache characterization tests.
- **EPIC 05.9**: PR #54 merged as `e8f724f` — ComponentCache OnDemand positive-membership semantics.
- **Architecture Enforcement Phase 1**: PR #57 merged as `6637cca` — Foundation layer with architecture tests.
- **JobHandle IL2CPP fix**: PR #56 merged as `d36b271` — Fixes IL2CPP compilation error in `JobHandleExtensions.IsValid()`.
- **EPIC 07 Phase 1**: PR #58 merged as `0862793` — Modular Game Composition System with plug-and-play module infrastructure.
- **EPIC 07 Phase 2**: PR #59 merged as `92c9abe` — CoreInstaller split into FrameworkCoreInstaller and GameCoreInstaller, module migration completed.
- **EPIC 08 Phases 1 & 2**: PR #60 merged as `8faddb9` — EventBus async cancellation support and reflection caching optimization (17.5x speedup).
- **EPIC 08 Phase 3**: PR #61 merged as `cad8439` — ComponentCache thread safety with ReaderWriterLockSlim, EventBus duplicate subscription detection, and pending events memory leak prevention (10K limit).
- **EPIC 08 Phase 4**: PR #62 merged as `e8bd55a` — DIContainer reflection optimization with metadata caching (3-7x speedup), InitializationTransaction rollback mechanism.

**Validation caveats:**
- PR #48's 44/44 Unity EditMode report is for local HEAD `40b6296`, not exact PR head `6631edf`. The GitHub workflows succeeded on the PR head but don't compile the full Unity project.
- Local investigation found no project-specific Unity Editor/test process. Do not kill long-lived Unity CLI server PID 18080 or delete caches based only on this evidence.
- PR #52 merged documentation reconciliation at `54a35204dab5ced9a825bb4e7946ab8efdb31ce3`.
- Current confirmed consumer map: `MovementSystem` owns three direct caches; `VisibilitySystem` owns two direct caches; `HeroSystem` obtains caches from `IComponentCacheRegistry`.

## Exactly one active roadmap item
**ACTIVE — EPIC 06 Phase 1: Component Serialization Foundation, scope to be defined.**

### Scope
EPIC 06 objectives from master plan:
- Audit component/entity/versioned/delta/network serialization separately
- Define stable schema/type identifiers when data crosses version/process boundaries
- Separate generic serialization from persistence policy and network wire formats
- Extract only proven-reusable contracts/runtime and test compatibility

### Phase 1 Next Steps
1. Audit current serialization approaches in codebase (components, entities, save data)
2. Identify serialization boundaries (persistence, networking, versioning)
3. Design stable type identifier system for cross-boundary data
4. Define serialization contracts and interfaces
5. Create proof-of-concept implementation with tests

### EPIC 01 exit gates remain open
- [ ] Exact-current-`dev` Unity EditMode with report and failure details.
- [ ] Full-project compile with clean Console.
- [ ] Player Build.
- [ ] Movement, Visibility and Hero smoke checks.
- [ ] Complete reverse-reference evidence for `IComponentCacheRegistry`, `GetCache<T>`, `FindEntities<T>`, `ForEach<T>`, cache extension methods, asmdef references, and Unity scene/prefab serialized references.

## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1)
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3)
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5)
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4)
- PR #2 and #49 were closed as duplicates.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation. Test results are valid only for the exact commit SHA tested.
