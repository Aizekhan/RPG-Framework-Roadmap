# Audit — UI

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/UI

## Name

UI

## Structure observed

UI contains:
- Core
- Controllers
- Models
- Navigation
- Presenters
- Services
- ViewConfigs
- Views

The structure attempts an MVP/MV* style separation with a shared UI core, navigation and presentation layers.

## 1. Is it needed by any RPG?

A presentation/UI layer is useful for virtually any RPG, but a universal Framework should provide UI contracts and infrastructure rather than game-specific screens.

## 2. Does it depend on a specific game?

Yes, substantially.

Generic UI concepts exist, but many models, presenters and views encode MythHunter gameplay:
- GameplayUIModel / GameplayUIPresenter
- Hero and lobby models/presenters/views
- Phase/Rune events
- game-state-driven navigation
- concrete gameplay panels and controls

## 3. Can it be reused without changes?

Only the lower-level UI infrastructure can be reused with limited changes.

Good Framework candidates:
- IView / IPresenter / IModel contracts
- view IDs/configuration mechanism
- generic navigation abstractions
- generic view factory concept
- modal/navigation interfaces

Game-specific:
- concrete screens, gameplay models, presenters and controllers
- phase/rune presentation
- hero/lobby UI
- game-specific ViewIds/configs

## 4. Framework or Game Layer?

**Mixed: Framework UI infrastructure + Game Layer presentation.**

The boundary should be explicit:
Framework owns UI contracts, navigation abstractions, presentation lifecycle and optional factory/resource adapters.
Game Layer owns concrete screens, models, presenters, controllers and game-specific view configuration.

## 5. Are there architectural problems?

- UI.Core is not purely generic: UISystem directly depends on Resources, DI, logging and resource infrastructure.
- UIViewFactory directly uses Unity Resources and pool infrastructure, coupling view creation to concrete resource/pooling implementations.
- UIService is incomplete: HideScreen and IsScreenActive are placeholders.
- UIViewBase discovers ViewId by matching prefab names/types instead of explicit configuration, which is fragile.
- UISystem contains legacy registration APIs alongside the newer ViewId-based API, increasing ambiguity.
- NavigationService is tightly coupled to MythHunter Core.Game and Domain Events.
- NavigationService also owns stack, modal state, transitions and event-driven game-state behavior, making it multi-responsibility.
- Concrete gameplay presenters subscribe directly to game-domain events and GamePhase.
- GameplayUIView contains substantial gameplay-specific UI logic.
- HeroGridController is a concrete game UI controller inside the generic UI module.
- SpriteService is really a resource/cache adapter and overlaps with the Resources subsystem.
- ViewConfigs mixes generic UI configuration with MythHunter categories.

## 6. What does it depend on?

Observed:
- UnityEngine / Unity UI / TextMeshPro
- Cysharp UniTask
- MythHunter.Core.DI
- MythHunter.Core.MonoBehaviours
- MythHunter.Resources.Core / Resources.Pool
- MythHunter.Utils.Logging
- MythHunter.Events
- MythHunter.Events.Domain
- MythHunter.Core.Game
- concrete MythHunter gameplay models/events

## 7. Who depends on it?

Likely consumers:
- Core installers/bootstrapping
- gameplay/lobby systems
- game screens and presenters
- resource/pool infrastructure
- event bus
- application navigation

Exact consumer mapping belongs to later detailed UI and dependency audits.

## 8. Are there unnecessary dependencies?

Yes, several candidates:
- UI.Navigation -> Core.Game / Domain Events
- UI.Core -> concrete resource and pool implementations
- UI.Services -> Resources subsystem
- generic UI module -> concrete gameplay models/controllers
- UIViewBase's runtime name matching can be removed in favor of explicit IDs/configuration

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split generic UI infrastructure from game presentation.
- Move gameplay-specific Controllers/Models/Presenters/Views into Game Layer modules.
- Keep Framework UI contracts free of game-domain dependencies.
- Extract navigation policy from NavigationService; leave a generic navigation stack/service.
- Make ViewId assignment explicit through serialized/configured metadata rather than name matching.
- Replace placeholder UIService behavior with a real screen lifecycle/state registry.
- Introduce resource/pool adapters behind abstractions.
- Separate Sprite loading/cache from UI.
- Simplify/remove legacy registration paths after consumer audit.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: UI is **mixed**. Its lower-level contracts and navigation/presentation infrastructure are Framework candidates, while most concrete models, presenters, controllers and views are Game Layer. The current module needs a clear boundary before it can become a reusable UI subsystem.
