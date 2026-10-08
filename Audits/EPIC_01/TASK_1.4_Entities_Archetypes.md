# Audit — Entities/Archetypes

## 1. Чи потрібна будь-якій RPG?
Так. Archetype/template concepts є фундаментальним ECS/RPG framework pattern для опису складу entity та створення entity з preset configuration.

## 2. Чи залежить від конкретної гри?
Змішано.
- Generic archetype detection/registry/template ideas — універсальні.
- Поточна реалізація містить MythHunter-specific components, HeroArchetypeSO, default Item archetype та gameplay data.

## 3. Чи можна перевикористати без змін?
Ні.
Модель потрібно розділити на generic archetype/template infrastructure та Game Layer definitions.

## 4. Framework чи Game Layer?
Mixed. Core archetype infrastructure — Framework candidate; HeroArchetypeSO та конкретні definitions — Game Layer.

## 5. Чи є архітектурні проблеми?
- Існує кілька перекриваючих моделей: IEntityArchetype/EntityArchetypeBase та ArchetypeTemplateRegistry/ArchetypeTemplateBuilder.
- ArchetypeSystem одночасно є ECS system, event subscriber, factory facade та registry coordinator.
- ArchetypeRegistry зберігає окремий кеш entity→archetype та archetype→entities, який може дублювати ECS structural information.
- ArchetypeDetector перебирає всі template IDs для кожної entity; це лінійний пошук і може масштабуватись погано.
- ArchetypeTemplateRegistry містить hardcoded MythHunter components та default Item archetype.
- Delegate cache має ручний whitelist трьох компонентів і fallback до reflection.
- Reflection fallback у hot/runtime path ускладнює продуктивність і типобезпеку.
- IArchetypeTemplateRegistry повертає concrete ArchetypeTemplateBuilder замість interface builder.
- Template overrides використовують Dictionary<Type, object>, що слабко типізовано.
- ArchetypeTemplate зберігає DefaultComponents як mutable Dictionary<Type, object>.
- Archetype matching переважно перевіряє наявність компонентів, але checker semantics дозволяють domain predicates у template infrastructure.
- ArchetypeSystem публікує EntityCreatedEvent сам і також реагує на EntityCreatedEvent, що створює зайве дублювання відповідальності.
- ArchetypeSystem успадковує SystemBase лише для event lifecycle; сама archetype registry логіка не потребує update loop.
- HeroArchetypeSO дуже сильно game/UI/domain specific і залежить від Unity ScriptableObject, HeroRace/HeroClass, stats, skills, combat modifiers, professions.
- EntityArchetypeBase є альтернативною implementation model, але не видно інтеграції з TemplateRegistry, що вказує на незавершену/legacy abstraction.

## 6. Від чого залежить?
Core.ECS, Core.DI, Events, Logging; UnityEngine для HeroArchetypeSO; concrete MythHunter Components і Heroes domain.

## 7. Хто залежить від цього?
EntityFactory, HeroFactory/archetype consumers, entity creation systems та gameplay/application composition.

## 8. Чи є зайві залежності?
Так.
- ArchetypeSystem -> EventBus лише через lifecycle integration.
- ArchetypeTemplateRegistry -> concrete MythHunter components.
- HeroArchetypeSO -> combat/UI/domain data.
- Logger у registry/detector додає infrastructure coupling.
- Reflection fallback компенсує відсутність generic ECS API.

## 9. Що потребує рефакторингу?
- Вибрати одну canonical archetype/template model.
- Винести generic ArchetypeDefinition/Template infrastructure у Framework.
- Перенести конкретні Hero/Item/Enemy definitions у Game Layer.
- Прибрати hardcoded defaults з framework registry.
- Замінити Dictionary<Type, object> на типізований/component-definition API.
- Додати generic ECS component-type enumeration/query API, щоб прибрати reflection fallback.
- Відокремити registry від event/system lifecycle.
- Прибрати duplicate EntityCreatedEvent ownership.
- Визначити, чи archetype є structural ECS archetype, semantic classification або spawn template. Зараз ці поняття змішані.
- Видалити/мігрувати legacy EntityArchetypeBase після вибору canonical model.

## Висновок:
☑ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Mixed. Поточний модуль має сильну Framework-ідею, але фактично змішує ECS archetype concepts, spawn templates, semantic classification і MythHunter hero definitions.