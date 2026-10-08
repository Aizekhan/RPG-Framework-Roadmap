# Audit — Events/Event Pipeline

## 1. Чи потрібна будь-якій RPG?
Так. Черга, порядок, async dispatch, batching та throttling можуть бути корисними в будь-якій RPG, але це окремі pipeline-модулі, а не обов'язкова відповідальність EventBus.

## 2. Чи залежить від конкретної гри?
Ідея generic, поточна реалізація сильно залежить від Unity, UniTask, MythLogger та поточного EventBus.

## 3. Чи можна перевикористати без змін?
Ні. Потрібне розділення на незалежні pipeline-компоненти.

## 4. Framework чи Game Layer?
Framework candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- EventBus одночасно виконує publication, scheduling, priority dispatch, sync/async execution, pooling, buffering і lifecycle.
- Sync/async queues живуть усередині EventBus; pipeline policy фактично захована в bus.
- Critical events обходять queue, створюючи окрему семантику ordering.
- EventBatcher не формує batch event, хоча API це обіцяє; фактично повторно публікує кожну подію окремо.
- EventBatcher використовує reflection для generic Publish.
- EventBatcher залежить від UnityEngine.Time.
- EventBatcher має TODO для batchProcessor, тобто частина контракту не реалізована.
- EventThrottler має дві різні моделі throttling; одна keyed by event type, інша keyed by arbitrary string.
- EventThrottler залежить від Unity Time і dynamic dispatch.
- EventPool змішує allocation policy, statistics та priority metadata.
- SimpleEventBus дублює IEventBus/EventBus і має іншу поведінку priority/queue semantics.
- EventHandlerBase додає logging/lifecycle wrapper навколо bus, що може бути Framework utility, але зараз змішує subscription lifecycle з application logging.
- DebugEventMiddleware є глобальним static pipeline hook.

## 6. Від чого залежить?
IEventBus, IEvent, EventPriority, UniTask, Unity time APIs, logger/DI та конкретні dispatch implementations.

## 7. Хто залежить від цього?
EventBus consumers, systems, services, UI, network events та debug tooling.

## 8. Чи є зайві залежності?
Так:
- Unity time should be abstracted from generic pipeline.
- Logging/DI should not be mandatory for low-level event scheduling.
- Reflection/dynamic invocation should not be required for batching/throttling.
- Pooling and batching should be optional modules.

## 9. Що потребує рефакторингу?
- Виділити EventDispatcher/Scheduler з EventBus.
- Формалізувати ordering guarantees та synchronous/asynchronous semantics.
- Виділити optional Queue/Batch/Throttle/Replay components.
- Винести time source через абстракцію.
- Прибрати reflection/dynamic із normal pipeline path.
- Видалити або виправдати SimpleEventBus як окрему тестову реалізацію.
- Чітко розділити memory pooling від event delivery.
- Додати cancellation/shutdown lifecycle як явний контракт.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
