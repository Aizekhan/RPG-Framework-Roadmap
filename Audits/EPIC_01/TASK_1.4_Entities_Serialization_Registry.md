# Audit — Entities/Serialization Registry

## 1. Чи потрібна будь-якій RPG?
Так. Component serialization registry є універсальною infrastructure capability для persistence, networking, save/load та replication.

## 2. Чи залежить від конкретної гри?
Концепція універсальна, але поточна реалізація має MythHunter-specific registrations та залежність від MythHunter logging/DI.

## 3. Чи можна перевикористати без змін?
Ні.
Registry architecture можна перевикористати після відділення concrete serializers та persistence/network concerns.

## 4. Framework чи Game Layer?
Framework candidate, але поточна реалізація mixed через hardcoded game registrations.

## 5. Чи є архітектурні проблеми?
- IComponentSerializerRegistry фізично знаходиться в Entities, але namespace належить Data.Serialization — ownership boundary нечіткий.
- ComponentSerializerRegistry hardcodes Core component serializers.
- Registry зберігає serializers як Dictionary<Type, object>, тому type safety частково втрачається на storage boundary.
- Serializer lookup помилково/надмірно залежить від runtime registration state.
- Відсутній явний duplicate registration policy.
- Missing serializer повертає null/default після logging, що може маскувати serialization failure.
- VersionedSerializer використовує один global CURRENT_VERSION для всіх типів, замість version per schema/type.
- VersionedSerializer записує CLR FullName у payload, що створює fragile runtime type identity.
- VersionedSerializer лише попереджає про version/type mismatch, а не зупиняє несумісне читання.
- Немає framing/validation limits для dataLength.
- Serialization implementations дублюються: частина компонентів має власний Serialize/Deserialize, паралельно існує external serializer registry.
- Binary serialization protocol не має central schema definition.

## 6. Від чого залежить?
Core.ECS/IComponent, Core.DI, MythLogger, System.IO, конкретні component serializers.

## 7. Хто залежить від цього?
Serialization installer, persistence/networking systems та компоненти, які використовують serializer contracts.

## 8. Чи є зайві залежності?
Так: registry залежить від MythHunter logger/DI; interface location в Entities створює architectural coupling; versioned serializer залежить від concrete CLR type names.

## 9. Що потребує рефакторингу?
- Перемістити serialization contracts у окремий Framework.Serialization module.
- Залишити component-specific serializers поза Entities.
- Прибрати hardcoded default serializers із generic registry.
- Ввести typed serializer registry без object-based storage або приховати casting у safe adapter.
- Визначити failure policy: missing/incompatible serializer має бути explicit error/result.
- Перейти до per-type schema/version contracts.
- Відмовитись від CLR FullName як stable wire identifier.
- Визначити єдиний canonical serialization path замість component-owned + registry-owned serialization.

## Висновок:
☑ Framework
☐ Game Layer
☑ Technical Debt

Загальна класифікація: Framework candidate + Technical Debt. Serialization registry має бути інфраструктурним модулем, а не частиною Entities; поточна реалізація має дублювання та нестабільний version/type protocol.