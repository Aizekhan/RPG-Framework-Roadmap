# Audit — Networking/Messages

## 1. Чи потрібна будь-якій RPG?
Так, для multiplayer RPG потрібен стабільний application/protocol message contract.

## 2. Чи залежить від конкретної гри?
Базовий message contract — Framework. Поточний NetworkEventMessage тісно залежить від EventBus, IEvent, NetworkEventPriority та MythHunter serialization.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Mixed: INetworkMessage — Framework candidate; NetworkEventMessage — Framework networking adapter candidate + Technical Debt.

## 5. Чи є архітектурні проблеми?
- INetworkMessage успадковує ISerializable, тому network protocol contract жорстко пов'язаний із persistence/general serialization.
- GetMessageId() у NetworkEventMessage створює новий Guid при кожному виклику. Ідентифікатор повідомлення таким чином не є стабільним і не підходить як protocol/message type ID.
- EventType зберігається як AssemblyQualifiedName — нестабільний wire identifier, залежний від CLR assembly/type naming.
- Binary serialization hardcoded у concrete message.
- Deserialize не має schema/version/length validation.
- Неконтрольовані довжини можуть створювати malformed input risk.
- DeserializeEvent використовує Activator.CreateInstance та reflection для кожного inbound event.
- GetMethod("Deserialize") не перевіряється на null.
- NetworkEventMessage містить transport policy fields IsReliable/Priority разом із payload.
- Generic NetworkEventMessage<TEvent> покладається на ISerializable самих event structs, змішуючи event model і wire format.
- Дублюється conceptual identity: GetMessageId(), EventType та event network ID.
- Немає explicit protocol version, message schema version або compatibility/migration policy.
- Немає централізованого registry зі стабільними numeric/string schema IDs.
- Serialization exceptions не мають чіткої boundary/error contract.

## 6. Від чого залежить?
ISerializable, IEvent, NetworkEventPriority, System.IO binary APIs, reflection/Activator.

## 7. Хто залежить від цього?
NetworkClient, NetworkServer, NetworkSystem, NetworkEventBus та serialization layer.

## 8. Чи є зайві залежності?
Так:
- INetworkMessage не повинен вимагати persistence ISerializable.
- Concrete event message не повинен знати CLR type identity як wire identity.
- Event payload serialization треба делегувати protocol serializer/registry.
- Transport reliability/priority policy краще винести з generic message payload, якщо transport layer вже володіє цим.

## 9. Що потребує рефакторингу?
- Розділити application message contract від serialization contract.
- Ввести stable message/schema ID.
- Додати protocol versioning.
- Винести serialization у dedicated serializer/codec registry.
- Забрати reflection/Activator з hot inbound path через registered factories/codecs.
- Визначити validation limits для payload lengths.
- Розділити transport metadata від payload DTO.
- Визначити stable message ID semantics окремо від correlation/instance ID.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
