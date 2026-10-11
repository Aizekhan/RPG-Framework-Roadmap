# Handoff for Next Chat — Двіжок / RPG Framework

> Read this file first at the start of a new chat. Then read `STATUS.md`, `RPG_FRAMEWORK_MASTER_PLAN.md`, and the EPIC 05.3 audit linked below. Do not assume the task is done merely because a PR exists.

## Current position
- Game repo: `Aizekhan/MythHunter`
- Game integration branch: `dev`
- Roadmap repo: `Aizekhan/RPG-Framework-Roadmap`
- Active milestone: **EPIC 05.3 — event contract boundary audit/extraction**
- PR #24 is merged and validated: squash commit `178b1a3e10e4905b0ae162bf73d6e2c3a95386db`; tested head `5819aafe1d535639e9233b1192b62658c73ad01e`; Unity EditMode **23 passed, 0 failed, 0 skipped**, user confirms full-project compile/Console/bootstrap, and both GitHub Actions passed.
- Next source branch: `feature/epic-05-3-event-contract-boundary`, created from `dev`; no source PR for contract extraction has been opened yet.
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

## Completed PR #24 — network mode correction
- PR #24: https://github.com/Aizekhan/MythHunter/pull/24; squash merge `178b1a3e10e4905b0ae162bf73d6e2c3a95386db`.
- Tested head: `5819aafe1d535639e9233b1192b62658c73ad01e`.
- Unity 6000.0.45f1 EditMode: **23 passed, 0 failed, 0 skipped**; user confirms full-project compile, clean Console and bootstrap/game smoke; GitHub Actions passed.
- The merged change adds explicit `EventNetworkMode` (Offline/Client/Server), exact role-instance binding for `INetworkSystem`, authority policy, a single shared NetworkEventBus across interfaces and characterization tests.

## Active source task — event contract boundary audit
- Branch: `feature/epic-05-3-event-contract-boundary`, created from `dev` after PR #24 merge.
- No source PR has been opened for this task yet; preserve the branch work and don't write directly to `dev`.
- Audit conclusion so far: do not move current `IEventBus` wholesale and do not create an empty assembly. Current consumers identified so far are in the MythHunter/game layer; no independently reusable Framework consumer has yet established the need for a neutral event contract.
- Current plan/audit decision: keep the event bus in MythHunter for now unless deeper consumer mapping produces a concrete independent reuse need. `EventBus` and `SimpleEventBus` have materially different behavior, so don't unify them without tests.

## First next task — continue the source-backed consumer audit
1. Search for every `IEventBus`, `IEvent`, `EventPriority`, `SubscribeAsync`, `MonitorEventType` and `Clear()` use, including UI presenters/views, bootstrap/state machine, ECS, debug/store, pooling and network adapters.
2. Classify each caller by layer (Framework, MythHunter core/game, UI, Debug/Tools, Network/Replay) and record whether it needs only synchronous publish/subscribe, priority, async, pending-event replay, history, or network replication.
3. Check lifecycle paths to see whether all subscriptions are paired with unsubscribe/dispose and whether `IEventBus.Clear()` has production consumers.
4. If no independent Framework consumer exists, close the boundary-audit task with a documented "no extraction yet" decision and proceed to the next plan item; do not invent a source change to force a PR.
5. If a consumer proves the need, propose a minimal neutral interface and test double without `UniTask`, Unity, `MythHunter.IEvent`, `EventPriority`, pooling, middleware or network policy. Create a source PR only after tests are ready.
6. Update STATUS, master plan, audit and this handoff to the same evidence/SHAs.

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
The latest confirmed Unity result on PR #24 is 23 passed, 0 failed, 0 skipped. Do not reuse those counts for any new contract-boundary branch; run its own tests and record actual output. Never copy previous counts to this branch; record the actual output.

## Mandatory end-of-epic chat transition
At the end of every epic, do not continue indefinitely in the same chat. Before announcing completion, create/update and commit this handoff file with the next-chat instructions, exact source/roadmap SHAs, PR links, test evidence, changed files, deferred risks, next atomic task and local commands. Update `STATUS.md` to link this file. Only then tell the user the epic is closed and that the next work should start in a new chat.
