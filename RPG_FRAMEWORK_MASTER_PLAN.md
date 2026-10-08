# RPG Framework Master Plan

## Main Goal

Create a universal modular RPG Framework that does not depend on any specific game and allows new RPG projects to be built quickly without rewriting core systems.

## Final Framework Scope

- ECS
- DI
- EventBus
- Networking
- Persistence
- Stats
- Resources
- Statuses
- Buffs / Debuffs
- Damage / Resistance
- Death State
- Inventory
- Equipment
- Items
- Loot
- Abilities
- Combat
- Progression
- Cooldowns
- AI interfaces
- UI / Presentation API
- Developer Tools
- Documentation
- Templates

## Core Principles

- Entity = Identity
- Component = Data
- System = Logic
- Event = Occurrence
- DI = Infrastructure
- Framework must not depend on Game Layer
- Game Layer may depend on Framework
- One module owns one responsibility
- Avoid cyclic dependencies
- Audit before refactoring
- Document architectural decisions
- Do not change implementation before the relevant audit is complete

## Audit Template

### Name

1. Is it needed by any RPG?
2. Does it depend on a specific game?
3. Can it be reused without changes?
4. Framework or Game Layer?
5. Are there architectural problems?
6. What does it depend on?
7. Who depends on it?
8. Are there unnecessary dependencies?
9. What needs refactoring?

### Conclusion

- [ ] Framework
- [ ] Game Layer
- [ ] Technical Debt

## Working Rules

1. Work strictly from top to bottom.
2. Exactly one item may be ACTIVE at any time.
3. Do not start the next item until the current ACTIVE item is completed.
4. Every completed item must have a documented conclusion.
5. Complete the audit first, refactor second.
6. Keep all architectural decisions documented.
7. Use separate audit documents for detailed findings.
8. The master plan records completion with checkboxes.
9. STATUS.md is the execution pointer and always identifies the single current ACTIVE item.
10. When an item is completed, mark its checkbox here, then advance STATUS.md to the next item in strict order.
11. When the last item of a task is completed, check the task itself and advance to the next task.
12. When the last task of an Epic is completed, mark the Epic complete and advance to the next Epic.
13. Never skip ahead, even when a later item appears easier or more important.

## Sequential Execution Model

Epic
→ Task
→ Item
→ File

At every moment there is one and only one ACTIVE Item.
All later items are LOCKED until the current item is completed.
STATUS.md must always point to the exact ACTIVE Item.

# EPIC 01 — Full MythHunter Audit

## Goal

Fully understand the current architecture and separate reusable Framework from game-specific code.

## Definition of Done

- [ ] All large modules analyzed
- [ ] Dependencies documented
- [ ] Unnecessary dependencies identified
- [ ] Technical debt identified
- [ ] Framework / Game boundary defined
- [ ] Dependency map created

### TASK 1.1 — Top-Level Structures

- [x] Core
- [x] Components
- [x] Entities
- [x] Systems
- [x] Events
- [x] Networking
- [x] Cloud
- [x] UI
- [x] Services
- [x] Utils

### TASK 1.2 — Core

- [x] ECS
- [x] DI
- [ ] Game
- [ ] Installers
- [ ] StateMachine
- [ ] SceneManagement
- [ ] Validation

### TASK 1.3 — Components

- [x] Core
- [x] Character
- [x] Combat
- [x] Movement
- [x] Lobby
- [x] Other components

### TASK 1.4 — Entities

- [ ] EntityFactory
- [ ] Archetypes
- [ ] Templates
- [ ] Serialization Registry

### TASK 1.5 — Systems

- [ ] Core Systems
- [ ] Gameplay Systems
- [ ] Hero Systems
- [ ] Lobby Systems
- [ ] Phase Systems
- [ ] System Groups

### TASK 1.6 — Events

- [x] EventBus
- [x] Event Types
- [x] Event Pipeline
- [x] Middleware
- [ ] Network Events

### TASK 1.7 — Networking

- [ ] Client
- [ ] Server
- [ ] Messages
- [ ] Serialization
- [ ] Security

### TASK 1.8 — Final Audit Report

- [ ] Framework Modules
- [ ] Game Layer Modules
- [ ] Technical Debt List
- [ ] Dependency Map
- [ ] Unnecessary Dependency List
- [ ] Refactoring Candidates

Known candidates to verify by file audit:

- [ ] Core/Game
- [ ] Gameplay Installers
- [ ] Phase Systems
- [ ] CombatSystem
- [ ] ComponentCache
- [ ] EntityManager Storage Layer

# EPIC 02 — Clean Core Boundaries

## Goal

Strictly separate the reusable Framework from the Game Layer.

- [ ] Extract Framework Core
- [ ] Extract Game Layer
- [ ] Remove cyclic dependencies
- [ ] Remove unnecessary dependencies
- [ ] Define dependency rules
- [ ] Validate all modules against the rules

# EPIC 03 — ECS Completion

## Goal

Make ECS complete, predictable and reusable.

- [ ] Entity Lifecycle
- [ ] Component Lifecycle
- [ ] Query API
- [ ] Scheduler
- [ ] System API
- [ ] Event Integration
- [ ] ECS Tests

# EPIC 04 — Infrastructure

## Goal

Make infrastructure consistent and reliable.

- [ ] DI
- [ ] EventBus
- [ ] Configuration
- [ ] Validation
- [ ] Logging
- [ ] Infrastructure Tests

# EPIC 05 — RPG Foundation

## Goal

Build the universal RPG data and runtime foundation.

- [ ] Stats
- [ ] Resources
- [ ] Runtime Statuses
- [ ] Buffs
- [ ] Debuffs
- [ ] Damage
- [ ] Resistances
- [ ] Death State

# EPIC 06 — Gameplay Modules

## Goal

Build reusable gameplay modules common to RPGs.

- [ ] Inventory
- [ ] Equipment
- [ ] Items
- [ ] Loot
- [ ] Abilities
- [ ] Combat
- [ ] Progression
- [ ] Cooldowns

# EPIC 07 — Persistence

- [ ] Save
- [ ] Load
- [ ] Serialization
- [ ] Versioning
- [ ] Migration

# EPIC 08 — Networking

- [ ] Transport
- [ ] Commands
- [ ] Replication
- [ ] Snapshots
- [ ] Security
- [ ] Networking Tests

# EPIC 09 — Tools

- [ ] Entity Inspector
- [ ] Component Inspector
- [ ] System Inspector
- [ ] Event Monitor
- [ ] Network Monitor
- [ ] Save Inspector

# EPIC 10 — Framework Release

- [ ] Documentation
- [ ] API Documentation
- [ ] Templates
- [ ] Sample RPG
- [ ] Final Cleanup
- [ ] Release Candidate
