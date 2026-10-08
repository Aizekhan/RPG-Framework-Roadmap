# Audit — Core/ECS

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/ECS

## Name

Core/ECS

## 1. Is it needed by any RPG?

Yes. ECS is a strong reusable foundation for a universal RPG Framework, although not every RPG must use ECS.

## 2. Does it depend on a specific game?

The basic entity/component/world interfaces are mostly game-agnostic. However, several implementation classes currently depend on MythHunter infrastructure or game concepts.

Examples:
- ComponentCacheRegistry depends on Events, DI, logging, Entities and IPhaseProvider.
- ComponentFactory depends on logging/DI and Entities namespace.
- SystemBase depends directly on EventBus and MythLogger.

## 3. Can it be reused without changes?

Only partially.

Reusable foundation:
- IComponent
- IEntityManager / EntityManager concept
- Entity identity
- IEcsWorld / EcsWorld concept
- basic component factory concept
- generic system base contracts

Current cache and optimization layers are not ready to be treated as universal ECS infrastructure.

## 4. Framework or Game Layer?

**Framework candidate with Technical Debt.**

ECS is central Framework infrastructure, but the current implementation needs to be separated from EventBus, Phase and MythHunter-specific services.

## 5. Are there architectural problems?

- EntityManager uses nested dictionaries rather than an archetype/chunk/component-storage model; this is a simple ECS implementation, not a data-oriented archetype ECS.
- GetEntitiesWith<T>() returns arrays, creating allocations on every query.
- EntityManager accepts missing entity IDs in AddComponent by silently creating the entity entry, weakening entity lifecycle invariants.
- Entity is only a thin wrapper around an int and appears redundant beside raw entity IDs.
- EcsWorld directly depends on Systems.Core.ISystemRegistry, coupling ECS world to the systems module.
- SystemBase couples ECS system infrastructure directly to EventBus and MythLogger.
- UpdateableSystemBase assumes a particular update model by implementing multiple update interfaces.
- ComponentFactory contains MythHunter-oriented logging/infrastructure and an empty default registration mechanism.
- ComponentCache duplicates component/entity storage already owned by EntityManager.
- ComponentCache supports several strategies but the implementation is complex relative to the basic ECS storage and can become a second source of truth.
- ComponentCacheRegistry is heavily coupled: phase provider, EventBus, DI, logging and concrete Components.Core defaults.
- ComponentCacheRegistry uses reflection to invoke Update on caches.
- ComponentCacheRegistry has game-phase semantics inside Core/ECS.
- EcsOptimizer is mostly a cache wrapper and contains placeholder optimization logic; it currently provides little real ECS optimization.
- No generic multi-component query API was found in the inspected core types.

## 6. What does it depend on?

Observed dependencies:
- System collections / reflection
- MythHunter.Core.DI
- MythHunter.Events
- MythHunter.Events.Domain
- MythHunter.Entities
- MythHunter.Systems.Core
- MythHunter.Utils.Logging
- IPhaseProvider and phase-related infrastructure

The foundational ECS interfaces themselves have very few dependencies, which is good.

## 7. Who depends on it?

Likely consumers include:
- gameplay systems
- entity factories
- component definitions
- hero/combat/movement systems
- DI installers
- event-driven system infrastructure

Exact consumer mapping will be refined in later detailed audits.

## 8. Are there unnecessary dependencies?

Yes.

Main candidates:
- ECS -> EventBus
- ECS -> PhaseProvider / domain phase concepts
- ECS -> MythHunter logging implementation
- EcsWorld -> Systems.Core registry
- ComponentCacheRegistry -> concrete Components.Core types
- ComponentFactory -> Entities namespace

These should be reduced so the ECS kernel can exist independently from application/game modules.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Define a small independent ECS kernel: Entity ID, component storage, queries, world lifecycle.
- Move EventBus integration to a separate system/integration layer.
- Move phase-aware caching out of Core/ECS.
- Decide whether Entity wrapper is needed or remove it in favor of a consistent EntityId value type.
- Design a real query API before introducing optimization layers.
- Reassess ComponentCache as a derived index rather than duplicate storage.
- Remove placeholder EcsOptimizer or replace it with measured optimization tooling later.
- Separate system scheduling/registry from ECS world so Core/ECS does not depend upward on Systems.

## Conclusion

- [x] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: **Framework candidate**. ECS is one of the strongest universal parts of MythHunter, but the current Core/ECS implementation is a simple dictionary ECS with significant coupling to Events, Systems, Phase and logging. The kernel should be isolated before further ECS expansion.
