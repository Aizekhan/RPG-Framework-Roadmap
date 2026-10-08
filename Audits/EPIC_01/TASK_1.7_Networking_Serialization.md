# Audit — Networking/Serialization

## 1. Чи потрібна будь-якій RPG?
Так, якщо RPG має multiplayer/network transport. Serialization є обов'язковим protocol concern для передачі повідомлень і стану.

## 2. Чи залежить від конкретної гри?
Концепція — ні. Поточна реалізація залежить від MythHunter message/serialization/logger infrastructure та CLR type discovery.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Framework candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- BinaryNetworkSerializer автоматично сканує всі assemblies і реєструє типи повідомлень.
- Serialization protocol використовує Type.FullName як wire identifier — нестабільний schema contract.
- Generic Deserialize<T> читає typeName, але фактично ігнорує його.
- Non-generic Deserialize приймає messageType, але фактично використовує typeName з payload; параметр messageType не є authoritative.
- Runtime Activator.CreateInstance використовується для inbound message creation.
- Немає protocol version, schema version або compatibility policy.
- Немає чіткої перевірки payload length перед ReadBytes/Deserialize.
- Повторно змішуються message identity, CLR type identity і serialization identity.
- Logger є mandatory dependency низькорівневого serializer.
- DeltaSerializer зберігає state cache у пам'яті без lifecycle/eviction/session ownership.
- DeltaSerializer працює на рівні raw byte offsets, тому schema change може зробити delta несумісною.
- ApplyDelta перевіряє offset < length, але негативний offset не перевіряється.
- DeltaSerializer прив'язаний до persistence ISerializable, хоча delta transport є networking concern.
- Поріг 0.8f hardcoded.
- Немає sequence number/base revision, тому reorder/loss/replay delta packets не контролюються.

## 6. Від чого залежить?
INetworkMessage, ISerializable, BinaryReader/BinaryWriter, reflection, DI/logger.

## 7. Хто залежить від цього?
NetworkClient, NetworkServer, NetworkSystem, NetworkEventBus, Delta-based replication.

## 8. Чи є зайві залежності?
Так:
- Persistence serialization не слід використовувати як network wire contract.
- CLR reflection/discovery не повинен визначати protocol identity.
- DeltaSerializer не повинен напряму залежати від generic persistence serializer.

## 9. Що потребує рефакторингу?
- Ввести explicit protocol schema registry з stable message IDs.
- Додати protocol/schema versions та migration/compatibility rules.
- Розділити message codec від persistence serialization.
- Замість assembly scanning використати explicit registration/bootstrap.
- Винести object factory/codec lookup з hot path reflection.
- Додати payload limits і malformed packet validation.
- Зробити serialization failure explicit.
- Для delta replication ввести entity/session/revision/sequence metadata.
- Визначити lifecycle/cache eviction policy для delta state.
- Не використовувати raw byte diff як єдиний replication model без versioning.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
