# Audit — Events/Middleware

## 1. Чи потрібна будь-якій RPG?
Так. Middleware корисний для logging, tracing, metrics, validation, filtering, debugging та інших cross-cutting concerns навколо доставки подій.

## 2. Чи залежить від конкретної гри?
Концепція — ні. Поточна реалізація має лише локальний generic event contract, але конкретний DebugEventMiddleware є runtime-global static hook.

## 3. Чи можна перевикористати без змін?
Ні. Потрібен формальний middleware/pipeline контракт.

## 4. Framework чи Game Layer?
Framework candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- DebugEventMiddleware є static global state.
- Middleware не є справжнім pipeline: він напряму викликається з EventBus dispatch path.
- Є окремий IDebugEventBus interface, але показана реалізація його не використовує — два несумісні підходи до debugging.
- Debug handlers отримують IEvent + Type, тобто middleware працює поза typed dispatch semantics.
- Один exception у debug handler може вплинути на весь notification cycle, бо немає ізоляції помилок на рівні middleware.
- Немає explicit order, short-circuit, replace/transform, exception policy або cancellation contract.
- Lifecycle middleware прив'язаний до static subscription/unsubscription.
- Debugging concern змішаний із core event delivery; EventBus викликає його без інверсії залежності.
- EventHandlerBase не є middleware і не повинен входити до цього pipeline: він є subscriber helper/lifecycle abstraction.

## 6. Від чого залежить?
IEvent та concrete EventBus dispatch path. Debug implementation — від static state та generic event type information.

## 7. Хто залежить від цього?
EventBus і потенційні debugging/monitoring tools.

## 8. Чи є зайві залежності?
Так: EventBus не повинен напряму знати про debug middleware. Global static state також створює прихований shared state.

## 9. Що потребує рефакторингу?
- Ввести явний middleware contract, наприклад IEventMiddleware.
- Middleware chain має бути injectable/configurable.
- Визначити pipeline order та exception policy.
- Відокремити Debug/Telemetry/Logging middleware від core bus.
- Прибрати static global state.
- Або реалізувати IDebugEventBus поверх нового middleware/monitoring API, або видалити дубльований контракт.
- Не дозволяти middleware породжувати залежність Framework на Game Layer.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
