# Audit — Systems/Gameplay Systems

## 1. Чи потрібна будь-якій RPG?
Так, gameplay systems є необхідною частиною будь-якої RPG. Але конкретні systems мають належати Game Layer, а Framework повинен містити лише reusable mechanics/infrastructure.

## 2. Чи залежить від конкретної гри?
Так, майже повністю.
- Combat systems знають конкретні Combat components, damage rules, Rage, Concentration, stances та ability IDs.
- EntitySpawnSystem знає PlayerCharacter/Enemy та MythHunter factory/archetypes.

## 3. Чи можна перевикористати без змін?
Ні. Окремі algorithms можна винести у generic modules, але поточні systems є MythHunter gameplay implementations.

## 4. Framework чи Game Layer?
Game Layer.

## 5. Чи є архітектурні проблеми?
- CombatAbilitySystem hardcodes concrete ability IDs and their implementations directly in the system.
- CombatAbilitySystem mixes ability registry, ability execution, passive processing, auto-activation, health checks, teleportation and combat event reactions.
- Passive effects are executed every Update for every entity with CombatAbilityComponent; implementations mutate stats directly and can repeatedly stack effects (for example HealthRegen and CombatMaster), which is a serious correctness risk.
- CombatAbilitySystem uses UniTask.Delay without an explicit cancellation/lifecycle ownership for delayed invulnerability reset.
- CombatAbilitySystem contains Unity-specific Position/Time usage.
- CombatSystem is a large multi-responsibility system covering detection, combat lifecycle, attack calculations, damage, death, resource regeneration, stance changes, rage/concentration exchange and ability delegation.
- CombatSystem combines rules, state mutation, timing, randomness and event publication.
- Combat mechanics are hardcoded into constants and switch logic instead of data-driven rule components/services.
- EntitySpawnSystem couples event handling, factory selection, archetype orchestration, logging and domain event emission.
- EntitySpawnSystem resolves concrete PlayerCharacter/Enemy semantics through IEntityFactory.
- Spawn events and spawned events create application workflow inside the system rather than generic spawn infrastructure.
- Gameplay systems depend on Core.ECS, DI, Events and Logging; some additionally depend on Unity/UniTask.

## 6. Від чого залежить?
Core.ECS, DI, Events, Components, EntityFactory/Archetypes, Logging; CombatAbilitySystem additionally uses UniTask and UnityEngine.

## 7. Хто залежить від цього?
Gameplay installers, phase systems, UI/gameplay controllers, event flows and other gameplay systems.

## 8. Чи є зайві залежності?
Так. Unity/UniTask, direct concrete component dependencies, direct event coupling and logging are tightly embedded in gameplay logic. More importantly, gameplay rules are centralized in monolithic systems.

## 9. Що потребує рефакторингу?
- Keep these systems in Game Layer.
- Split CombatSystem into focused systems/services: combat detection, attack resolution, damage application, resource regeneration, stance/rule evaluation, death handling.
- Extract ability definitions/handlers from CombatAbilitySystem into data + ability execution modules.
- Make passive effects event-driven or explicitly periodic and idempotent instead of unconditional per-frame mutation.
- Introduce generic damage/health/targeting contracts where Framework reuse is desired.
- Separate spawn orchestration from domain-specific character/enemy factories.
- Move balancing constants into configuration/data.
- Define cancellation/lifecycle for delayed async effects.
- Reduce direct event and logger coupling through explicit ports where useful.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer. Gameplay systems correctly belong outside the reusable Framework, but current combat/spawn systems are heavily coupled, monolithic and contain correctness/performance risks.