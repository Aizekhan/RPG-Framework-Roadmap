# EPIC 05.3 — Event Dispatch Candidate Audit

## Status
Source-backed audit and the event network-mode correction are complete. PR #23 merged as `8e8ad7b16f813dcdebc4b2d057616f75f73f2fe8`. PR #24 https://github.com/Aizekhan/MythHunter/pull/24 merged via squash as `178b1a3e10e4905b0ae162bf73d6e2c3a95386db`; Unity EditMode 23 passed, 0 failed, 0 skipped; user confirmed full-project compile/Console/bootstrap and GitHub Actions passed. Current next task is an evidence-driven boundary audit on branch `feature/epic-05-3-event-contract-boundary`; do not move `IEventBus` wholesale. Decide whether any neutral publisher/subscriber contract is warranted only after consumer/semantics mapping.

## Source baseline
Reviewed `Aizekhan/MythHunter` on `dev` after EPIC 05.2 merge `9c6f03675a85085fe5f713dfd9a9be871b5c1e59`.

Key sources:
- `Assets/_MythHunter/Code/Events/IEvent.cs`
- `Assets/_MythHunter/Code/Events/IEventBus.cs`
- `Assets/_MythHunter/Code/Events/EventBus.cs`
- `Assets/_MythHunter/Code/Events/SimpleEventBus.cs`
- `Assets/_MythHunter/Code/Events/NetworkEvents/NetworkEventBus.cs`
- `Assets/_MythHunter/Code/Events/IEventPool.cs`
- `Assets/_MythHunter/Code/Events/EventPool.cs`
- `Assets/_MythHunter/Code/Core/Installers/CoreServices/CoreInstaller.cs`
- callers/wrappers: `EventHandlerBase.cs`, `SystemBase.cs`, `EventBatcher.cs`, `Extensions/SystemEventExtensions.cs`, `Debug/Events/EventDebugTool.cs`.

## Observed facts

1. `IEventBus` is not a neutral framework contract as currently shaped:
   - generic events must be structs;
   - sync and async APIs require MythHunter `IEvent` and `EventPriority`;
   - async signatures expose Cysharp `UniTask`;
   - its API includes `Clear()`, but no subscription token/disposable handle.

2. `IEvent` requires `GetEventId()` and `GetPriority()`. `EventPriority` is defined in the MythHunter.Events namespace. This embeds domain/logging/queue policy into the contract.

3. There are multiple implementations with material semantic differences:
   - `EventBus`: priority ordering, sync event queues, immediate `Critical` dispatch, monitored/pending event cache, event pooling, debug middleware, and an asynchronous processing loop.
   - `SimpleEventBus`: straightforward handler lists, ignores priority argument, dispatches synchronously in registration order, runs async handlers via `UniTask.WhenAll`, no queue/pending-event logic, and its current `Clear()` clears handlers but does not dispose/cancel processing or return events to the pool.
   - `NetworkEventBus` derives from `EventBus` and overrides publish to add network synchronization; it is an adapter/game capability, not part of a neutral event contract.
   - `CoreInstaller` currently binds `IEventBus` to `SimpleEventBus`. This is the observed composition-root binding, not an assumption that `EventBus` is the runtime implementation.

4. Error behavior differs by code path:
   - sync handler exceptions are logged and other handlers generally continue;
   - async dispatch collects/logs handler errors in `EventBus`, while the exact behavior differs in `SimpleEventBus` and its `WhenAll`;
   - pending-event replay in `EventBus.Subscribe` catches an individual failure and leaves the failed pending item present;
   - cancellation/disposal semantics are tied to concrete `EventBus` implementation and are not part of `IEventBus` except `Clear()`.

5. Subscription lifecycle:
   - callers explicitly pair Subscribe/Unsubscribe in `EventHandlerBase`, `SystemBase` and phase systems.
   - No disposable subscription token is exposed by `IEventBus`; no such API should be added until caller/lifecycle migration is designed.
   - `EventBus.Clear()` clears handler/queue state under a lock, but its additional cached processors/pending events/monitored type state and async loop lifecycle require explicit review before redefining behavior.

6. Pooling is coupled to the current `IEvent` struct shape and `EventPriority`; `EventPool` maintains mutable type pools and statistics. Keep pooling as a separate optional policy/adapter, not a mandatory dependency of a neutral bus.

## Registration and caller mapping — follow-up

7. Installer order is confirmed from `InstallerRegistry.RegisterInstallers`: `CoreInstaller` runs before `EventsInstaller`. `CoreInstaller` initially binds `IEventBus -> SimpleEventBus`; `EventsInstaller` later binds either `IEventBus -> NetworkEventBus` when `INetworkSystem` is registered, or `IEventBus -> EventBus` otherwise. The final resolved service therefore depends on DI duplicate-binding semantics. The current `DIContainer.RegisterWithLifetime` replaces `_registrations[serviceType]`, but the installer helper uses binding methods and the full binding path must be inspected before declaring which implementation actually wins. `GameBootstrapper` resolves `IEventBus` after all installers run.

8. Caller search establishes broad dependence on `IEventBus` from Game Flow/State Machine, Systems, UI presenters/views, ECS `SystemBase` and `ComponentCacheRegistry`, event batching/throttling/debugging. Sync subscribe/unsubscribe is widely used. Async subscribe usage is visibly concentrated in `PhaseSystem` plus its helper wrappers; exact comprehensive counts should be generated from the full tree, not inferred from top search results.

9. `EventsInstaller` also registers `EventBatcher`, `EventThrottler`, `EventThrottlerUpdateSystem`, `EventStore`, and `EventLogger`, making the installer and debug/logging features part of the compatibility mapping. `NetworkEventBus` derives from `EventBus`, subscribes to network messages, and overrides `Publish<T>` to synchronize marked/registered network events. Any neutral local event contract must not drag networking message/authority abstractions into Framework Core.

10. **Confirmed registration-order defect:** `InstallerRegistry` executes `CoreInstaller` before `EventsInstaller`. `CoreInstaller` registers `IEventBus -> SimpleEventBus`. Inside `EventsInstaller`, `container.Resolve<ISystemRegistry>()` runs before the later `IEventBus` registration. `SystemRegistry` constructor immediately resolves/captures `IEventBus` and subscribes to `PhaseChangedEvent`, which materializes the current `SimpleEventBus` singleton. Later registration of `EventBus` or `NetworkEventBus` only overwrites `_registrations[IEventBus]`; `DIContainer.ResolveByType` checks `_singletons` before `_registrations`, so it can continue returning the already-created `SimpleEventBus`. `GameBootstrapper` resolves the same cached `IEventBus` after installers. This conflicts with EventsInstaller's stated intent to replace the simple bus. CI/source inspection proves the path; runtime branch behavior should be verified with a focused test before changing code.

11. `InstallerRegistry` catches installer exceptions and continues; `EventsInstaller` also registers the timer system through `ISystemRegistry`. PR #23 preserved that lifecycle and changed only the order of bus registration versus registry resolution. It added `EventsInstaller_RegistersFinalEventBusBeforeResolvingSystemRegistry`; Unity 6000.0.45f1 EditMode XML reports **23 passed, 0 failed, 0 skipped**, and GitHub Actions on head `280882f1550cd3befcd58def29f76d08fd93b163` passed. Full-project compile/Console and bootstrap smoke specific to PR #23 are not separately confirmed.

12. **Confirmed network selection gap:** `NetworkingInstaller` binds `IClientNetworkSystem -> ClientNetworkSystem` and `IServerNetworkSystem -> ServerNetworkSystem` as singletons, but it does not register the exact `INetworkSystem` key. `DIContainer.IsRegistered(Type)` checks exact dictionary keys, not assignability; the `EventsInstaller` check therefore returns false for these bindings alone and chooses regular `EventBus`.

13. `INetworkSystem` is a shared interface; `IClientNetworkSystem` and `IServerNetworkSystem` both derive from it but expose role-specific capabilities. `ClientNetworkSystem` wraps `INetworkClient`; `ServerNetworkSystem` wraps `INetworkServer`. A separate `NetworkSystem : INetworkSystem` also exists and composes both low-level client and server services, but `NetworkingInstaller` does not register it.

14. `NetworkAuthority` directly depends on both `IClientNetworkSystem` and `IServerNetworkSystem`. `NetworkEventBus` depends on `INetworkSystem` and uses `is IClientNetworkSystem` / `is IServerNetworkSystem` to enforce event-authority attributes. Blindly binding either role singleton to `INetworkSystem` may cause wrong behavior for the other role. Registering the separate combined `NetworkSystem` is not a drop-in solution: that class implements only `INetworkSystem`, so `NetworkEventBus`'s role checks would not recognize it as client or server. It would need a designed role abstraction or adapter and reconciliation with the duplicate wrappers.

15. Source search finds no production call sites for `NetworkAuthority.SetClientActive` / `SetServerActive` and no visible gameplay call sites of `IClientNetworkSystem.ConnectToServerAsync` / `IServerNetworkSystem.StartServer` outside their declarations/implementations in the indexed source. Treat this as a likely incomplete networking integration, not proof that no reflection/Unity scene/external code depends on these APIs. Before implementing composition, trace serialized scene objects, UI/menu call paths, and bootstrap/runtime usage in the checked-out project.

## Follow-up implementation — PR #24 (merged and validated)

- PR head at last recorded review: `9ced67674576127d98b3cd328b9b85917faacb18`.
- Explicit `EventNetworkMode` supports `Offline`, `Client`, `Server`; `GameBootstrapper` stores it as a serialized composition-root setting, with Offline as the compatibility default.
- `NetworkingInstaller` binds the selected existing role singleton object to the exact `INetworkSystem` key. Offline does not bind that key.
- `EventsInstaller` now chooses the event bus from `EventNetworkMode`, not a DI type-presence heuristic.
- For network mode, register/resolves one `NetworkEventBus` singleton under `INetworkEventBus` and binds that same object instance as `IEventBus`. This avoids independently constructed buses with split subscriber registries/queues.
- `EventNetworkAuthorityPolicy` decides whether the configured local role may send Server/Client/Both-authority events. `NetworkEventBus.Dispose` unregisters its network receive handler before disposing the base bus.
- Reflection-based EditMode tests cover authority policy, the exact role instance alias, offline/client/server bus selection, and bus identity shared with `SystemRegistry`.
- GitHub Actions on the reviewed head passed: framework boundary/GUID validation and .NET ECS tests; Minimal CI was successful but is only a no-op workflow.
- Unity 6000.0.45f1 EditMode on the PR #24 head: **23 passed, 0 failed, 0 skipped**; user confirmed full project compile, clean Console and bootstrap/game smoke. Both GitHub Actions passed. Merge SHA: `178b1a3e10e4905b0ae162bf73d6e2c3a95386db`.
- Host mode is deliberately absent: current implementation does not support simultaneous client+server operation. Do not represent Host as working unless transport behavior and authority are implemented and tested.
- New Unity assets currently have explicit Unity `.meta` GUIDs: `EventNetworkMode.cs.meta` = `d153502a8dbb4a09a100000000000004`; `EventNetworkAuthorityPolicy.cs.meta` = `d153502a8dbb4a09a100000000000005`.

## Decision
Do not move/rename `IEvent`, `IEventBus`, or `IEventPool` into a Framework assembly yet. A neutral extraction would first require an explicit choice about event payload identity, priority (if any), sync/async abstraction, ordering, cancellation, error propagation, and subscription lifetime. Avoid creating an empty contracts assembly or changing runtime behavior in an audit task.

## Next bounded task
1. Trace the application/game mode decision: offline, client, server, or combined host.
2. Decide the canonical composition path: bind the existing combined `NetworkSystem`, explicitly select one role service, or use a deliberate adapter. Do not alias an arbitrary singleton.
3. Add installer/DI characterization tests proving offline uses `EventBus`, network mode uses `NetworkEventBus`, and `SystemRegistry` captures the same selected bus.
4. Verify `NetworkAuthority` and existing callers remain compatible with the selected networking model.
5. Continue event-semantics mapping; consider a neutral synchronous contract only if concrete cross-module reuse justifies it.
