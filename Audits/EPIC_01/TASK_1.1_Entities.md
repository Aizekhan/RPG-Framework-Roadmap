# Audit — Entities

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Entities

## Name

Entities

## Structure observed

Entities contains:
- Archetypes
- EntityFactory.cs
- Heroes
- IComponentSerializerRegistry.cs
- IEntityFactory.cs

Important observation: the Archetypes directory appears to implement entity creation/templates/archetype metadata. It is not evidence that the ECS storage itself is archetype-based. EntityManager is currently dictionary-based.

## 1. Is it needed by any RPG?

Yes, partially.

Generic RPG frameworks need entity creation, factories, reusable entity templates/archetypes, and serialization boundaries. The current Hero-specific layer is not required by every RPG.

## 2. Does it depend on a specific game?

Partially / Yes.

EntityFactory exposes CreatePlayerCharacter, CreateEnemy and CreateItem and directly references MythHunter components. HeroFactory/HeroArchetypeRegistry are explicitly game-domain specific.

## 3. Can it be reused without changes?

Partially.

Reusable candidates:
- generic entity factory abstraction
- archetype/template mechanisms
- serializer registry abstraction

Game-specific and non-reusable without changes:
- hardcoded player/enemy/item creation
- Hero-specific classes
- concrete MythHunter component references
- current cloning implementation

## 4. Framework or Game Layer?

Shared abstraction / mixed.

Generic entity lifecycle, factory contracts and template/archetype infrastructure are Framework candidates. Hero-specific creation/data belongs to Game Layer.

## 5. Are there architectural problems?

- EntityFactory mixes generic entity creation with concrete MythHunter gameplay concepts.
- IEntityFactory exposes MythHunter-specific creation operations instead of a generic factory contract.
- CloneEntity relies on reflection and a manually maintained component-type list.
- GetAllComponentTypes explicitly says it is a simplified implementation and that EntityManager should provide component types.
- Hero-specific entity/data code is colocated with framework-oriented entity infrastructure.
- IComponentSerializerRegistry lives under Entities but uses a serialization namespace, indicating unclear module ownership.
- There is ambiguity between entity templates/archetypes and ECS storage archetypes.

## 6. What does it depend on?

Observed dependencies:
- MythHunter.Core.ECS
- MythHunter.Core.DI
- MythHunter.Entities.Archetypes
- MythHunter.Utils.Logging
- concrete MythHunter Components
- Hero/domain models
- System/.NET collections and reflection

## 7. Who depends on it?

Likely consumers:
- entity creation/gameplay systems
- DI/installers
- hero/lobby systems
- persistence/serialization
- networking/entity replication
- entity template/archetype infrastructure

Exact consumers belong to later file-level audits and dependency mapping.

## 8. Are there unnecessary dependencies?

Likely yes.

Candidates:
- entity factory to concrete MythHunter Components
- entity infrastructure to hero-specific code
- factory to logging
- factory to concrete archetype implementation rather than a generic abstraction

These require file-level verification.

## 9. What needs refactoring?

No code changes in this audit.

Candidates:
- split universal entity factory/interfaces from MythHunter-specific creation
- replace hardcoded creation methods with generic template/archetype definitions
- remove hardcoded component enumeration from cloning
- provide a real EntityManager component-type/query API
- separate Hero entities/data into Game Layer
- move serialization contracts to a clear persistence/serialization boundary
- clearly distinguish entity templates/archetypes from ECS storage archetypes

## Conclusion

- [ ] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: Entities are a mixed/shared structure. Generic entity identity, factory and template/archetype abstractions are Framework candidates; current hero-specific and MythHunter-specific creation logic belongs to the Game Layer. Technical debt exists in cloning, API specificity and module boundaries.
