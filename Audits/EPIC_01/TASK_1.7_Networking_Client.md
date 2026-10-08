# Audit — Networking/Client

## 1. Чи потрібна будь-якій RPG?
Не кожній RPG, але для multiplayer RPG клієнтський networking — базова інфраструктура.

## 2. Чи залежить від конкретної гри?
Концепція — ні. Поточна реалізація залежить від MythHunter DI/logger/messages/serialization, UniTask та конкретного ще не реалізованого transport.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Framework candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- NetworkClient фактично не має transport: ConnectAsync/DisconnectAsync містять штучний Delay, SendMessage лише серіалізує та логгує.
- Відсутня реальна socket/transport abstraction.
- ConnectAsync не має cancellation token, timeout policy або reconnect policy.
- DisconnectAsync також не має cancellation/timeout policy.
- Serializer підключений, але фактичне доставлення даних відсутнє.
- SimulateMessageReceived є тестовим API всередині production implementation.
- SendMessage silently degrades to logging when disconnected instead of explicit result/failure contract.
- Connection state та transport state змішані; IsActive фактично дублює connected state.
- Події Action не мають явної error isolation policy.
- Адреса/порт — raw primitives без endpoint/value object.
- Client API працює через concrete INetworkMessage, тому serialization/schema/transport boundaries тісно пов'язані.
- NetworkClient залежить від MythLogger, роблячи logger mandatory для низькорівневого transport adapter.
- NetworkSystem зверху дублює client/server orchestration, що свідчить про нечіткий ownership lifecycle.

## 6. Від чого залежить?
INetworkMessage, INetworkSerializer, IMythLogger, DI, UniTask.

## 7. Хто залежить від цього?
NetworkSystem, NetworkEventBus та інші multiplayer systems.

## 8. Чи є зайві залежності?
Так:
- Logging не повинен бути обов'язковою частиною transport adapter.
- Serialization має бути окремим protocol layer.
- Client не повинен одночасно визначати transport + application message contract.
- Simulation/test API має бути окремим test transport.

## 9. Що потребує рефакторингу?
- Виділити transport client abstraction.
- Реалізувати або підключити реальний transport adapter.
- Додати cancellation, timeout, reconnect/disconnect semantics.
- Розділити transport bytes, protocol serialization і application messages.
- Ввести explicit send result/error policy.
- Прибрати SimulateMessageReceived з production client.
- Винести test/mock transport окремо.
- Чітко визначити ownership connection state між Client та NetworkSystem.
- Зробити networking module optional для single-player RPG.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
