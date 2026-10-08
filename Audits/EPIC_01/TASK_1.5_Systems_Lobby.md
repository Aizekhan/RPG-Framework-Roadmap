# Audit — Systems/Lobby Systems

## 1. Чи потрібна будь-якій RPG?
Multiplayer RPGs can need lobby/session and character-selection systems, but these are application/game-flow concerns, not universal RPG Framework core.

## 2. Чи залежить від конкретної гри?
Так, дуже сильно.
- HeroSelectionSystem is built around HeroArchetypeSO, HeroRace/HeroClass, ManaCost and hero categories.
- LobbySystem uses LobbyStateComponent, hero selection, archetypes, game settings and game-start events.

## 3. Чи можна перевикористати без змін?
Ні.
Only generic session/ready/selection concepts can be extracted later.

## 4. Framework чи Game Layer?
Game Layer.

## 5. Чи є архітектурні проблеми?
- HeroSelectionSystem directly loads Unity Resources and concrete HeroArchetypeSO assets.
- HeroSelectionSystem contains UI-oriented HeroInfo projection and category strings.
- HeroSelectionSystem has async void Initialize, which makes lifecycle/error control weak.
- HeroSelectionSystem falls back to hardcoded test heroes in production code.
- LobbySystem is highly coupled to GameSettings, TimerSystem, ArchetypeSystem, ArchetypeTemplateRegistry, ECS entities and domain events.
- LobbySystem creates temporary ECS entities only to represent hero selections, mixing lobby/application state with gameplay entity model.
- LobbySystem owns turn/player index, selection economy, readiness, timer, game start orchestration and hero spawning.
- StartGameAsync contains an arbitrary 1000 ms delay as fake loading.
- LobbySystem creates actual heroes after GameStartedEvent, creating ordering/lifecycle ambiguity.
- LobbySystem depends on concrete MythHunter Mana and hero archetype semantics.
- LobbyStateComponent itself already mixes several concerns; LobbySystem amplifies this coupling.
- Hero selection validation is partly duplicated between HeroSelectionSystem and LobbySystem.
- ArchetypeTemplateRegistry is checked directly from lobby flow, leaking entity infrastructure into application logic.

## 6. Від чого залежить?
Core.ECS, DI, Events, Components.Lobby, Resources/UnityEngine, Archetypes, GameSettings, TimerSystem, UniTask and logging.

## 7. Хто залежить від цього?
Game flow, Lobby UI, hero selection UI/presenters and transition-to-gameplay code.

## 8. Чи є зайві залежності?
Так. UI-facing data, Unity asset loading, archetype registry access, timer mechanics and hero spawning are all embedded in lobby systems.

## 9. Що потребує рефакторингу?
- Separate generic MultiplayerSession/Lobby state from MythHunter HeroSelection gameplay.
- Move asset discovery to a resource/catalog service.
- Make HeroSelection a domain service with explicit hero catalog and selection rules.
- Separate lobby state, turn/ready state, resource budget and selection timer.
- Move hero spawning into a dedicated transition/game-start service.
- Remove arbitrary delays and test-data fallback from runtime path.
- Define explicit transaction/order for GameStart -> scene load -> entity creation -> GameStarted event.
- Stop creating temporary ECS entities for UI/application selection unless they are true gameplay entities.
- Remove direct dependency on ArchetypeTemplateRegistry from lobby logic.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer. Lobby Systems are application/domain flow, but current implementation is tightly coupled to Unity assets, ECS/archetypes, timers, settings and gameplay start orchestration.