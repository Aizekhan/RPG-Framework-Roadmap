# Audit — Core/StateMachine

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/StateMachine

## Name

Core/StateMachine

## 1. Is it needed by any RPG?

Yes. A state machine is broadly reusable Framework infrastructure for lifecycle and application/game modes.

## 2. Does it depend on a specific game?

The generic state abstraction is reusable, but the current GameStateMachine and concrete states are strongly MythHunter-specific.

GameStateType contains concrete states such as Boot, MainMenu, Profile, Loading, Lobby, Gameplay, Pause and GameOver. Concrete states directly reference MythHunter UI, scenes, systems, events, game flow and settings.

## 3. Can it be reused without changes?

Only the generic state abstraction can be reused with limited changes.

The current GameStateMachine is an application-specific implementation and should not be the universal Framework state machine.

## 4. Framework or Game Layer?

**Mixed: Framework state-machine primitives + Game/Application Layer state implementation.**

## 5. Are there architectural problems?

- IState<T> is generic and reusable, but is placed beside a game-specific state implementation.
- IGameStateMachine is tied directly to GameStateType, so the interface cannot be reused with another game's state enum.
- GameStateMachine manually constructs every concrete MythHunter state, defeating DI composition and making it impossible to reuse as a generic state machine.
- State transitions are triggered asynchronously with UniTask fire-and-forget behavior.
- ChangeState sets _currentState before async Enter completes, so another transition can occur while the previous Enter is still running.
- No explicit transition serialization/cancellation mechanism was observed.
- State context is stored as mutable object, creating runtime casts and weak contracts.
- GameStateType includes NavigationBehavior metadata, coupling state definitions to UI navigation policy.
- Concrete states perform scene loading, UI navigation, system initialization and event publication, causing states to become orchestration classes.
- LobbyState is especially complex and contains duplicate/dead-looking enter flow logic plus arbitrary frame delays and semaphore protection.
- GameplayState, MainMenuState and ProfileState directly depend on scene names and UI navigation.
- BootState directly calls GameFlowManager.
- StateMachine currently overlaps responsibilities with GameFlowManager and SceneManagement.

## 6. What does it depend on?

Observed:
- Cysharp.Threading.Tasks
- MythHunter.Core.DI
- MythHunter.Utils.Logging
- MythHunter.Events and domain events
- MythHunter.UI.Navigation / UI.Core
- MythHunter.Core.SceneManagement
- MythHunter.Systems.Core
- MythHunter.Services.GameSettings
- MythHunter.States

Generic IState itself has almost no dependencies.

## 7. Who depends on it?

- GameBootstrapper updates and initializes IGameStateMachine.
- GameFlowManager changes game state.
- NavigationService reads GameStateType navigation attributes.
- Concrete application/game states coordinate UI, scene and systems.

## 8. Are there unnecessary dependencies?

Yes.

- StateMachine -> UI navigation
- StateMachine -> SceneManagement
- StateMachine -> EventBus/domain events
- StateMachine -> GameFlowManager
- StateMachine -> Systems
- StateMachine -> GameSettings

These dependencies belong in application-specific state implementations or transition coordinators, not the generic state-machine kernel.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Extract a generic StateMachine<TStateId> and generic IState<TStateId>.
- Make state registration/injection external instead of hardcoding concrete states.
- Separate transition orchestration from state behavior.
- Introduce controlled async transitions with cancellation/serialization.
- Replace object state context with typed transition context where needed.
- Remove UI navigation metadata from generic state identifiers.
- Move concrete Boot/MainMenu/Lobby/Gameplay/Profile states to Game/Application layer.
- Clarify boundary with GameFlowManager so one owns flow orchestration and the other owns state lifecycle, not both.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Mixed**. The state-machine abstraction is a strong Framework candidate, but the current implementation is mostly a MythHunter application state machine and is tightly coupled to UI, scenes, systems, events and GameFlowManager.
