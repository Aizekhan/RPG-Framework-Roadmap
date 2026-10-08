# Audit — Networking/Server

## 1. Чи потрібна будь-якій RPG?
Не кожній RPG, але для multiplayer RPG серверна інфраструктура є базовим Framework-модулем.

## 2. Чи залежить від конкретної гри?
Концепція — ні. Поточна реалізація залежить від MythHunter DI/logger/messages/serialization і не має реального transport.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Framework candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- NetworkServer фактично є stub: Start/Send/Broadcast не виконують мережевої доставки.
- Реального transport/listener/socket abstraction немає.
- SimulateMessageReceived і SimulateClientConnection є тестовою функціональністю всередині production implementation.
- DisconnectClient доступний публічно та змішує transport/session lifecycle.
- Broadcast serializes message один раз, але фактичного send немає.
- Client collection та next client ID лише моделюють sessions локально, а не представляють реальні transport connections.
- StopAsync використовує штучний delay замість transport shutdown lifecycle.
- Немає cancellation, timeout, backpressure, connection limits або graceful transport shutdown contract.
- Немає sender/session validation на incoming messages.
- Server API напряму працює з application-level INetworkMessage, змішуючи transport та protocol layers.
- Error semantics — тільки logging; caller не отримує explicit send/start/stop result.
- Logger/serialization є обов'язковими dependency низькорівневого server adapter.

## 6. Від чого залежить?
INetworkMessage, INetworkSerializer, IMythLogger, DI, UniTask.

## 7. Хто залежить від цього?
NetworkSystem, network event bridge та multiplayer/game session infrastructure.

## 8. Чи є зайві залежності?
Так:
- transport layer відокремити від protocol/application messages;
- serializer винести на protocol boundary;
- logging зробити optional через diagnostics abstraction;
- simulation methods винести в test/mock server.

## 9. Що потребує рефакторингу?
- Ввести transport server abstraction.
- Реалізувати connection/session management через transport.
- Відокремити transport frames/bytes, serialization та application messages.
- Ввести explicit lifecycle/result/error model.
- Додати cancellation/timeout/shutdown policy.
- Додати inbound validation та client/session identity.
- Прибрати simulation/test API з production server.
- Зробити server module optional для single-player RPG.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
