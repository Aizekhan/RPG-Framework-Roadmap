# Audit — Events

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Events

## Name

Events

## Structure observed

Events contains:
- EventBus / SimpleEventBus
- Event contracts and subscriptions
- EventPool
- EventBatcher
- EventThrottler
- EventHandlerBase
- Domain events
- Debugging tools
- Network events
- Extensions

## 1. Is it needed by any RPG?

Yes.

An event mechanism is useful for decoupled communication between systems, lifecycle notifications, domain occurrences and integration boundaries.

The universal Framework should provide the event infrastructure and contracts. Concrete RPG/game events belong to the Game Layer or specific modules.

## 2. Does it depend on a specific game?

Partially.

The base event abstractions can be universal, but the current implementation contains strong game/application dependencies.

Examples:
- EventBus imports MythHunter domain namespaces and UnityEditor.
- Domain events contain MythHunter GamePhase and GameStateType.
- NetworkEventBus depends directly on MythHunter.Networking.
- EventStore and EventLogger depend on Unity runtime information.

## 3. Can it be reused without changes?

Partially.

Reusable candidates:
- IEvent
- IEventBus
- subscription/publishing model
- EventPriority
- generic EventPool concept
- generic throttling concept
- generic subscriber lifecycle

Not reusable unchanged:
- current EventBus implementation
- Domain events
- NetworkEventBus
- current batching implementation
- Unity-specific debugging/history logic

## 4. Framework or Game Layer?

Shared abstraction / mixed.

Framework candidates:
- IEvent
- IEventBus
- EventPriority
- generic subscriber abstraction
- generic middleware/pipeline concepts
- generic event pooling/throttling where justified

Game Layer:
- Domain events
- MythHunter GamePhase/GameState events
- concrete lobby/gameplay events
- network-specific event mappings

Infrastructure/Tools:
- EventStore
- EventLogger
- debug middleware

## 5. Are there architectural problems?

- EventBus is very large and combines subscription, synchronous dispatch, asynchronous dispatch, priorities, queues, pending events, processor caches, throttling-related integration and lifecycle concerns.
- EventBus contains a direct UnityEditor dependency, which is inappropriate for reusable runtime Framework code.
- SimpleEventBus duplicates EventBus responsibility and ignores event priority semantics.
- EventBatcher is incomplete: batchProcessor is accepted but not stored/used, and processing currently republishes individual events rather than producing a batch event.
- EventBatcher uses reflection to invoke generic Publish.
- EventThrottler contains Unity Time dependencies and uses dynamic dispatch.
- Domain events are colocated with generic event infrastructure.
- NetworkEventBus inherits from EventBus and directly couples the event infrastructure to networking transport/system abstractions.
- NetworkEventBus scans assemblies by the literal MythHunter assembly name, making reuse problematic.
- Event contracts use GetEventId implementations that create a new Guid on every call for struct events, meaning an event's ID is not stable across repeated reads.
- EventPool uses object queues despite events being structs, adding boxing and complexity.
- Event debugging is embedded into the event layer rather than separated cleanly as optional tooling.

## 6. What does it depend on?

Observed dependencies:
- System/.NET
- Cysharp UniTask
- MythHunter.Core.DI
- MythHunter.Utils.Logging
- MythHunter.Core.Game
- MythHunter.Events.Domain
- MythHunter.Networking.Core
- MythHunter.Networking.Messages
- UnityEngine
- UnityEditor

## 7. Who depends on it?

Likely consumers:
- Core/SystemRegistry
- Systems and Components integration
- gameplay/domain modules
- networking
- UI/presentation
- debugging tools
- application lifecycle

The exact consumer graph belongs to the later dependency-map and file-level audits.

## 8. Are there unnecessary dependencies?

Yes, clear candidates exist:
- EventBus -> UnityEditor
- generic EventBus -> concrete domain events/namespaces
- generic event layer -> networking
- EventStore -> Unity runtime details
- generic Event layer -> MythHunter-specific assembly scanning

These should be verified and removed from the future Framework core boundary.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split Event contracts, EventBus runtime, middleware, tooling, domain events and network events into separate modules.
- Remove UnityEditor from runtime EventBus.
- Make the core EventBus independent of concrete domain events.
- Eliminate or clearly justify SimpleEventBus as a separate implementation/test double.
- Redesign EventBatcher around actual batching semantics.
- Separate throttling from EventBus and make it platform-neutral.
- Rework EventPool only if profiling proves pooling useful.
- Move network event transport integration behind an abstraction.
- Stabilize event identity semantics.
- Separate debug/history tooling from event transport.

## Conclusion

- [ ] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: Events is a mixed/shared structure. The event contracts and generic bus concepts are Framework candidates, while domain, networking and debug implementations should be separated. The current EventBus and supporting utilities contain substantial technical debt and several dependencies that violate the intended reusable Framework boundary.
