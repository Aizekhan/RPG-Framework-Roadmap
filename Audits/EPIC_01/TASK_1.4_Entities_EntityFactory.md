# Audit — Entities/EntityFactory

## 1. Чи потрібна будь-якій RPG?
Так, фабрика сутностей потрібна ECS/RPG framework як abstraction для створення entity. Але конкретні convenience methods для player/enemy є game-specific.

## 2. Чи залежить від конкретної гри?
Так.
- CreatePlayerCharacter та CreateEnemy є конкретними RPG/game concepts.
- Archetype IDs "Character", "Enemy", "Item" hardcoded.
- Direct dependencies на Components.Core.
- Logging використовує MythHunter logger.

## 3. Чи можна перевикористати без змін?
Ні. Поточна реалізація не є generic entity factory.

## 4. Framework чи Game Layer?
Mixed. Generic factory mechanism — Framework candidate; current EntityFactory/IEntityFactory contracts — Game Layer.

## 5. Чи є архітектурні проблеми?
- Interface IEntityFactory exposes only CreatePlayerCharacter/CreateEnemy and therefore encodes game domain into framework boundary.
- CreateEnemy receives health and attackPower, але ці параметри взагалі не записуються в overrides; API і behavior не узгоджені.
- CreatePlayerCharacter/CreateEnemy/CreateItem hardcode archetype strings.
- EntityFactory directly references concrete components.
- CloneEntity uses reflection to invoke generic ECS APIs.
- GetAllComponentTypes is explicitly incomplete and manually maintains a short list of components.
- CloneEntity can therefore silently lose components when cloning entities containing anything beyond the listed core components.
- Clone starts by checking IdComponent rather than a generic entity-existence API.
- Factory depends on archetype system, making entity creation and archetype management tightly coupled.
- Logging is embedded in the factory.
- IEntityFactory does not expose CreateItem despite EntityFactory implementing it, indicating interface inconsistency.

## 6. Від чого залежить?
IEntityManager, IArchetypeSystem, IMythLogger, DI/InjectAttribute, concrete Core components, System.Collections.Generic and reflection.

## 7. Хто залежить від цього?
Entity creation systems, HeroFactory/archetype systems and gameplay/application composition.

## 8. Чи є зайві залежності?
Так: logger, concrete components, direct archetype naming and reflection-based clone plumbing are all candidates for separation.

## 9. Що потребує рефакторингу?
- Replace game-specific IEntityFactory with generic entity creation contract.
- Move player/enemy/item convenience factories into Game Layer.
- Replace string archetype IDs with typed/configured identifiers.
- Give EntityManager a generic GetComponentTypes(entity) API or a proper clone/duplicate operation.
- Remove reflection from normal entity cloning.
- Separate creation from logging.
- Fix CreateEnemy contract so health/attackPower are either applied or removed.
- Align IEntityFactory with actual capabilities.

## Висновок:
☑ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Mixed. Поточна фабрика є Game Layer API поверх Framework ECS, і потребує чіткого розділення generic entity creation від game-specific factories.