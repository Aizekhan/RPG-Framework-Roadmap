# Audit — Components/Lobby

## 1. Чи потрібна будь-якій RPG?
Лише частково. Lobby/ready/selection concepts можуть бути потрібні multiplayer RPG, але це не універсальна RPG core-модель.

## 2. Чи залежить від конкретної гри?
Так, явно.
- HeroSelectionComponent прив'язаний до ArchetypeId, ManaCost та конкретної моделі вибору героїв.
- LobbyStateComponent містить SelectedHeroIds, RemainingMana та SelectionTimeLeft.
- Назви та поля відображають конкретний MythHunter lobby flow.

## 3. Чи можна перевикористати без змін?
Ні. Для універсального framework потрібні generic session/player-selection contracts, а не конкретні hero/mana semantics.

## 4. Framework чи Game Layer?
Game Layer.

## 5. Чи є архітектурні проблеми?
- Lobby component layer змішує multiplayer session state і game-specific hero selection.
- ManaCost/RemainingMana — gameplay economy, яка не належить generic lobby.
- Hero IDs і ArchetypeId прямо протікають у lobby abstraction.
- Serialization methods є placeholder і повертають порожній byte array.
- LobbyStateComponent агрегує ready state, resource state, selected heroes і timer.
- PlayerIndex та SelectedHeroIds потребують окремої моделі player/selection context.

## 6. Від чого залежить?
Core.ECS / ISerializableComponent. Domain semantics залежать від Hero Archetype та Lobby gameplay.

## 7. Хто залежить від цього?
Lobby systems, hero selection systems, UI/presenters і game flow.

## 8. Чи є зайві залежності?
Прямих зайвих imports немає, але самі data contracts містять зайві для generic lobby gameplay concerns.

## 9. Що потребує рефакторингу?
- Винести generic lobby/session state у окремий multiplayer/session module.
- Винести hero selection у Game Layer.
- Відокремити ready state, resource state, timer та selection state.
- Перенести serialization із компонентів.
- Універсалізувати player/session identifiers.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer. Компоненти моделюють конкретний MythHunter lobby та hero-selection flow.