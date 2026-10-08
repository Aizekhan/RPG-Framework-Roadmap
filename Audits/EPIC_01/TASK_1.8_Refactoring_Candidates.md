# Final Audit — Refactoring Candidates

## Purpose
Consolidated list of concrete refactoring candidates discovered during the full MythHunter audit. This is an audit output, not implementation work.

## Priority A — Architecture boundary

### 1. Core/Game → Framework separation
Candidate: Core/Game/GameFlowManager
Problem: owns concrete scenes, game states, lobby lookup, game settings and UI navigation; lives under Core while implementing application flow; starts async work from constructor.
Target: move to Game Layer/Application; keep only generic state and scene contracts in Framework; make lifecycle explicit.

### 2. Gameplay installers / composition roots
Candidate: Core/Installers/CoreInstaller and system-group installers
Problem: core installer registers concrete game systems, entities, phase provider, scene/game flow and settings; composition root mixes Framework and Game Layer.
Target: Framework installer registers only Framework services; Game composition root registers game services on top.

### 3. Framework assembly boundary
Candidate: entire Core/Event/Networking/Serialization infrastructure
Problem: no hard assembly boundary; generic code can import MythHunter domain types.
Target: separate Framework assemblies from Game Layer assemblies.

## Priority B — Duplicate / overloaded infrastructure

### 4. EventBus decomposition
Candidate: Events/EventBus, SimpleEventBus, batcher/throttler/pool
Target: minimal event bus; separate queue/scheduling, middleware, pooling, replay/debugging; remove duplicate bus.

### 5. ECS storage
Candidate: Core/ECS/EntityManager
Problem: nested dictionaries rather than structural ECS storage; AddComponent can implicitly create an entity; query APIs allocate arrays.
Target: authoritative ECS world/storage; explicit entity lifetime; reusable query paths where needed.

### 6. ComponentCache
Candidate: ComponentCacheRegistry
Problem: duplicates EntityManager indexing; depends on EventBus, phase provider and concrete component types; reflection in update loops; phase-aware caching duplicates scheduling.
Target: remove if ECS storage/query layer makes it unnecessary; otherwise isolate it as an optional performance index with explicit ownership.

### 7. Phase architecture
Candidate: Systems/Phase/PhaseSystem, SystemRegistry and PhaseSystemGroup
Problem: phase state, timing, ordering and activation overlap; concrete game phase rules are mixed with generic lifecycle.
Target: generic scheduler/lifecycle in Framework; game-specific phase policy in Game Layer.

## Priority C — Large game systems

### 8. CombatSystem
Candidate: Systems/Gameplay/Combat/CombatSystem
Problem: monolithic combat responsibilities; concrete stats/resources/events and game-specific rules; orchestration mixed with calculations.
Target: split damage resolution, resource handling, ability execution, death/state transitions and combat orchestration.

### 9. HeroSystem
Problem: creation/loading/saving/destruction, cache interaction and race/class rules are combined.
Target: split entity lifecycle from Hero rules and persistence.

## Priority D — Factory / registry decoupling

### 10. EntityFactory / archetypes
Problem: generic factory infrastructure knows concrete Hero/Enemy/Item creation; archetype/template concepts overlap.
Target: generic factory/template contracts; explicit game registrations; separate structural archetype from content template.

### 11. Serialization / networking
Problem: unstable type identity and reflection; persistence and network serialization are coupled.
Target: stable schema IDs/versioning; explicit codec registry; separate persistence serialization from network protocol serialization.

## Priority order
1. Framework/Game assembly boundary
2. Composition root split
3. Event infrastructure split
4. ECS storage decision
5. Remove or isolate ComponentCache
6. Phase architecture split
7. CombatSystem decomposition
8. Hero/factory/archetype cleanup
9. Serialization/network protocol stabilization

## Conclusion
The strongest refactoring candidates are the components that currently prevent a clean extraction of a reusable Framework: Core/Game, installers, EventBus, ECS storage, ComponentCache and phase infrastructure.
No source code was modified during this audit.

Classification:
- Framework extraction candidates: ECS, DI, events, networking, serialization, generic entity/template infrastructure.
- Game Layer refactoring candidates: GameFlowManager, combat, hero, lobby, phase and concrete installers.
- Technical Debt: duplicate caches/buses, overloaded registries, reflection hot paths and missing assembly boundaries.