# RPG Framework Master Plan

## Master Objective

Transform MythHunter into a reusable, modular RPG Framework plus a concrete Game Layer, then build universal RPG modules on top of that stable foundation.

## Working Rules

1. Work strictly top to bottom.
2. Exactly one item may be ACTIVE.
3. Audit first, architecture second, implementation third.
4. Do not mix external reference documents with source-code evidence.
5. Every completed item has a documented conclusion.
6. Master Plan is the execution order; STATUS.md is the exact pointer.
7. Re-plan after major architectural discoveries when the existing future plan is no longer precise enough.
8. Do not optimize or generalize systems before their ownership and contracts are stable.

# EPIC 01 — Full MythHunter Audit

## Goal
Understand the real source architecture and define the Framework/Game boundary.

Status: COMPLETE

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
- [x] Game
- [x] Installers
- [x] StateMachine
- [x] SceneManagement
- [x] Validation

### TASK 1.3 — Components
- [x] Core
- [x] Character
- [x] Combat
- [x] Movement
- [x] Lobby
- [x] Other components

### TASK 1.4 — Entities
- [x] EntityFactory
- [x] Archetypes
- [x] Templates
- [x] Serialization Registry

### TASK 1.5 — Systems
- [x] Core Systems
- [x] Gameplay Systems
- [x] Hero Systems
- [x] Lobby Systems
- [x] Phase Systems
- [x] System Groups

### TASK 1.6 — Events
- [x] EventBus
- [x] Event Types
- [x] Event Pipeline
- [x] Middleware
- [x] Network Events

### TASK 1.7 — Networking
- [x] Client
- [x] Server
- [x] Messages
- [x] Serialization
- [x] Security

### TASK 1.8 — Final Audit Report
- [x] Framework Modules
- [x] Game Layer Modules
- [x] Technical Debt List
- [x] Dependency Map
- [x] Unnecessary Dependency List
- [x] Refactoring Candidates
- [x] Core/Game
- [x] Gameplay Installers
- [x] Phase Systems
- [x] CombatSystem
- [x] ComponentCache
- [x] EntityManager Storage Layer

# EPIC 01.5 — Audit Completeness Check

## Goal
Verify that no major top-level system outside the original audit matrix was omitted from the architecture model.

- [x] Replay
- [x] Resources / Preload / Pool
- [x] Cloud / External Services
- [x] Debug / Developer Tools
- [x] Authoring / Editor Tooling
- [x] Utils / Logging / Validation
- [x] Final completeness verdict

# EPIC 02 — Architecture Rebuild

## Goal
Rebuild the roadmap from the actual audit findings before implementation begins.

- [x] Freeze Framework/Game boundary
- [x] Define target module map
- [x] Define assembly/package boundaries
- [ ] Define allowed dependency direction
- [ ] Define forbidden dependencies
- [ ] Define composition-root strategy
- [ ] Define migration strategy
- [ ] Final architecture baseline

# EPIC 03 — Framework Extraction

## Goal
Physically extract the reusable Framework without changing gameplay behavior unnecessarily.

- [ ] Framework contracts
- [ ] ECS contracts and runtime
- [ ] DI contracts and runtime
- [ ] Event contracts and runtime
- [ ] System lifecycle/scheduler
- [ ] Serialization contracts
- [ ] Generic entity/template infrastructure
- [ ] Logging abstraction
- [ ] Framework composition root
- [ ] Framework build/test validation

# EPIC 04 — Game Layer Extraction

## Goal
Move MythHunter-specific application, domain, gameplay and presentation ownership into a clean Game Layer.

- [ ] Application/Game Flow
- [ ] Concrete states and scene flow
- [ ] Lobby
- [ ] Heroes
- [ ] Phase rules
- [ ] Combat and abilities
- [ ] Entity content/archetypes
- [ ] Domain events
- [ ] Game services/settings
- [ ] UI/presentation
- [ ] Game composition root

# EPIC 05 — Dependency Cleanup

## Goal
Remove architectural coupling revealed by the audit.

- [ ] Remove cyclic dependencies
- [ ] Remove unnecessary dependencies
- [ ] Remove duplicate EventBus
- [ ] Remove or isolate ComponentCache
- [ ] Separate persistence vs network serialization
- [ ] Replace implicit assembly scanning
- [ ] Remove reflection/dynamic hot paths
- [ ] Stabilize async lifecycle/cancellation
- [ ] Validate dependency graph

# EPIC 06 — ECS Completion

## Goal
Make ECS predictable, reusable and suitable as Framework infrastructure.

- [ ] Entity lifecycle
- [ ] Component lifecycle
- [ ] Entity identity rules
- [ ] Storage architecture decision
- [ ] Archetype/query model
- [ ] Query API
- [ ] Scheduler integration
- [ ] System API
- [ ] Event integration
- [ ] ECS tests
- [ ] Performance validation

# EPIC 07 — Infrastructure Hardening

## Goal
Make generic infrastructure reliable enough to serve multiple RPGs.

- [ ] DI lifecycle/scopes
- [ ] Event ordering/error semantics
- [ ] Middleware
- [ ] Logging
- [ ] Validation
- [ ] Configuration
- [ ] Lifecycle/cancellation
- [ ] Infrastructure tests

# EPIC 08 — Resource & Runtime Services

## Goal
Extract optional reusable runtime services discovered in the source project.

- [ ] Resource abstraction
- [ ] Resource providers
- [ ] Scene loading abstraction
- [ ] Preload system
- [ ] Object pooling abstraction
- [ ] Optional Unity adapters
- [ ] Cloud integration boundary
- [ ] Developer/debug service boundaries
- [ ] Replay service boundary

# EPIC 09 — RPG Foundation

## Goal
Build universal RPG domain modules.

- [ ] Stats
- [ ] Resources
- [ ] Statuses
- [ ] Buffs
- [ ] Debuffs
- [ ] Damage
- [ ] Resistances
- [ ] Death state

# EPIC 10 — Gameplay Modules

## Goal
Build reusable RPG gameplay modules.

- [ ] Inventory
- [ ] Equipment
- [ ] Items
- [ ] Loot
- [ ] Abilities
- [ ] Combat
- [ ] Progression
- [ ] Cooldowns

# EPIC 11 — Persistence

## Goal
Build universal persistence.

- [ ] Save
- [ ] Load
- [ ] Serialization
- [ ] Schema/versioning
- [ ] Migration
- [ ] Persistence tests

# EPIC 12 — Networking

## Goal
Build networking as an independent optional Framework module.

- [ ] Transport
- [ ] Session lifecycle
- [ ] Commands
- [ ] Replication
- [ ] Snapshots
- [ ] Stable network IDs
- [ ] Authentication
- [ ] Replay protection
- [ ] Security tests

# EPIC 13 — Tools

## Goal
Build tooling around the Framework.

- [ ] Entity Inspector
- [ ] Component Inspector
- [ ] System Inspector
- [ ] Event Monitor
- [ ] Network Monitor
- [ ] Save Inspector
- [ ] Dependency Graph
- [ ] Framework Module Manager

# EPIC 14 — Framework Release

## Goal
Make the Framework usable to create new RPG projects.

- [ ] Documentation
- [ ] API documentation
- [ ] Project template
- [ ] Module templates
- [ ] Sample RPG
- [ ] Automated validation
- [ ] Final cleanup
- [ ] Release candidate
