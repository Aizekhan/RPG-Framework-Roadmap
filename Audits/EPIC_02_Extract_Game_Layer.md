# EPIC 02 — Extract Game Layer

## Scope

Define which current MythHunter modules constitute the Game Layer and how they should sit above the reusable Framework.

## Game Layer modules

### Application / Game Flow
- Core/Game/GameFlowManager
- GameBootstrapper and concrete GameState implementations

These coordinate MythHunter scenes, modes, lobby flow, profile flow, preload and gameplay transitions.

### Gameplay Systems
- Hero systems
- Lobby systems
- Phase systems
- Combat systems
- Ability systems
- AI and spawning
- Movement/gameplay-specific systems

These contain rules and behavior specific to the game.

### Domain and Content
- concrete domain events
- Hero entities and services
- concrete components such as HeroIdentity, Inventory, SocialSkills and game-specific stats
- concrete archetypes/templates
- game settings
- prefab/resource definitions

### Presentation
- UI views
- presenters
- navigation
- UI services/controllers

### Composition
- game-specific installers
- Game Layer composition root

## Framework dependencies allowed

Game Layer may depend on:
- ECS contracts/infrastructure
- DI contracts/infrastructure
- generic event contracts/bus
- generic system lifecycle/scheduling
- serialization contracts
- networking contracts
- resource abstractions where needed

Game Layer must not be required by Framework.

## Target structure

Framework
  ↓
Game Layer
  ├─ Application
  ├─ Domain
  ├─ Gameplay
  ├─ Content
  ├─ Presentation
  └─ Composition

## Important boundary decisions

1. Moving code into Game Layer does not mean rewriting it immediately.
2. Existing domain behavior can remain initially; the goal is to establish ownership and dependency direction.
3. UI belongs above gameplay/application contracts rather than inside Framework.
4. Game-specific events remain in Game Layer even when delivered through Framework EventBus.
5. Concrete archetypes/templates are Game Layer content; generic template infrastructure remains Framework.
6. Networking adapters may live in Game Layer when they encode MythHunter-specific messages or authority rules.

## Conclusion

The Game Layer is already present in the source project, but it is distributed through folders named Core, Systems, Events, Entities, Services and UI.

The task is therefore primarily **boundary extraction and ownership correction**, not wholesale recreation of gameplay.

Classification:
- Game Layer: application flow, gameplay, domain, content, presentation and composition.
- Framework dependencies: generic contracts/infrastructure consumed by Game Layer.
- Technical Debt: misplaced Game code under Core, mixed installers, concrete domain events inside generic Events, and cross-layer dependencies.
