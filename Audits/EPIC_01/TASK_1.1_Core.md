# Audit — Core

Source project: `Aizekhan/MythHunter`  
Branch: `dev`  
Path: `Assets/_MythHunter/Code/Core`

## Name

Core

## 1. Is it needed by any RPG?

**Yes.**

The Core area contains foundational mechanisms needed by a broad class of RPGs, including ECS, dependency injection, state management, scene management, installers, validation, and Unity integration.

However, the current `Core` directory is not uniformly universal. Some parts are clearly framework-level, while others currently contain game-specific or presentation-specific responsibilities.

## 2. Does it depend on a specific game?

**Partially.**

Evidence from the current repository shows Core code depending on game/UI/components/events in places that should be investigated during the detailed Core audit. Examples include:

- `Core/Installers/UIInstaller.cs` imports `MythHunter.UI.*`.
- `Core/Validation/DIValidator.cs` imports `MythHunter.Core.Game`, `MythHunter.Core.MonoBehaviours`, and utilities.
- `Core/MonoBehaviours/MovementView.cs` imports `MythHunter.Components.Movement` and domain events.
- `Core/StateMachine/ProfileState.cs` imports `MythHunter.Core.Game` and `MythHunter.Events.Domain`.

This indicates that the directory currently mixes reusable infrastructure with game/application integration.

## 3. Can it be reused without changes?

**Partially.**

The conceptual areas ECS, DI, state machine, scene management, and validation are reusable. The current implementation cannot be assumed to be reusable without changes because some files reference MythHunter-specific gameplay, UI, components, or application state.

## 4. Framework or Game Layer?

**Shared abstraction / mixed.**

Core should become the primary Framework foundation, but the current directory contains mixed responsibilities. The detailed audit must separate true Framework Core from Game Layer integration.

## 5. Are there architectural problems?

- Core is not a clean architectural boundary yet; it contains both reusable infrastructure and game-specific/application integration.
- Some Core files depend upward on UI, gameplay components, domain events, or game-state concepts.
- `Core` is too broad as a single boundary and contains Unity/presentation-facing code alongside framework mechanisms.
- Some dependencies appear to violate the intended direction: Framework Core should not depend on concrete Game Layer modules.

## 6. What does it depend on?

Observed dependencies include:

- System/.NET
- Cysharp UniTask
- UnityEngine
- MythHunter.Events
- MythHunter.Components
- MythHunter.UI
- MythHunter.Utils
- MythHunter.Core.Game
- MythHunter.Core.DI

Exact dependency graph must be established during TASK 1.2 file-level audit.

## 7. Who depends on it?

At the top level, Core functionality is foundational and is expected to be consumed by:

- Components/systems and gameplay code
- Event infrastructure
- Entities/ECS-related code
- Installers and services
- UI/presentation integration
- Networking and application infrastructure

Exact consumers must be mapped during TASK 1.2 and the final dependency-map task.

## 8. Are there unnecessary dependencies?

**Likely yes.**

Confirmed candidates for investigation:

- Core → UI
- Core → gameplay Components
- Core → domain-specific Events
- Core → game-specific state

These may be valid integration dependencies in an application layer, but they are suspicious inside universal Framework Core.

## 9. What needs refactoring?

No code changes in this audit.

Refactoring candidates:

- Split universal Core from application/game integration.
- Remove upward dependencies from universal Core into Game Layer.
- Revisit `Core/Installers` boundary.
- Revisit `Core/MonoBehaviours` placement.
- Revisit `Core/Game` and StateMachine game-specific state types.
- Define explicit Framework Core dependency rules.
- Build a file-level dependency map before moving or rewriting code.

## Conclusion

- [ ] Framework
- [ ] Game Layer
- [x] Technical Debt

**Classification:** Core is a **mixed/shared structure** that is intended to become Framework, but currently contains Technical Debt caused by responsibility mixing and upward dependencies.
