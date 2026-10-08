# Audit — Components/Character

## 1. Чи потрібна будь-якій RPG?
Частково. Загальні character concepts — stats, skills, inventory, identity — потрібні багатьом RPG. Але конкретний набір полів і поведінка тут відображають конкретну RPG-модель героя.

## 2. Чи залежить від конкретної гри?
Так, значною мірою.
- HeroIdentityComponent має Race, Image, Skins.
- SocialSkillsComponent має Religion, Ideology, Class, Professions.
- StatType містить конкретний набір gameplay/economy stats.
- InventoryComponent має конкретні equipment slots, weapons, artifact, elixirs.
- StatRegistry містить конкретні правила, назви, діапазони та UI colors.

## 3. Чи можна перевикористати без змін?
Ні. Загальні концепції можна винести у Framework, але поточні компоненти потребують декомпозиції.

## 4. Framework чи Game Layer?
Переважно Game Layer, із Framework-кандидатами: generic stat container, generic skill container, generic inventory primitives та character identity primitives.

## 5. Чи є архітектурні проблеми?
- Components реалізують власну binary serialization, змішуючи data model і persistence.
- HeroIdentityComponent та SocialSkillsComponent містять явні game/domain semantics.
- InventoryComponent агрегує inventory, equipment та bag у одному компоненті.
- StatsComponent використовує game-specific StatType.
- StatsComponent містить convenience properties тільки для окремих stats — domain logic у data component.
- StatRegistry є static registry з hardcoded gameplay balance values.
- StatRegistry прямо залежить від UnityEngine.Color, змішуючи data/config та presentation concerns.
- StatRegistry namespace MythHunter.Data не відповідає шляху Components/Character.
- StatDefinition mutable.
- Binary serialization не має versioning/schema safeguards.
- List/Dictionary reference-type collections усередині struct components потребують чіткого контролю value/reference semantics.
- Hero-specific naming робить модуль вужчим за універсальний Character module.

## 6. Від чого залежить?
Core.ECS / ISerializableComponent, System.Collections.Generic, System.IO; StatRegistry додатково залежить від UnityEngine та StatType/StatCategory.

## 7. Хто залежить від цього?
Очікувано character, combat, hero, inventory та gameplay systems. Точні usages потребують окремого dependency scan.

## 8. Чи є зайві залежності?
Так: serialization implementation у components, UnityEngine.Color у StatRegistry, hardcoded registry knowledge та game-specific convenience properties.

## 9. Що потребує рефакторингу?
- Відокремити universal Character Framework від MythHunter Hero model.
- Перенести serialization у dedicated serializer/registry layer.
- Розділити Inventory, Equipment і Bag.
- Винести StatDefinition/StatRegistry у data/config module.
- Прибрати Unity Color з core stat definition.
- Зробити stat identity/registry більш універсальними.
- Прибрати game-specific convenience properties із generic stats container.
- Ввести versioned serialization contracts.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer, але всередині є матеріал для майбутнього універсального Character/Stats/Inventory Framework.