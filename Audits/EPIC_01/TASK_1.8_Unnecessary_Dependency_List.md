# Final Audit — Unnecessary Dependency List

## Purpose

Identify dependencies that are not required by a module's core responsibility and therefore block extraction into a reusable RPG Framework.

## Confirmed unnecessary / boundary-breaking dependencies

- Core/Game -> LobbySystem, Hero/archetype data, concrete UI/game scenes. Application flow does not belong in Framework Core.
- Core/ECS -> Events implementation and concrete game/domain knowledge. ECS primitives should not require a concrete event bus or domain events.
- Core/ECS -> Entities namespace from generic factories/cache. This creates a reverse dependency from infrastructure toward concrete entity model.
- EventBus -> Concrete MythHunter domain events. Generic event infrastructure must not know game content.
- EventBus -> UnityEditor. Editor tooling must not leak into runtime infrastructure.
- NetworkEventBus -> Concrete domain events and MythHunter assembly scanning. Networking infrastructure should consume registered contracts.
- Serialization -> Concrete serializers registered in generic infrastructure. Codec selection should be explicit and modular.
- Systems.Core/SystemRegistry -> Phase policy and game-specific event semantics. System lifecycle infrastructure should not own game rules.
- Systems.Phase -> Generic system registry responsibilities. Phase flow and system scheduling are separate concerns.
- LobbySystem -> UI.Core. Gameplay/lobby orchestration should not depend directly on presentation.
- HeroSystem -> Concrete hero data, race/class rules and persistence details. Generic infrastructure should not contain MythHunter rules.
- ComponentCache -> EntityManager plus event/domain knowledge. It duplicates ECS responsibility and couples storage to application behavior.
- EntityFactory -> Concrete Hero/Enemy/Item creation rules. A generic factory should receive registrations/providers.
- Domain events -> Concrete components/services/entities. Event contracts should depend on stable DTO/value contracts, not large domain graphs.
- UI presenters/views -> Core.Game and concrete gameplay systems where avoidable. Presentation should consume application contracts/events.
- Core installers -> Gameplay systems/services/entities. Framework composition must not hardcode game content.

## Dependency patterns to eliminate

### Framework -> Game Layer
Core infrastructure, event infrastructure, and generic ECS helpers must not import concrete lobby, hero, phase, gameplay, or MythHunter entity types.

Action: invert through interfaces, contracts, and explicit registries.

### Infrastructure -> Presentation
Gameplay and core services should not know concrete views/controllers.

Action: use events or application-facing interfaces.

### Generic module -> Concrete content discovery
Assembly scans filtered by MythHunter, reflection-based discovery of game types, and hardcoded serializer/entity registrations create hidden coupling.

Action: explicit registration at the composition root.

### Duplicate responsibility dependencies
ComponentCache duplicates EntityManager storage; PhaseSystem duplicates SystemRegistry phase semantics; SimpleEventBus duplicates EventBus.

Action: remove duplicate infrastructure or make one implementation authoritative.

## Classification

Framework candidates require decoupling before extraction:
- ECS
- DI
- Event contracts and bus
- Generic systems infrastructure
- Serialization contracts
- Network transport/protocol
- Generic components
- Generic entity/template/factory infrastructure

Game Layer:
- Hero, Lobby, Phase and Combat systems
- Concrete domain events
- Hero archetypes/templates
- Game services/settings
- UI
- Game flow/bootstrap

## Conclusion

The primary unnecessary dependencies are reverse dependencies from generic infrastructure into MythHunter-specific code, presentation dependencies inside gameplay, and duplicate infrastructure dependencies.

These are the main blockers to extracting a standalone RPG Framework.

- Framework: dependencies must be removed before extraction.
- Game Layer: concrete dependencies are expected.
- Technical Debt: duplicate and reverse dependencies require refactoring.
