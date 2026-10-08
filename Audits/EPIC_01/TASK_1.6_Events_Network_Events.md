# Audit — Events/Network Events

## 1. Чи потрібна будь-якій RPG?
Не кожній RPG потрібна мережа, але для multiplayer RPG network events є корисним Framework infrastructure module.

## 2. Чи залежить від конкретної гри?
Концепція reusable, поточна реалізація — ні.
NetworkEventBus залежить від MythHunter Networking.Core, Networking.Messages, конкретних network system interfaces та assembly naming.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Framework candidate, але зараз тісно змішаний із Networking implementation. Concrete network events remain domain/game concerns.

## 5. Чи є архітектурні проблеми?
- NetworkEventBus успадковується від EventBus і тому успадковує весь його Technical Debt.
- Event bus напряму знає про transport/network system, тобто domain event dispatch та network replication змішані.
- Constructor автоматично сканує всі assemblies та фільтрує їх по "MythHunter" — жорстка game-specific прив'язка.
- Runtime reflection використовується для discovery, Type.GetType та generic PublishNetworkEvent.
- AssemblyQualifiedName використовується як мережевий type identifier — нестабільний контракт для production protocol/versioning.
- Authority перевіряється локальним type check конкретних server/client network interfaces, що змішує policy з implementation hierarchy.
- Надсилання відбувається прямо всередині Publish, тому local event delivery і network transmission мають один життєвий цикл та можуть мати різні failure semantics.
- Отримана мережна подія викликає локальний Publish через reflection; немає явного validation/authentication/versioning boundary.
- INetworkEvent одночасно є IEvent і ISerializable, тому network transport contract і persistence serialization contract пов'язані.
- Є два джерела metadata: NetworkEventAttribute та INetworkEvent methods.
- Reliable/priority можуть визначатися двома різними шляхами з ризиком розбіжностей.
- IsRegistered фактично не використовується як повноцінний lifecycle state.
- Помилки reflection/assembly scanning частково замовчуються.
- Немає явної перевірки duplicate registration, schema version, replay protection, sender validation або authorization на receive side.

## 6. Від чого залежить?
EventBus/IEvent, serialization, NetworkSystem, network messages, DI/logger, reflection and concrete transport interfaces.

## 7. Хто залежить від цього?
Network-aware gameplay systems and future multiplayer modules.

## 8. Чи є зайві залежності?
Так:
- EventBus не повинен безпосередньо знати transport.
- Persistence serialization не повинен автоматично бути network contract.
- Assembly scan by game name is unnecessary.
- Runtime reflection should be limited to explicit registration/bootstrap.

## 9. Що потребує рефакторингу?
- Відокремити Local EventBus від Network Event Bridge/Replicator.
- Introduce explicit network event registry with stable schema/message IDs.
- Keep network authority/reliability/priority in network metadata, not duplicated across event contract and attribute unless explicitly designed.
- Separate network serialization from persistence serialization.
- Define inbound validation/authentication/versioning.
- Replace automatic assembly name scanning with explicit registration or framework discovery policy.
- Avoid reflection on hot receive path through registered serializers/factories.
- Define loop prevention and delivery guarantees explicitly.
- Make networking optional so non-networked RPG projects do not depend on networking infrastructure.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
