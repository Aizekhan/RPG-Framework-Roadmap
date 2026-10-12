# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked MythHunter audit source. This handoff is an execution guide, not a second status database.

## Active roadmap item
**EPIC 05.9 bounded implementation: OnDemand positive-membership semantics — PR #54.**

- PR: https://github.com/Aizekhan/MythHunter/pull/54
- Branch: `fix/epic-05-9-ondemand-positive-membership`
- Exact head: `aaaaee9f864c66a746d9c64a0c7dcfaa6b8249db`
- Change: OnDemand `ComponentCache<T>.Contains` now queries authoritative `IEntityManager.HasComponent<T>` even for IDs not previously cached; refreshes cache on positive membership; removes stale cache state on absence.
- Added test: `ComponentCache_OnDemandContainsFindsUncachedAuthoritativeMembership`.
- CI on exact head passed: RPGFramework ECS Runtime [38150268459](https://github.com/Aizekhan/MythHunter/actions/runs/38150268459), Minimal CI [38150268442](https://github.com/Aizekhan/MythHunter/actions/runs/38150268442). Neither workflow executes Unity EditMode.
- Keep draft until focused Unity EditMode execution is confirmed; do not report pass from CI alone.

## Just merged
- PR #53 merge commit: `6bef9b210ee35b40e4c28e74e5550c3f1c58d762`. Persists movement/path/stamina struct changes from `MovementSystem.Update()` to the authoritative `IEntityManager`; avoids restoring stale values after `StopMovement`. CI passed on tested PR head `ab5b7c386163c8a1fcb7314840b8b223acd2868d`, but Unity validation was not run before the user's instruction to merge. This caveat must remain explicit.
- PR #50 merge commit: `0af6ab74f8236b5de7455349c795377c883b5295` — fixes editor-only GUI compile reference in `PreloadDebugTool`.
- PR #48 merge commit: `0d97060d45d978cafbe376cd43f00eb8fdf9f175` — ComponentCache consistency/reconciliation changes.

## Cloud session findings (PR #54 code review and out-of-Unity validation)
- **Code review (PR #54 HEAD `aaaaee9`):** New `Contains` branch mirrors OnDemand `Get` logic; `_entityIds` is `HashSet<int>` (no duplicates possible). Non-blocking observation: successful `Contains` sets `_isDirty = true` each time.
- **CI coverage gap:** No workflow compiles `ComponentCache.cs` or `DIContainerCharacterizationTests.cs` in Unity. The ECS workflow only compiles `Assets/_Framework` subset.
- **Out-of-Unity test (.NET 8, real `ComponentCache.cs` + `CacheStatistics.cs` + `_Framework/ECS/Runtime`, 4 `ComponentCache_*` tests verbatim, typed fixture instead of reflection):**
  - PR #54 `aaaaee9` → 4/4 passed.
  - `dev` `6bef9b2` → 3/4 passed; `OnDemandContainsFindsUncachedAuthoritativeMembership` failed (Expected True, was False).
  - This is NOT Unity EditMode and does NOT close validation gates.

## EPIC 01 gates remain open
- Exact current-`dev` Unity EditMode report; historical 44/44 belongs to local SHA `40b6296`.
- Full-project compile and clean Console; Player Build; Movement/Visibility/Hero smoke.
- Complete reverse-reference checks for cache registries/extensions, asmdef references and Unity scene/prefab serialized references.
- Do not mark EPIC 01 complete or start broad ECS extraction.

## Session 2026-10-11: PR #54 validation completed, process files added, JobHandle blocker found

### Process organization (PR #55 merged to dev as `55e554e`)
- Created `CLAUDE.md` — permanent rules (Framework principles, git/Unity guardrails, proof requirements, session workflow)
- Created `.claude/commands/start.md` — `/start` session startup checklist
- Created `.claude/commands/handoff.md` — `/handoff` milestone completion procedure
- **Purpose:** standardize session handoffs, reduce context repetition

### PR #54 validation on exact HEAD `aaaaee9f864c66a746d9c64a0c7dcfaa6b8249db`
**Unity EditMode:** ✅ **46/46 tests PASSED**
- New test `ComponentCache_OnDemandContainsFindsUncachedAuthoritativeMembership` — **PASSED**
- All assemblies passed: `MythHunter.DI.Characterization.Tests` (36/36), `RPGFramework.ECS.Runtime.Tests` (8/8), `RPGFramework.Logging.Runtime.Tests` (1/1), Addressables stub (1/1)
- Report: `D:\MythHunter-Git\PR54-EditMode-validation.xml` (51 KB)
- Duration: ~2 seconds after compilation

**Full Project Compile:** ✅ **Successful** (exit code 0, no CS errors)

**Player Build:** ❌ **Blocked by environment requirement**
- First blocker: `JobHandleExtensions.cs:22` — IL2CPP incompatibility (`handle != default`)
  - Fixed in PR #56 (`fix/jobhandle-il2cpp-compatibility`)
  - Changed to `!handle.Equals(default(JobHandle))`
- Second blocker: Missing Visual Studio 2022 with C++ compilers and Windows SDK (10.0.19041.0+)
  - **Environment requirement**, not code issue
  - Required for IL2CPP C++ code compilation

**Conclusion:** PR #54 ready to merge from EditMode + Compile perspective. Player Build gate requires environment setup (VS2022 C++ tools).

### Discovered issues
- **PR #56 created:** JobHandle IL2CPP fix (pre-existing blocker in dev, not from PR #54)
- **Timeout root cause:** New worktree "Rebuilding Library" takes >10 minutes (imports thousands of assets)
  - **Solution:** use existing worktree with prepared Library

### EPIC 01 gates status for PR #54
- ✅ Unity EditMode on exact PR HEAD `aaaaee9`
- ✅ Full project compile
- ⏸️ Player Build (environment requirement: VS2022 C++ tools)
- ⏸️ Smoke tests (depend on Player Build)
- ⏸️ Reverse-reference checks

## Next actions
1. **Install Visual Studio 2022** with C++ compilers and Windows SDK (10.0.19041.0+) for IL2CPP builds
2. **Merge PR #56** (JobHandle IL2CPP fix) — independent of PR #54, fixes pre-existing dev blocker
3. **Merge PR #54** — validation passed for EditMode and Compile gates
4. After environment setup: retry Player Build → Movement/Visibility/Hero smoke tests
5. Continue EPIC 05 one bounded cache contract at a time: OnDemand enumeration semantics, cache-local mutation vs authoritative writes

## Hard guardrails
- Exactly one ACTIVE roadmap item, in STATUS.md.
- Plan is canonical for scope/history; STATUS is the operational pointer.
- Never write directly to `dev`; future source changes through PRs.
- Never reset, clean, or discard local user files.
- GitHub CI, timeouts or results from another SHA are not Unity validation.
- EPIC 05.9 remains bounded; EPIC 01 validation remains open.
