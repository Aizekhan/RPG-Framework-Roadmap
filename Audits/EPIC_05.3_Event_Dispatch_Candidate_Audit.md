# EPIC 05.3 — Event Dispatch Candidate Audit

## Status
Initial source-backed audit COMPLETE. Event dispatch contract extraction is **not yet approved**; implementation must wait for semantics and all consumers to be mapped.

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

11. `InstallerRegistry` catches installer exceptions and continues; `EventsInstaller` also registers the timer system through `ISystemRegistry`. Preserve that lifecycle while fixing the binding sequence. Do not change dispatch/pooling semantics as part of the wiring correction.

## Decision
Do not move/rename `IEvent`, `IEventBus`, or `IEventPool` into a Framework assembly yet. A neutral extraction would first require an explicit choice about event payload identity, priority (if any), sync/async abstraction, ordering, cancellation, error propagation, and subscription lifetime. Avoid creating an empty contracts assembly or changing runtime behavior in an audit task.

## Next bounded task
1. Inventory all registrations/bindings of `IEventBus` and concrete event-bus construction/use.
2. Map publish/subscribe consumers, including systems, network adapter, event batcher and debug middleware.
3. Record which semantics existing production callers actually depend on, especially ordering and async behavior.
4. Propose a minimal neutral synchronous contract first only if concrete cross-module reuse justifies it; keep async, priority, pooling and networking as separately justified capabilities.
5. Add characterization tests before any implementation change.
