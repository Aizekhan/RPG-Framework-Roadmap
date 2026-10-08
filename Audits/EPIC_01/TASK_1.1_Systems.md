# Audit — Systems

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Systems

## Name

Systems

## Structure observed

Systems is split into:
- AI
- Core
- Gameplay
- Groups
- Heroes
- Lobby
- Phase

The key distinction is between the generic system infrastructure in Core/Groups and the concrete gameplay systems in the other areas.

## 1. Is it needed by any RPG?

Yes.

Every ECS/RPG framework needs a way to execute systems, control lifecycle, ordering, scheduling, grouping and update phases. Concrete gameplay systems are also common in RPGs, but their exact behavior is game-specific.

## 2. Does it depend on a specific game?

Partially, and in several places strongly.

Framework-oriented infrastructure includes SystemRegistry, SystemGroup, lifecycle/update contracts, priorities and scheduling concepts.

Game-specific systems include:
- HeroSystem
- LobbySystem
- HeroSelectionSystem
- SimpleLobbyAI
- PhaseSystem with MythHunter's concrete Rune/Planning/Active/Freeze phases
- EntitySpawnSystem using concrete MythHunter entity factory/archetypes

SystemRegistry also currently knows about GamePhase and phase-filtering, pulling game-phase concepts into system infrastructure.

## 3. Can it be reused without changes?

Partially.

The generic execution infrastructure is reusable in principle.

The current implementations cannot all be reused unchanged because:
- phase handling is tied to MythHunter GamePhase
- gameplay systems reference concrete MythHunter components/services/events
- lobby/hero/AI systems are game-specific
- the parallel registry mixes scheduling, dependency metadata, phase filtering and execution concerns.

## 4. Framework or Game Layer?

Shared abstraction / mixed.

Framework candidates:
- ISystemRegistry
- SystemRegistry concept
- SystemGroup
- execution priorities
- lifecycle management
- scheduling/parallel execution abstractions

Game Layer:
- Heroes
- Lobby
- gameplay spawning
- concrete AI
- concrete PhaseSystem
- MythHunter-specific phase and gameplay logic

## 5. Are there architectural problems?

- System infrastructure is coupled to game-phase concepts.
- SystemRegistry has multiple responsibilities: registration, initialization, ordering, phase filtering, event subscription and DI-container exposure.
- ParallelSystemRegistry adds parallel scheduling, dependency graphs and execution policy on top of SystemRegistry, making it a large responsibility boundary.
- SystemGroup is generic, but PhaseSystemGroup directly depends on IPhaseProvider and concrete GamePhase compatibility APIs.
- TimerSystem is generic at first glance, but contains Lobby/Phase/Combat convenience methods and therefore mixes generic timing with game semantics.
- HeroSystem mixes hero creation, persistence calls, component-cache maintenance and ECS lifecycle.
- LobbySystem is strongly application/game-specific and has broad dependencies including UI, game settings, archetypes, hero selection and timers.
- HeroSelectionSystem is directly coupled to Unity Resources and MythHunter HeroArchetypeSO data.
- Async workflows exist inside systems, creating potential lifecycle/disposal and deterministic-simulation concerns.
- Error handling/logging is embedded across system infrastructure rather than isolated by policy.

## 6. What does it depend on?

Observed dependencies include:
- MythHunter.Core.ECS
- MythHunter.Core.DI
- MythHunter.Events
- MythHunter.Events.Domain / concrete event types
- MythHunter.Utils.Logging
- MythHunter.Entities
- MythHunter.Entities.Archetypes
- MythHunter.Components
- MythHunter.Services.GameSettings
- MythHunter.UI.Core
- UnityEngine / UnityEngine.Resources
- Cysharp.Threading.Tasks
- ECS Jobs/scheduler infrastructure

## 7. Who depends on it?

Systems are consumed by:
- EcsWorld / system execution infrastructure
- DI/installers
- gameplay and application services
- UI/application integration
- entity spawning and gameplay flows
- phase-dependent systems

Exact dependency consumers must be mapped in later file-level audits and the final dependency map.

## 8. Are there unnecessary dependencies?

Yes, candidates for investigation include:
- SystemRegistry -> GamePhase
- PhaseSystemGroup -> concrete GamePhase compatibility API
- TimerSystem -> Lobby/Combat/Phase domain concepts
- LobbySystem -> UI.Core
- HeroSelectionSystem -> Unity Resources and presentation-oriented asset loading
- HeroSystem -> persistence/database operations
- system infrastructure -> concrete Event/Domain types

These may be legitimate in Game Layer systems but are not appropriate inside reusable framework infrastructure.

## 9. What needs refactoring?

No implementation changes in this audit.

Refactoring candidates:
- Separate generic System Runtime from game-specific Systems.
- Extract scheduler/execution policy from SystemRegistry.
- Extract phase gating from generic SystemRegistry.
- Split ParallelSystemRegistry into smaller scheduling/dependency responsibilities.
- Keep generic TimerSystem generic; move Lobby/Combat/Phase helpers to adapters or domain services.
- Move Hero/Lobby/AI systems to Game Layer.
- Replace Unity Resources usage in gameplay systems with an abstraction/asset provider.
- Separate persistence from HeroSystem.
- Define a generic scheduling/phase abstraction that does not require MythHunter GamePhase.
- Later audit CombatSystem specifically as a known refactoring candidate.

## Conclusion

- [ ] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: Systems is a mixed/shared structure with a reusable execution core surrounded by strongly game-specific systems. The largest architectural concern is that framework-level system infrastructure currently knows about game-specific phase and event concepts. Several systems also violate single-responsibility boundaries.
