# Final Audit — Dependency Map

## Layered dependency model

### Layer 0 — Framework primitives
Core contracts, ECS primitives, generic components, event contracts, serialization contracts, validation primitives.

These must depend only on language/runtime abstractions or explicitly isolated infrastructure.

### Layer 1 — Framework infrastructure
DI, EventBus implementation, system lifecycle/scheduling, template/factory infrastructure, logging abstraction, optional networking transport/protocol/security.

These depend on Layer 0 contracts and must not reference Game Layer.

### Layer 2 — Game Layer domain
Hero, lobby, phase, combat, abilities, item/content definitions, concrete entity templates, game rules and domain events.

These may depend on Framework Layers 0–1.

### Layer 3 — Presentation/Application
Game flow/bootstrap, UI presenters/views/controllers, scene composition, game-specific installers and application services.

These may depend on Game Layer + Framework.

## Important current violations
- Core/Game references concrete game flow/application behavior.
- EventBus directly references concrete domain events.
- NetworkEventBus couples event delivery to networking implementation.
- Serialization uses persistence contracts and CLR reflection as protocol identity.
- Systems/Phase and SystemRegistry both participate in phase semantics.
- Lobby/Hero/Combat systems depend on concrete MythHunter components, assets and settings.
- UI and application services depend on gameplay events/systems.
- Automatic assembly scanning creates implicit dependencies across module boundaries.

## Target dependency direction

Game Layer
  ↓
Framework contracts/infrastructure
  ↓
Runtime/platform adapters

Never:
Framework → Game Layer
Framework Core → UI
Domain → Presentation
Generic networking → Concrete game content
