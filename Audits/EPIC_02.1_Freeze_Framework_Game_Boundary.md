# EPIC 02.1 — Freeze Framework/Game Boundary

## Status
ACTIVE

## Purpose

Freeze the ownership boundary between the reusable RPG Framework and the concrete MythHunter Game Layer before implementation begins.

This document is the architecture baseline for all following EPIC 02 tasks.

## Core rule

The dependency direction is strictly:

Game Layer
→ Framework contracts/infrastructure
→ platform/runtime adapters

The Framework must never depend on MythHunter-specific Game Layer code.

## Framework ownership

Framework owns reusable mechanics and infrastructure that can exist without MythHunter.

### Framework Core
- ECS contracts and runtime
- entity identity/lifecycle contracts
- component contracts
- system contracts and lifecycle
- scheduler contracts
- DI contracts/runtime
- event contracts/runtime
- serialization contracts
- generic entity/template infrastructure
- logging abstraction
- validation abstraction

### Optional Framework infrastructure/modules
- resource abstraction
- resource providers
- scene-loading abstraction
- preload
- object pooling
- replay contracts/service boundary
- networking contracts/runtime
- persistence infrastructure
- developer/debug service boundaries

### Framework tooling
- dependency graph tooling
- entity/component/system inspectors
- event/network/save inspection
- framework module tooling

Tooling is editor/developer infrastructure and must not leak into runtime Framework Core.

## Game Layer ownership

MythHunter-specific ownership includes:

- Game/Application Flow
- concrete Game States and scene flow
- Lobby
- Heroes
- factions/classes/races and their rules
- MythHunter-specific phase rules
- concrete combat rules and abilities
- concrete gameplay systems
- concrete entities/archetypes/templates/content
- domain events specific to MythHunter
- game services and settings
- UI/presentation
- MythHunter-specific resource configuration/content
- MythHunter-specific authoring tools
- Game composition root

## Boundary rules

### Framework may depend on
- framework contracts
- framework infrastructure
- explicitly defined adapter interfaces
- platform-neutral abstractions

### Framework must not depend on
- MythHunter namespaces/types
- concrete hero/lobby/combat/game state types
- UI/presentation
- concrete scene names
- concrete resource keys
- MythHunter content/configuration
- Unity Editor APIs in runtime assemblies
- game-specific events
- game-specific phase enums/rules

### Game Layer may depend on
- Framework contracts
- Framework infrastructure
- optional Framework modules
- platform/Unity adapters
- concrete content/configuration

### Game Layer must not push into Framework
- MythHunter domain rules
- MythHunter content
- MythHunter UI assumptions
- game-specific type discovery
- hardcoded game identifiers

## Adapter rule

Unity-specific and provider-specific implementations belong behind Framework abstractions.

Examples:
- Unity scene loader implements generic scene-loading contract.
- Unity resource provider implements generic resource provider contract.
- Cloud provider implements cloud/service interfaces.
- Unity/editor tooling stays outside runtime Framework Core.

## Composition rule

There must be a single Game composition root for MythHunter.

Framework composition must create/configure Framework infrastructure only.

The Game composition root is responsible for:
1. selecting Framework modules,
2. registering MythHunter services/content,
3. wiring adapters,
4. starting the game runtime.

Framework code must never resolve MythHunter services implicitly.

## Known boundary corrections from audit

The following existing areas are explicitly classified for later migration:

- GameFlowManager → Game Layer
- GameBootstrapper → Game composition root
- concrete GameStates → Game Layer
- LobbySystem/HeroSystem/PhaseSystem/CombatSystem → Game Layer
- Hero archetype/content → Game Layer
- concrete MythHunter events → Game Layer
- NetworkEventBus coupling to game events → remove during networking refactor
- CoreInstaller/InstallerRegistry → split Framework composition from Game composition
- EventBus → split generic event infrastructure from game events
- ComponentCache → remove/isolate; it cannot define Framework ECS ownership
- PreloadManager concrete keys/phases → Game configuration over generic resource infrastructure
- Cloud/Auth/Analytics implementations → adapters/integration layer
- Authoring tools → Editor/Game tooling boundary

## Non-goals

This task does not:
- rewrite source code,
- choose final ECS storage,
- choose final assembly names,
- implement networking,
- optimize performance,
- create RPG gameplay modules.

Those are later EPIC 02/03+ decisions.

## Exit criteria

The Framework/Game ownership rule is considered frozen when:

- every major audited subsystem has an owner,
- no Framework ownership depends on MythHunter semantics,
- optional infrastructure is explicitly separated from Framework Core,
- adapters are recognized as boundaries,
- composition roots are separated conceptually,
- later implementation work can proceed without re-deciding ownership.

## Architectural decision

**Framework = reusable runtime/contracts/infrastructure.**

**Game Layer = MythHunter application/domain/gameplay/content/presentation.**

**Adapters = Unity/provider-specific implementations.**

**Editor/Developer Tools = separate tooling boundary.**

No source code changes are made by this task.
