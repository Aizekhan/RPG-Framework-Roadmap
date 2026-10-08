# Final Audit — Framework Modules

## Purpose
Зведений аудит визначає, які частини MythHunter можуть стати універсальними Framework modules.

## Framework core candidates
- ECS: Entity/component/query/lifecycle infrastructure.
- DI: dependency registration/resolution infrastructure.
- EventBus: typed event contract + dispatch infrastructure after separation of scheduling/middleware/domain events.
- Generic Components: identity, value, position/movement/health/team-like data where semantics are generic.
- Generic Entity/Template infrastructure: factory, templates, serialization registry after separation from MythHunter-specific archetypes.
- System infrastructure: registry lifecycle, scheduler, generic system groups after removing game phase/domain concerns.
- Networking infrastructure: transport, protocol, serialization, security as optional modules; current implementations require major refactoring.
- Generic validation, scene abstraction, state machine primitives, logging abstractions, developer tooling APIs.

## Framework modules requiring extraction/refactoring
- Stats / Resources / Statuses / Buffs / Debuffs / Damage / Resistance / Death.
- Inventory / Equipment / Items / Loot.
- Abilities / Combat / Progression / Cooldowns.
These are final-framework scope candidates but are not yet fully universalized in the current MythHunter code.

## Boundary rule
Framework must not reference MythHunter domain types, concrete game phases, hero data, lobby rules, concrete scenes, game UI, or game-specific content.

## Major findings
- Current codebase is strongly mixed: Framework intent exists, but Game Layer and infrastructure are interwoven.
- Large technical debt is caused by monolithic classes, reflection-based discovery, static globals, hidden lifecycle side effects, duplicated abstractions, Unity dependencies in low-level modules, and persistence/network contract mixing.
- Networking is architecturally incomplete: client/server are stubs, protocol identifiers are unstable, security is incomplete.
- Events are architecturally overloaded: EventBus owns too many pipeline responsibilities.
- ECS is reusable in concept but is not a complete archetype/chunk ECS.
- Components/entities/systems contain both universal infrastructure and MythHunter-specific gameplay.

## Recommended extraction strategy
1. Create a pure Framework assembly/package with zero references to Game Layer.
2. Move only generic contracts/primitives first.
3. Introduce explicit registries instead of automatic assembly scans.
4. Replace reflection/dynamic hot paths with registered factories/codecs.
5. Split local event delivery from scheduling, middleware, pooling and networking.
6. Make Unity-specific adapters sit outside pure Framework core.
7. Add test coverage around contracts before moving implementations.
8. Migrate Game Layer systems to depend on Framework contracts.

## Conclusion
Framework boundary is identifiable, but current codebase is not yet cleanly separable without refactoring.
