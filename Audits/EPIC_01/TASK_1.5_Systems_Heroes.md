# Audit — Systems/Hero Systems

## 1. Чи потрібна будь-якій RPG?
Hero/character lifecycle systems потрібні RPG, але race/class bonuses — domain-specific mechanics.

## 2. Чи залежить від конкретної гри?
Так.
- HeroSystem залежить від IHeroFactory, concrete Hero data, Stats/Health components та persistence.
- RaceClassBonusSystem напряму використовує HeroRace/HeroClass і конкретний MythHunter StatType/skill IDs.

## 3. Чи можна перевикористати без змін?
Ні. Generic character lifecycle idea можна винести, але current implementation — MythHunter Game Layer.

## 4. Framework чи Game Layer?
Game Layer.

## 5. Чи є архітектурні проблеми?
- HeroSystem одночасно створює, завантажує, зберігає, знищує героїв і керує ComponentCache.
- HeroSystem має залежність від ComponentCacheRegistry, хоча cache maintenance — окрема infrastructure/runtime concern.
- HeroSystem Update фактично підтримує лише два кеші, тобто lifecycle system виконує технічну роботу замість domain logic.
- CreateHeroWithComponents використовує Dictionary<Type, object>, слабко типізований API.
- RaceClassBonusSystem hardcodes all race/class definitions, stat bonuses and ability IDs directly in executable code.
- RaceClassBonusSystem змішує configuration/balance data та application logic.
- Bonus application напряму мутує StatsComponent/SkillsComponent.
- Event-driven application може повторно застосувати bonuses, якщо HeroCreatedEvent буде доставлений повторно; explicit idempotency policy відсутня.
- ClassBonuses/RacialBonuses є mutable data objects.
- HeroSystem defines IHeroSystem inline у concrete system file, що розмиває contract ownership.
- Heavy dependencies на Hero domain, ECS, persistence factory, DI, events and logging.

## 6. Від чого залежить?
Core.ECS, Core.DI, Events, Hero entities/factory, Character/Combat components, ComponentCache, UniTask та logging.

## 7. Хто залежить від цього?
Hero creation/load flows, gameplay installers, UI/lobby hero selection та systems, які очікують HeroCreatedEvent/hero lifecycle.

## 8. Чи є зайві залежності?
Так.
- HeroSystem -> ComponentCacheRegistry.
- RaceClassBonusSystem -> EventBus та concrete event semantics для simple configuration application.
- Bonus definitions -> concrete StatsComponent/StatType and skill IDs.

## 9. Що потребує рефакторингу?
- Розділити Hero lifecycle/application service від cache maintenance.
- Винести race/class definitions у data/config assets or registries.
- Створити generic modifier/bonus model незалежно від конкретних HeroRace/HeroClass.
- Зробити bonus application deterministic/idempotent.
- Розділити hero persistence від hero runtime lifecycle.
- Винести IHeroSystem у окремий contract file.
- Не передавати Dictionary<Type, object> як основний public customization API.
- Перевірити, чи ComponentCache взагалі потрібен HeroSystem після завершення ECS refactor.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer. Hero Systems містять конкретну MythHunter domain logic; основна проблема — змішування lifecycle, persistence, caching та balance configuration.