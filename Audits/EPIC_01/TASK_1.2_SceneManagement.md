# Audit — Core/SceneManagement

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/SceneManagement

## Name

Core/SceneManagement

## 1. Is it needed by any RPG?

Scene/application loading infrastructure is useful for Unity RPGs, although not every Framework needs scenes if it targets another runtime or a single-scene architecture.

## 2. Does it depend on a specific game?

The basic loading operations are generic Unity infrastructure. The current dispatcher also contains MythHunter-specific gameplay data and assumes a Unity SceneManager-based application.

## 3. Can it be reused without changes?

Partially.

ISceneDispatcher and basic SceneLoader operations are reusable within Unity. `LoadGameSceneAsync(string[] selectedHeroArchetypes)` and the static scene data dictionary are application-specific patterns.

## 4. Framework or Game Layer?

**Mixed, leaning Framework infrastructure with Game-specific additions.**

## 5. Are there architectural problems?

- SceneDispatcher and SceneLoader overlap responsibilities but the layering is reasonable in principle.
- SceneDispatcher exposes a game-specific `LoadGameSceneAsync` method, which does not belong in generic scene infrastructure.
- Static Dictionary<string, object> scene data is global mutable state, weakly typed and lifecycle-unbounded.
- Scene data keys are arbitrary strings, making contracts fragile.
- SceneLoader is placed under `Resources.SceneManagement` while its abstraction is under `Core.SceneManagement`, creating unclear module ownership.
- SceneLoader directly depends on UnityEngine.SceneManagement, so it is Unity-specific infrastructure and should not leak into a pure Framework core if portability is a goal.
- `LoadSceneAsync` has a loading-screen flag but the actual loading UI integration is a TODO.
- No cancellation, timeout, progress reporting, duplicate-load policy or explicit load failure abstraction is visible.
- Direct string scene names are used throughout application code.
- No explicit scene lifecycle event contract was observed in these core files.

## 6. What does it depend on?

Observed:
- Cysharp.Threading.Tasks
- UnityEngine
- UnityEngine.SceneManagement
- MythHunter.Utils.Logging
- MythHunter.Core.DI
- MythHunter.Resources.SceneManagement

The generic interface itself only depends on UniTask.

## 7. Who depends on it?

- GameFlowManager
- concrete StateMachine states
- GameBootstrapper/services
- resource/application flow

Likely UI/navigation and preload flows also depend on scene transitions indirectly.

## 8. Are there unnecessary dependencies?

Yes.

- Core SceneManagement -> Resources namespace for SceneLoader placement is a boundary smell.
- SceneDispatcher -> selected hero archetypes is game-specific.
- Static object dictionary is unnecessary global state and should be replaced by an explicit transition/session context.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Keep a generic Unity scene loader/dispatcher focused only on scene lifecycle.
- Move `LoadGameSceneAsync` into Game/Application flow.
- Replace string keys/objects with typed scene transition context where necessary.
- Introduce explicit progress, cancellation and error semantics.
- Define scene lifecycle events if other modules need them.
- Clarify ownership so SceneManagement does not depend upward on Resources naming.
- Consider scene IDs/configuration rather than raw strings for application code.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Mixed**. The low-level scene loading mechanism is valid Framework infrastructure for Unity, while the dispatcher currently contains MythHunter-specific game-flow concerns that should move to Game/Application code.
