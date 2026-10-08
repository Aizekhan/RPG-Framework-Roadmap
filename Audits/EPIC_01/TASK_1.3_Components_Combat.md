# Audit — Components/Combat

## 1. Чи потрібна будь-якій RPG?
Частково. Health, team, combat abilities, combat stats та combat stance є типовими RPG concepts. Але поточні поля й правила моделюють конкретну бойову систему.

## 2. Чи залежить від конкретної гри?
Так.
- CombatStatsComponent містить конкретний набір combat stats.
- CombatStyleComponent має три конкретні стійки та hardcoded modifiers.
- CombatAbilityComponent містить конкретну модель auto-activation.
- CombatStatsComponent і HealthComponent містять runtime combat state.

## 3. Чи можна перевикористати без змін?
Ні. Частину компонентів можна узагальнити, але поточна модель занадто прив'язана до конкретних mechanics.

## 4. Framework чи Game Layer?
Переважно Game Layer. HealthComponent і TeamComponent мають найсильніший Framework potential.

## 5. Чи є архітектурні проблеми?
- Компоненти самі реалізують binary serialization.
- CombatStatsComponent змішує persistent stats та transient combat state: IsInCombat, target, LastAttackTime, rage/concentration.
- HealthComponent змішує health data, death state, invulnerability, regeneration і timing state.
- CombatAbilityComponent змішує ability definitions/ids, cooldown/use state та AI-like auto activation.
- CombatStyleComponent зберігає в компоненті balance modifiers для stance rules.
- TeamComponent з int TeamId не має універсального team identity contract.
- UnityEngine у CombatStatsComponent є зайвою залежністю: файл імпортує UnityEngine, але не використовує його.
- Немає очевидного separation між definition/config data і runtime state.

## 6. Від чого залежить?
Core.ECS / IComponent / ISerializableComponent; System.IO; System; а CombatStatsComponent додатково має зайвий UnityEngine dependency.

## 7. Хто залежить від цього?
Очікувано Combat, AI, Movement/Targeting, Damage/Health та gameplay systems.

## 8. Чи є зайві залежності?
Так. UnityEngine using у CombatStatsComponent не використовується. Більша проблема — serialization/policy та gameplay rules знаходяться всередині data components.

## 9. Що потребує рефакторингу?
- Відокремити generic Health, Team і combat state primitives від MythHunter-specific combat model.
- Розділити definition/config data та runtime state.
- Винести serialization у dedicated infrastructure.
- Розділити ability definition, ability loadout та ability runtime state.
- Винести stance modifiers/rules у configuration або combat rules module.
- Визначити універсальний TeamId/Team relationship contract.
- Видалити невикористаний UnityEngine dependency.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer. Найбільш універсальні кандидати для Framework — HealthComponent та TeamComponent; решта потребує суттєвої декомпозиції.