# Handoff for Next Chat — Двіжок / RPG Framework

> Read this file first at the start of a new chat. Then read `STATUS.md`, `RPG_FRAMEWORK_MASTER_PLAN.md`, and the EPIC 05.3 audit linked below. Do not assume the task is done merely because a PR exists.

## Current position
- Game repo: `Aizekhan/MythHunter`
- Game integration branch: `dev`
- Roadmap repo: `Aizekhan/RPG-Framework-Roadmap`
- Active milestone: **EPIC 05.3 — explicit network event-bus selection and role-aware authorization**
- Completed source fix: [PR #23](https://github.com/Aizekhan/MythHunter/pull/23), merge commit `8e8ad7b16f813dcdebc4b2d057616f75f73f2fe8`.
- User has confirmed PR #23 full-project compilation, Unity Console and bootstrap are clean.
- Unity 6000.0.45f1 EditMode on PR #23: **23 passed, 0 failed, 0 skipped**. GitHub Actions passed.

## What PR #23 fixed
`EventsInstaller` now selects/registers the final event bus before resolving `ISystemRegistry`. Previously `SystemRegistry` could resolve and cache the early `SimpleEventBus` singleton before `EventsInstaller` registered `EventBus` or `NetworkEventBus`.

## Confirmed network wiring issue
- `NetworkingInstaller` registers `IClientNetworkSystem -> ClientNetworkSystem` and `IServerNetworkSystem -> ServerNetworkSystem`, but does not register the exact `INetworkSystem` key.
- `DIContainer.IsRegistered(Type)` checks exact registered keys. The current `EventsInstaller` check of `IsRegistered<INetworkSystem>()` therefore selects ordinary `EventBus` in the current installer setup.
- A separate `NetworkSystem : INetworkSystem` composes `INetworkClient` and `INetworkServer`, but `NetworkEventBus` decides `Client`/ `Server` authority by checking whether the resolved object implements `IClientNetworkSystem` or `IServerNetworkSystem`. `NetworkSystem` implements neither. Registering it alone does not solve event-authority semantics.
- `NetworkAuthority` currently depends directly on both `IClientNetworkSystem` and `IServerNetworkSystem`.
- Source search did not expose clear production callers for connect/start or role activation methods; inspect scene objects, serialized config, UI/menu and runtime startup paths before finalizing design.
- Do **not** alias either client or server singleton arbitrarily to `INetworkSystem`, and do not silently change `NetworkAuthority`.

## Active branch / PR
- Implementation branch: `feature/epic-05-3-network-eventbus-selection`, created from `dev`.
- Draft source PR #24: https://github.com/Aizekhan/MythHunter/pull/24
- Last implementation head at latest audit: `5819aafe1d535639e9233b1192b62658c73ad01e`.
- Source changes include `EventNetworkMode`, explicit composition-root mode selection in `GameBootstrapper`, role-to-`INetworkSystem` aliasing in `NetworkingInstaller`, mode-driven bus selection and singleton alias in `EventsInstaller`, authority policy in `NetworkEventBus`, and reflection-based EditMode characterization tests.
- GitHub Actions on earlier head `9ced67674576127d98b3cd328b9b85917faacb18` passed. Latest implementation head is `5819aafe1d535639e9233b1192b62658c73ad01e`; latest CI boundary/GUID validation and .NET ECS test steps succeeded, with workflow cleanup still completing when last checked. **Unity EditMode and branch-specific full-project/bootstrap confirmation are not yet done.** Review actual current head before running tests; don't assume commits are compile-clean.
- Do not write directly to `dev`. Use a narrow PR with a rollback path.

## First next task
Finish validation and correct any failures for the in-progress explicit network-mode design:
1. Review current source diffs on `feature/epic-05-3-network-eventbus-selection`. Current intended modes are **Offline, Client, Server**; combined host is deliberately unsupported because the current role systems forbid simultaneous client/server operation.
2. Ensure `GameBootstrapper` registers the serialized mode before `InstallerRegistry`; Offline remains default for backward-compatible behavior.
3. Verify `NetworkingInstaller` binds the selected role implementation instance under the exact `INetworkSystem` key, and does not remove `NetworkAuthority` registration.
4. Verify `EventsInstaller` binds `IEventBus` and `INetworkEventBus` to the exact same `NetworkEventBus` singleton (not two separate singleton registrations).
5. Verify `NetworkEventBus` uses the configured mode for `Server/Client/Both` authorization and unsubscribes its network callback on Dispose.
6. Run Unity EditMode; record actual counts and failures; user confirms full project compile/Console/bootstrap before merge.
7. PR #24 is already open as a draft; update its description and roadmap/audit/handoff with the exact validated head, and merge only after all validation gates pass.

## Files to inspect first
- `Assets/_MythHunter/Code/Core/Installers/InstallerRegistry.cs`
- `Assets/_MythHunter/Code/Core/Installers/CoreServices/NetworkingInstaller.cs`
- `Assets/_MythHunter/Code/Core/Installers/CoreServices/EventsInstaller.cs`
- `Assets/_MythHunter/Code/Networking/Core/INetworkSystem.cs`
- `Assets/_MythHunter/Code/Networking/Core/NetworkSystem.cs`
- `Assets/_MythHunter/Code/Networking/Core/ClientNetworkSystem.cs`
- `Assets/_MythHunter/Code/Networking/Core/ServerNetworkSystem.cs`
- `Assets/_MythHunter/Code/Networking/Core/NetworkAuthority.cs`
- `Assets/_MythHunter/Code/Events/NetworkEvents/NetworkEventBus.cs`
- `Assets/_MythHunter/Code/Core/Game/GameBootstrapper.cs`
- `Assets/_MythHunter/Tests/DI/DIContainerCharacterizationTests.cs`
- Roadmap: `Audits/EPIC_05.3_Event_Dispatch_Candidate_Audit.md`, `RPG_FRAMEWORK_MASTER_PLAN.md`, `STATUS.md`.

## Test command / result extraction
Use the Unity CLI convention already used locally and change the output path for this run:
```powershell
unity test "D:\\MythHunter-Git" --mode EditMode --output "D:\\MythHunter-Git\\EPIC-05-3-Network-Selection.xml" --verbose --timeout 600
[xml]$r = Get-Content "D:\\MythHunter-Git\\EPIC-05-3-Network-Selection.xml"
$r.SelectNodes("//test-run") | Select-Object result,total,passed,failed,skipped
```
The latest confirmed Unity result is PR #23's 23/23 only. PR #24's new branch has not yet had Unity tests confirmed. Never copy previous counts to this branch; record the actual output.

## Mandatory end-of-epic chat transition
At the end of every epic, do not continue indefinitely in the same chat. Before announcing completion, create/update and commit this handoff file with the next-chat instructions, exact source/roadmap SHAs, PR links, test evidence, changed files, deferred risks, next atomic task and local commands. Update `STATUS.md` to link this file. Only then tell the user the epic is closed and that the next work should start in a new chat.
