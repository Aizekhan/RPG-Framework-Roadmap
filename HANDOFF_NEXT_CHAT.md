# Handoff for Next Chat — Двіжок / RPG Framework

> **Read this file first in the new chat**, then read `STATUS.md`, `RPG_FRAMEWORK_MASTER_PLAN.md` and `Audits/EPIC_05.4_System_Lifecycle_Audit.md`. Keep the work on the existing branch/PR; do not restart completed investigations.

## Completed item 5.3 — Event dispatch
- PR #23 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/23; merge `8e8ad7b16f813dcdebc4b2d057616f75f73f2fe8`. Fix: register final bus before resolving `ISystemRegistry`.
- PR #24 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/24; squash merge `178b1a3e10e4905b0ae162bf73d6e2c3a95386db`; tested head `5819aafe1d535639e9233b1192b62658c73ad01e`.
- PR #24 Unity 6000.0.45f1 EditMode: **23 passed, 0 failed, 0 skipped**; user confirmed full-project compilation, clean Unity Console and bootstrap/game smoke. Both GitHub Actions passed.
- Event-network configuration is explicit (Offline/Client/Server), with selected role-to-`INetworkSystem` binding and a single shared NetworkEventBus instance.
- Contract boundary decision: all found `IEventBus` consumers live in MythHunter/game code; no independent Framework consumer was found. The current `IEventBus` binds to MythHunter `IEvent`, `EventPriority`, `UniTask`, pooling/queue and game semantics. **Do not create a neutral event assembly or move `IEventBus` wholesale without new evidence.** Details are in `Audits/EPIC_05.3_Event_Dispatch_Candidate_Audit.md`.

## Active item 5.4 — System lifecycle and scheduling
- Source repo: `Aizekhan/MythHunter`; integration branch `dev`.
- Branch: `feature/epic-05-4-system-lifecycle-audit`.
- Draft PR #25: https://github.com/Aizekhan/MythHunter/pull/25
- Last checked head: `b9bd216d0425102dabc979cab8a0a0106709bb78`.
- Roadmap audit: `Audits/EPIC_05.4_System_Lifecycle_Audit.md`.
- GitHub Actions on that head: **Minimal CI passed** and **RPGFramework ECS Runtime passed** (framework boundary/GUID validation + .NET ECS tests).
- **Unity EditMode and user integration confirmation are pending for PR #25. Do not merge until both validation gates pass.** PR #24's 23/23 result is unrelated and must not be reused.

## Changes currently in PR #25
- `SystemRegistry.UpdateAll`, `FixedUpdateAll`, and `LateUpdateAll` run only successfully initialized active systems.
- `SystemRegistry.DisposeAll` disposes in reverse priority/registration order and clears its initialized-system tracking references.
- `SystemGroup` tracks successfully initialized children, doesn't update a child whose init failed, isolates individual init/update/dispose exceptions, disposes children in reverse order, and clears lifecycle tracking when a child is removed.
- Reflection-based Unity EditMode tests were added for update-before-init, reverse registry disposal order, and group callback failure isolation.
- These changes are not yet locally validated by Unity; the branch is a draft PR until verified.

## First action in new chat
1. Read this handoff, `STATUS.md`, the master plan and `Audits/EPIC_05.4_System_Lifecycle_Audit.md` before editing code.
2. Fetch PR #25 and check whether its current head is still `b9bd216d0425102dabc979cab8a0a0106709bb78`. Inspect the current diff; preserve all existing changes. Do not write directly to `dev`.
3. In `D:\\MythHunter-Git`, first run `git status --short --branch` and inspect any local modifications before switching branches. Do **not** reset or discard user changes; a prior local change to `Assets/_Framework/ECS/Runtime/EntityManager.cs.meta` was observed in an earlier pass. If any local change is still present, preserve it or use a separate worktree before switching to `feature/epic-05-4-system-lifecycle-audit`. Fetch the branch only after the worktree situation is safe.
4. Make sure the new lifecycle tests use the available `DispatchProxy` API in Unity 6000.0.45f1; if Unity compilation fails, replace only the test proxy mechanism without weakening the assertions.
5. Have the user run the following command in the local checkout on branch `feature/epic-05-4-system-lifecycle-audit`:
   ```powershell
   unity test "D:\\MythHunter-Git" --mode EditMode --output "D:\\MythHunter-Git\\EPIC-05-4-System-Lifecycle.xml" --verbose --timeout 600
   [xml]$r = Get-Content "D:\\MythHunter-Git\\EPIC-05-4-System-Lifecycle.xml"
   $r.SelectNodes("//test-run") | Select-Object result,total,passed,failed,skipped
   ```
4. Record actual XML totals/test names. Ask for confirmation of full-project compilation, clean Console and bootstrap/game smoke for this branch.
5. Fix failures, rerun tests on the exact new head, then merge PR #25 only after both gates pass.

## Separate questions after PR #25 validation
- `GameBootstrapper.Update()` calls `EcsWorld.Update` and the state machine; no production call to `SystemRegistry.FixedUpdateAll` or `LateUpdateAll` was found by source search. Confirm whether fixed/late scheduling is intentionally unused or missing.
- `IDILifecycleManager` is registered but no production call to `Shutdown()`, `ExecuteShutdownHandlers()`, or `DisposeAllScopes()` was found. Trace tracked-scope/shutdown consumers before wiring it into `GameBootstrapper.OnDestroy`.
- Characterize adding/removing systems after a group is initialized and registry reuse after `DisposeAll`; keep separate unless concrete consumers/tests justify them.

## Epic handoff rule
At the end of item 5.4, update and commit this report with exact merge SHA, test evidence, changed files, remaining decisions and the next atomic task. Ensure `STATUS.md` links to it before recommending a new chat.
