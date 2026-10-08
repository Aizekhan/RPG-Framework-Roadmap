# Final Audit — Game Layer Modules

## Game Layer boundary

### Clearly Game Layer
- Core/Game: GameBootstrapper, GameFlowManager, concrete game state/flow orchestration, game action semantics.
- Systems/Heroes: HeroSystem, Race/Class bonus rules, hero lifecycle and persistence behavior.
- Systems/Lobby: hero selection, player slots, mana/readiness/timers, lobby orchestration.
- Systems/Phase: MythHunter-specific phase enum/order/durations and phase lifecycle.
- Gameplay Systems: combat rules, abilities, runes, AI behavior, spawning rules and other game-specific gameplay.
- Domain Events: lobby, hero, phase, scene and game-specific events.
- UI: presenters, views, controllers, navigation and concrete game screens.
- Services/Heroes/Prefabs/GameSettings: content and application-specific services/assets.
- Concrete Archetypes/Templates/EntityFactory behavior for player/enemy/item definitions.

### Mixed / adapter layers
- Installers: framework composition + game registrations.
- SceneManagement: generic scene abstraction + concrete MythHunter scene flow.
- Validation: generic validation primitives + MythHunter-specific validators.
- Serialization: generic serialization infrastructure + game/domain schemas.
- Networking events: generic network bridge idea + game-specific event types.
- Cloud/UI infrastructure: reusable interfaces with application-specific implementations.

### Boundary rule
Game Layer may depend on Framework. Framework must never depend on these game-specific modules.

## Main Game Layer risks
- Business rules are embedded directly in systems and components.
- Concrete game content is registered through global/automatic discovery.
- Application flow, phase flow, lobby flow and hero flow have overlapping responsibilities.
- UI and gameplay communicate through broad event dependencies.
- Game-specific concepts leak into Core infrastructure via phases, events, logging, installers and registries.

## Extraction rule
Game Layer should own:
1. Game rules.
2. Content definitions.
3. Concrete entities/archetypes.
4. Concrete screens and presenters.
5. Game-specific compositions/installers.
6. Game-specific networking messages/events.

Framework should expose contracts and reusable runtime infrastructure only.
