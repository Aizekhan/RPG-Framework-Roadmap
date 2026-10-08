# EPIC 02 — Extract Framework Core

## Scope

Define the first clean Framework Core boundary before moving or rewriting implementation code.

## Framework Core candidates

### ECS contracts
- IComponent
- IEntityManager
- ISystem
- IEcsWorld

These are currently the cleanest Framework candidates: they contain generic contracts and no MythHunter domain types.

### DI contracts
- IDIContainer
- DI registration/lifetime abstractions
- generic injection contracts

DI is a Framework concern, but the current container is oversized and contains lifecycle/scope features that require separate validation before extraction.

### Event contracts
- IEvent
- IEventBus

The contracts are Framework candidates, but the current IEvent contract embeds priority/identity policy and the EventBus implementation contains additional scheduling/debug/pooling responsibilities. Extract contracts first, implementation later.

### Generic system infrastructure
- System lifecycle abstractions
- generic registry/scheduling concepts

The current ISystemRegistry is NOT clean Framework Core yet because it exposes a DI container and is coupled to category/phase concerns. It should be split before extraction.

### Serialization contracts
- ISerializable and codec/serializer contracts

These can belong to Framework infrastructure, but persistence and network serialization must remain separate.

## Explicitly excluded from Framework Core

- Core/Game
- GameBootstrapper
- concrete GameState types
- Lobby, Hero, Combat and Phase systems
- concrete domain events
- Hero archetypes/templates
- UI
- GameSettings
- concrete Resources/Game scene flow
- concrete MythHunter EntityFactory implementations
- NetworkEventBus tied to game events

## Proposed target structure

Framework.Core
  ECS
  DI
  Events.Contracts
  Systems.Contracts
  Serialization.Contracts

Framework.Infrastructure
  DI implementation
  EventBus implementation
  System scheduler/registry
  Serialization implementation
  optional logging

Game Layer
  MythHunter domain
  gameplay systems
  game flow
  content
  UI
  concrete installers

## Critical boundary rules

1. Framework Core must not reference MythHunter namespaces outside Framework namespaces.
2. Framework Core must not reference concrete UI, lobby, hero, phase or gameplay code.
3. Framework contracts must not require concrete networking.
4. Generic contracts must not discover game types through assembly scanning.
5. Composition roots may depend on both Framework and Game Layer; Framework must not depend on Game Layer.
6. Unity-specific adapters should be isolated from pure contracts where practical.

## Extraction order

1. Create Framework assembly boundaries.
2. Move pure contracts first.
3. Split ISystemRegistry responsibilities before moving its implementation.
4. Split EventBus implementation from domain events.
5. Move DI implementation after contract boundary is stable.
6. Move ECS implementation after EntityManager storage decisions are settled.
7. Validate dependency direction with compilation/tests.

## Current blockers

- no assembly boundary;
- EntityManager storage is not final ECS architecture;
- EventBus is monolithic;
- SystemRegistry mixes registry, DI access, phase/category behavior;
- DI container is overextended;
- Unity dependencies are mixed into lower-level infrastructure.

## Conclusion

Framework Core is extractable, but not as a blind folder move.

The correct extraction is **contracts first, infrastructure second, Game Layer third**.

Classification:
- Framework: ECS, DI, event, system and serialization contracts.
- Framework Infrastructure: concrete implementations after decoupling.
- Game Layer: application/game rules and content.
- Technical Debt: assembly boundary, overloaded registries/buses, ECS storage, hidden discovery dependencies.
