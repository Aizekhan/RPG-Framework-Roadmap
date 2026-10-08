# EPIC 02 — Remove Cyclic Dependencies

## Scope

Identify current dependency cycles or bidirectional dependency paths that prevent a clean Framework → Game Layer direction.

## Confirmed cycles / cycle risks

### 1. Game Flow ↔ UI

Observed:
- GameFlowManager imports UI.Navigation/UI.Views.
- UI navigation and presenters import Core.Game.

This creates a bidirectional relationship between application flow and presentation.

Target:
- Game/Application publishes navigation/application intents.
- UI depends on application contracts or events.
- GameFlowManager must not know concrete UI views/navigation.

### 2. Core ECS ↔ Entities

Observed:
- Core/ECS ComponentFactory imports MythHunter.Entities.
- IComponentCacheRegistry is declared in namespace MythHunter.Entities while implementation is in Core/ECS.
- Entity/archetype code imports Core/ECS contracts.

This is a reverse dependency from Core infrastructure into the Game/Entity layer.

Target:
- move generic factory/cache contracts into Framework.ECS or Framework.Entities infrastructure;
- concrete entity implementations depend on those contracts;
- Core/ECS must not import MythHunter.Entities.

### 3. Events ↔ Game Core

Observed:
- domain GameEvents import MythHunter.Core.Game.
- Core/Game imports Events.Domain.

This creates a cycle between application flow and domain event definitions.

Target:
- domain events live in Game Layer;
- events depend only on stable value/contract types;
- Framework EventBus knows only IEvent/IEventBus, never GameEvents.

### 4. System Registry ↔ Domain Events / Phase

Observed:
- Systems.Core/SystemRegistry imports Events.Domain.
- gameplay systems import SystemRegistry and Phase;
- Core/Game also resolves systems.

Target:
- generic registry handles lifecycle/ordering only;
- game-specific systems subscribe to game events externally;
- phase policy remains Game Layer.

### 5. Networking ↔ Events

Observed:
- NetworkAuthority imports Events and NetworkEventMessage imports event contracts.
- NetworkEventBus depends on networking messages.

This is acceptable only at an explicit adapter boundary. It becomes a cycle when generic networking and generic events contain each other's concrete implementations.

Target:
- Framework EventBus remains local and transport-agnostic;
- Network adapter bridges events to network messages;
- networking protocol layer must not depend on concrete domain events.

### 6. Installers as cycle amplifier

Core installers reference UI, gameplay, entities, networking, phase and Core simultaneously.

Installers therefore connect otherwise separable modules and hide cycles behind composition.

Target:
- Framework composition root;
- Game composition root;
- explicit dependency direction;
- optional adapters registered only by Game composition.

## Cycle-removal rules

1. Framework assemblies never reference Game Layer assemblies.
2. Game Layer may reference Framework.
3. Presentation may reference Application/Game contracts, but Application must not reference Presentation implementations.
4. Domain event types belong to the layer that owns the domain meaning.
5. Generic registries must not import domain events.
6. Network transport/protocol must not import concrete game events.
7. Composition roots may reference all modules but are not dependencies of the modules they compose.

## Priority

1. GameFlowManager ↔ UI
2. Core/ECS ↔ Entities
3. Events ↔ Core/Game
4. SystemRegistry ↔ Domain/Phase
5. Networking ↔ Events
6. Installer composition

## Conclusion

MythHunter contains several direct reverse dependencies and cycle risks. The most important are not algorithmic cycles but **architectural bidirectional dependencies across layers**.

No source code was modified.

Classification:
- Technical Debt: confirmed.
- Framework boundary violation: confirmed.
- Refactoring: required before a clean standalone Framework extraction.
