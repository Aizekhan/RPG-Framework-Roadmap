# Audit — Events/EventBus

## 1. Чи потрібна будь-якій RPG?
Так. Event bus is generic Framework infrastructure for decoupled domain/application communication.

## 2. Чи залежить від конкретної гри?
Поточна реалізація — так.
- Direct imports of MythHunter domain events.
- UnityEditor dependency in runtime source.
- MythLogger/UniTask/custom extensions are embedded.

## 3. Чи можна перевикористати без змін?
Ні. Core event-bus idea is reusable, but implementation must be split into a small generic bus and optional scheduling/middleware modules.

## 4. Framework чи Game Layer?
Framework candidate with significant Technical Debt and accidental Game Layer coupling.

## 5. Чи є архітектурні проблеми?
- EventBus is a very large multi-responsibility class: subscription, sync/async dispatch, priority queues, pending events, caching processors, lifecycle/cancellation, pooling and debugging integration.
- It directly imports concrete MythHunter domain events and Lobby events.
- It imports UnityEditor in runtime code, creating an invalid/undesirable editor dependency in the event infrastructure.
- DynamicInvoke is used for cached sync handlers, so the hot path still performs dynamic invocation.
- Async dispatcher cache stores a single dispatcher per event type even though multiple async handlers can be registered, which risks overwriting prior handlers.
- Pending-event caching semantics are implicit and can retain events until a handler appears.
- Critical events bypass the normal queue, so ordering semantics differ by priority.
- Event pool return semantics are embedded in dispatch, coupling bus delivery with memory-management policy.
- Cancellation token source starts background processing from the constructor, making object construction have runtime side effects.
- Error policy is mixed: some handler failures are caught/logged while dispatch continues, with no explicit failure/result contract.
- Event logging calls GetEventId/GetPriority on every publish; event ID stability is a separate contract concern.
- DebugEventMiddleware is directly invoked from core dispatch.

## 6. Від чого залежить?
System.Collections, threading, UniTask, DI, logging, event extensions, concrete domain events and UnityEditor.

## 7. Хто залежить від цього?
Most systems/services/UI that publish or subscribe to events; NetworkEventBus also builds on this infrastructure.

## 8. Чи є зайві залежності?
Так: concrete domain events, UnityEditor, debug middleware, logger, pooling and queue scheduler all belong outside the minimal bus.

## 9. Що потребує рефакторингу?
- Reduce EventBus to generic typed subscription/publication.
- Move scheduling/priority queue into a separate dispatcher/scheduler.
- Move async dispatch into separate async event dispatcher.
- Move pooling to an optional pool/allocator layer.
- Move pending-event/replay semantics into explicit replay/buffer middleware.
- Move debug middleware out of the core bus.
- Remove UnityEditor dependency entirely from runtime event code.
- Remove direct domain event references from generic bus.
- Define stable event ordering and error semantics explicitly.
- Verify multi-handler async registration semantics and cancellation lifecycle.

## Висновок:
☑ Framework
☐ Game Layer
☑ Technical Debt

Загальна класифікація: Framework candidate + Technical Debt. EventBus is strategically important Framework infrastructure, but the current implementation is an oversized dispatcher with game/editor/debug/pooling concerns mixed together.