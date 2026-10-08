# Final Audit — Technical Debt List

## Critical / architecture-breaking
- Framework and Game Layer are interwoven; no hard assembly/package boundary.
- Core infrastructure references concrete MythHunter domain/application concerns.
- EventBus is a monolith combining delivery, scheduling, async execution, pooling, buffering and debugging.
- Network client/server are stubs rather than real transport implementations.
- Network protocol uses unstable CLR type names and reflection-based runtime discovery.
- Network security lacks proper key provisioning, replay protection, encryption and session/authentication policy.
- ECS is dictionary-based storage rather than a complete archetype/chunk ECS; ComponentCache can become a second state source.
- Multiple composition roots/registries and duplicated responsibilities exist.

## High
- Reflection/dynamic invocation in hot paths.
- Static global registries/hooks and implicit shared state.
- Automatic assembly scanning based on MythHunter name.
- Generic infrastructure directly imports game-domain events.
- Game phase semantics duplicated across Registry/PhaseSystem/Providers/SystemGroups.
- Concrete game flow, lobby, hero and combat logic concentrated in large systems.
- Serialization contracts mixed between persistence and networking.
- Missing explicit lifecycle/cancellation/error policies across async infrastructure.
- Duplicate EventBus implementation (SimpleEventBus).
- Hardcoded gameplay data inside systems.

## Medium
- UnityEngine/UnityEditor dependencies leak into low-level infrastructure.
- Generic modules require MythLogger/DI unnecessarily.
- Runtime configuration and presentation data mixed with domain/runtime state.
- Placeholder serializers and incomplete TODO paths.
- Test/simulation APIs present in production networking classes.
- Broad catch-and-log error handling hides failures.

## Refactoring priority
1. Establish Framework/Game assembly boundary.
2. Extract pure contracts/primitives.
3. Replace implicit reflection/global discovery with explicit registries.
4. Split EventBus into bus/dispatcher/middleware/optional pipeline modules.
5. Separate network transport/protocol/security from application events.
6. Stabilize ECS storage and remove duplicate state sources.
7. Split large gameplay/application systems by responsibility.
8. Add lifecycle, cancellation, error and compatibility contracts.
9. Add tests around all extracted Framework modules.
