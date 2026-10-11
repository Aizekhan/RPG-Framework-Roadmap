# Handoff for Next Chat — Двіжок / RPG Framework

> Read this file first at the start of a new chat. Then read `STATUS.md`, `RPG_FRAMEWORK_MASTER_PLAN.md`, and the EPIC 05.3 audit linked below. Do not assume the task is done merely because a PR exists.

## Current position
- Game repo: `Aizekhan/MythHunter`
- Game integration branch: `dev`
- Roadmap repo: `Aizekhan/RPG-Framework-Roadmap`
- Active milestone: **EPIC 05.3 — Event dispatch**
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
- Implementation branch was created from current `dev`: `feature/epic-05-3-network-eventbus-selection`.
- At handoff creation no source PR for this network-selection task has been opened yet. Check the branch and open PR metadata first; preserve any work already on the branch.
- Do not write directly to `dev`. Use a narrow PR with a rollback path.

## First next task
Design an explicit role/capability contract for event network synchronization:
1. Distinguish whether networking/event replication is enabled from the actual runtime role: Offline, Client, Server, or Combined Host.
2. Decide who owns role selection, based on real application configuration/runtime callers, not assumptions.
3. Ensure `NetworkEventBus` can enforce `NetworkEventAuthority.Server/Client/Both` accurately for all supported roles; a shared interface should not be mistaken for a role interface.
4. Preserve behavior for offline games. Do not make network dependencies mandatory in Framework Core.
5. Add focused installer/DI tests for bus selection, shared instance between `IEventBus` and `ISystemRegistry`, and authority behavior across roles.
6. Run Unity EditMode; record the exact XML counts and relevant test names; ask the user to confirm full-project compile/Console/bootstrap before merge.
7. Update this handoff and the roadmap to the new exact PR/commit/test state.

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
The latest confirmed result is only PR #23's 23/23. Never copy those counts to a future branch; record the actual output.

## Mandatory end-of-epic chat transition
At the end of every epic, do not continue indefinitely in the same chat. Before announcing completion, create/update and commit this handoff file with the next-chat instructions, exact source/roadmap SHAs, PR links, test evidence, changed files, deferred risks, next atomic task and local commands. Update `STATUS.md` to link this file. Only then tell the user the epic is closed and that the next work should start in a new chat.
