# EPIC 05.4 — System Lifecycle and Scheduling Audit

## Status
- Active implementation: draft PR #25, https://github.com/Aizekhan/MythHunter/pull/25
- Branch: `feature/epic-05-4-system-lifecycle-audit`
- Head at latest recorded review: `b9bd216d0425102dabc979cab8a0a0106709bb78`
- GitHub Actions on this head: Minimal CI passed; RPGFramework ECS Runtime passed (framework boundary/GUID validation and .NET ECS tests).
- **Pending:** Unity 6000.0.45f1 EditMode result for this exact branch and user confirmation of full-project compile, clean Console and bootstrap/game smoke. The existing event-dispatch 23/23 result does not validate this branch.

## Source files reviewed
- `Assets/_MythHunter/Code/Core/ECS/ISystem.cs`
- `Assets/_MythHunter/Code/Core/ECS/EcsWorld.cs`
- `Assets/_MythHunter/Code/Core/Game/GameBootstrapper.cs`
- `Assets/_MythHunter/Code/Systems/Core/ISystemRegistry.cs`
- `Assets/_MythHunter/Code/Systems/Core/SystemRegistry.cs`
- `Assets/_MythHunter/Code/Systems/Core/SystemInitializationCategory.cs`
- `Assets/_MythHunter/Code/Systems/Core/SystemCategoryAttribute.cs`
- `Assets/_MythHunter/Code/Core/ECS/ISystemFeatures.cs`
- `Assets/_MythHunter/Code/Systems/Groups/SystemGroup.cs`
- `Assets/_MythHunter/Code/Core/DI/IDILifecycleManager.cs`
- `Assets/_MythHunter/Code/Core/DI/DILifecycleManager.cs`
- State entry points: `Core/StateMachine/MainMenuState.cs`, `LobbyState.cs`, `GameplayState.cs`.

## Confirmed lifecycle model
- `ISystem` requires `Initialize()`, `Update(float)`, and `Dispose()`; optional feature interfaces expose fixed and late update.
- `EcsWorld.Initialize()` invokes `ISystemRegistry.InitializeAll()`; current `InitializeAll` targets `OnBoot`. `GameBootstrapper.InitializeGameSystems` also calls OnBoot category initialization. Main menu, Lobby and Gameplay state entry points initialize their own categories.
- `SystemRegistry` sorts registered systems by descending priority and tracks successful initialization in a `HashSet<ISystem>`.
- Groups compose child `ISystem` instances; group initialization/update/disposal must therefore be robust to one child's callback failing.

## Problems found and implemented in draft PR #25
1. Before this branch, `UpdateAll`, `FixedUpdateAll`, and `LateUpdateAll` iterated every active registration without checking whether its initialization had succeeded. Systems registered under Lobby/Gameplay/OnDemand could receive updates before `Initialize()`.
2. Before this branch, a child exception inside `SystemGroup.Initialize`, `Update`, or `Dispose` stopped processing remaining children. Also a child whose `Initialize` failed could later be updated by the group.
3. `SystemRegistry.DisposeAll` disposed in forward priority order and cleared registrations but retained disposed system references in `_initializedSystems`.

PR #25 changes:
- Registry update/fixed/late-update passes only invoke active systems successfully initialized.
- Registry disposes in reverse registered priority order and clears lifecycle tracking.
- SystemGroup tracks successfully initialized children, skips failed children for updates, catches callback exceptions individually, disposes children in reverse order and clears removed child references from tracking.
- Reflection-based EditMode tests exercise update-before-init, reverse registry disposal order, and per-child initialization/update/disposal failure isolation.

## Separate questions deliberately not included in PR #25
- `GameBootstrapper.Update()` calls `EcsWorld.Update` and state-machine `Update`, but no production caller for `SystemRegistry.FixedUpdateAll` or `LateUpdateAll` was found by source search. Confirm whether fixed/late scheduling is intentionally unsupported or missing.
- `IDILifecycleManager` is registered in `CoreInstaller`, but source search found no production call to `Shutdown()`, `ExecuteShutdownHandlers()`, or `DisposeAllScopes()`. Trace real tracked-scope and shutdown-handler consumers before wiring into `GameBootstrapper.OnDestroy`.
- Characterize group add/remove after initialization and registry reinitialization after `DisposeAll`; do not extend PR #25 unless these paths have concrete consumers and tests.
- Keep game phase/state policy outside any neutral framework lifecycle abstraction.

## Next actions
1. Inspect the current PR #25 patch and ensure its head is still `b9bd216d0425102dabc979cab8a0a0106709bb78`.
2. Run Unity 6000.0.45f1 EditMode in the local checkout on `feature/epic-05-4-system-lifecycle-audit`. Suggested command:
   ```powershell
   unity test "D:\\MythHunter-Git" --mode EditMode --output "D:\\MythHunter-Git\\EPIC-05-4-System-Lifecycle.xml" --verbose --timeout 600
   [xml]$r = Get-Content "D:\\MythHunter-Git\\EPIC-05-4-System-Lifecycle.xml"
   $r.SelectNodes("//test-run") | Select-Object result,total,passed,failed,skipped
   ```
3. Record actual counts/failures and relevant test names. Then require the user to confirm full-project compile, clean Unity Console and bootstrap/game smoke on this branch.
4. Fix any failing test or behavior and rerun Unity tests on the exact updated head. Do not merge using evidence from PR #24.
5. Merge PR #25 only after all gates pass. Record the exact merge SHA and test evidence in STATUS, this audit, the plan and handoff.
6. Then investigate Fixed/Late scheduling and DI shutdown lifecycle as separate evidence-backed tasks.

## Chat handoff requirements
At the end of item 5.4, update and commit `HANDOFF_NEXT_CHAT.md` with merged PR/SHAs, test evidence, unresolved lifecycle decisions, current branch/PR, changed files, the next atomic task and local validation commands. Link it from `STATUS.md` before switching to a new chat.
