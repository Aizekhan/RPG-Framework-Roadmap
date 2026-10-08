# Audit — Core/Game

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/Game

## Name

Core/Game

## 1. Is it needed by any RPG?

A game/application flow layer is common in RPGs, but the concrete implementation here is not universal Framework core. A reusable Framework can provide lifecycle and application-flow abstractions, while each game defines its actual states, scenes and transitions.

## 2. Does it depend on a specific game?

Yes, strongly.

GameFlowManager directly references MythHunter-specific GameStateType, LobbySystem, GameSettings/GameMode, domain events, navigation, concrete scene names, profile flow, preload configuration and gameplay archetypes.

GameBootstrapper also assembles MythHunter-specific ECS, Systems, StateMachine, Debug and Scene infrastructure.

ActionInfo is a concrete UI/gameplay model using Unity Sprite.

## 3. Can it be reused without changes?

No.

The lifecycle/bootstrap pattern is reusable, but the current implementation encodes this project's scenes, game states, lobby, profile and loading behavior.

## 4. Framework or Game Layer?

**Game Layer / Application Layer.**

A thin generic application bootstrap contract may later belong in Framework infrastructure, but this implementation should not be treated as universal Core.

## 5. Are there architectural problems?

- GameFlowManager is a large orchestration class with scene management, state management, UI navigation, preload, game settings, events and lobby coordination.
- It hardcodes scene names such as MainMenuScene, LobbyScene, GameScene, ProfileScene and LoadingScene.
- It hardcodes MythHunter-specific GameMode and GameStateType semantics.
- It reaches into ISystemRegistry to fetch ILobbySystem, creating coupling from high-level flow to concrete gameplay systems.
- GameFlowManager depends directly on UI.Navigation and UI.Views, making Core/Game depend upward on presentation code.
- It invokes UI setup from application flow, mixing application orchestration with presentation.
- GameFlowManager starts asynchronous preload work from its constructor, which makes object creation have side effects and complicates lifecycle/testing.
- It contains arbitrary delays such as 800ms, 1500ms and 600ms as flow control.
- Some methods are async void (EnterLobby), making error/lifecycle handling harder.
- GameBootstrapper is a Unity MonoBehaviour that directly constructs DI and logger infrastructure instead of receiving a composition root abstraction.
- GameBootstrapper manually orchestrates ECS, state machine, systems, dependency injection and debug UI, making it a second composition root in addition to installers.
- GameBootstrapper Update directly ticks ECS and the state machine, coupling application entry to those implementations.
- OnDestroy only disposes ECS; shutdown of DI scopes/services is not clearly centralized here.
- GameBootstrapper contains an explicit 5-frame startup delay before Boot state.
- ActionInfo is not Core infrastructure; it is a gameplay/UI presentation model and is misplaced under Core/Game.

## 6. What does it depend on?

Observed:
- Cysharp.Threading.Tasks
- UnityEngine
- MythHunter.Core.DI
- MythHunter.Core.SceneManagement
- MythHunter.Core.ECS
- MythHunter.Systems.Core
- MythHunter.States
- MythHunter.Events / Events.Domain
- MythHunter.UI.Core / UI.Navigation / UI.Views
- MythHunter.Resources
- MythHunter.Services.GameSettings
- MythHunter.Entities.Archetypes
- MythHunter.Debug
- MythHunter.Utils.Logging

## 7. Who depends on it?

Likely consumers include startup/bootstrap scene objects, installers, game-state/navigation consumers and UI/application flows. Exact dependency graph belongs to the final audit.

## 8. Are there unnecessary dependencies?

Yes, especially:
- Core/Game -> UI
- Core/Game -> concrete LobbySystem
- Core/Game -> GameSettings GameMode
- Core/Game -> game-domain Events
- Core/Game -> concrete scene/resource configuration
- GameBootstrapper -> Debug dashboard

These are signs that Game/Application orchestration is sitting above Framework boundaries but is physically located inside Core.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Move GameFlowManager to Game/Application layer.
- Keep Core limited to framework lifecycle contracts, not concrete game flow.
- Define generic application state/scene transition interfaces if reusable behavior is needed.
- Make flow definitions/configuration data-driven instead of hardcoding scene names and delays.
- Remove direct UI dependencies from flow orchestration; publish application events or use presentation adapters.
- Split GameFlowManager into smaller responsibilities: application state flow, scene flow, loading/preload coordination.
- Make bootstrap a thin composition root that delegates to installers/lifecycle abstractions.
- Move ActionInfo to the appropriate gameplay/UI module.

## Conclusion

- [ ] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Game Layer / Application Layer**. Core/Game is not reusable Framework core in its current form; it is the MythHunter application orchestration layer and should be separated from the universal Framework foundation.
