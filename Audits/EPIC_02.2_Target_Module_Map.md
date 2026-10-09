# EPIC 02.2 — Target Module Map

## Status
DONE

## Purpose
Define the logical ownership and module map for the reusable RPG Framework, platform adapters, tools, and the concrete MythHunter Game Layer. This is a logical map; exact Unity assembly definition/package names are decided in the next task.

## Design principles
1. Framework modules must be reusable without MythHunter domain content.
2. The default dependency direction is Game Layer → Framework abstractions/modules → platform adapters at composition boundaries.
3. Optional capabilities must not become mandatory dependencies of Framework Core.
4. UI, Unity Editor, concrete game content, and provider SDKs must not leak into Framework Core.
5. Keep a module only when it has a clear responsibility and explicit contract.
6. A proposed module is not proof that the current implementation is already cleanly separable; audit findings show substantial refactoring is required.

## Logical module map

### A. Framework contracts / primitives
These contain platform-neutral contracts and fundamental types. They must not depend on Unity, MythHunter, presentation, or provider SDKs.

- **Framework.Contracts** — shared abstractions only where genuinely cross-module; avoid a catch-all dumping ground.
- **Framework.ECS.Contracts** — entity IDs/identity, component and entity-manager contracts, world/query-facing contracts.
- **Framework.Systems.Contracts** — system APIs, lifecycle contracts, update/scheduling abstractions.
- **Framework.Events.Contracts** — event types and event publisher/subscriber/bus contracts; no concrete MythHunter events.
- **Framework.DI.Contracts** — registration/resolution/lifecycle contracts.
- **Framework.Serialization.Contracts** — codec/serializer contracts and schema/version metadata; distinct from network protocol identity and save-file policy.
- **Framework.Resources.Contracts** — resource and provider abstractions.
- **Framework.Logging.Contracts** — logging abstraction.
- **Framework.Validation** — reusable validation primitives and guard/Ensure APIs, without Editor-only scanning.

These are logical ownership groups, not a commitment to create one assembly per bullet. The next task will decide assembly/package grouping.

### B. Framework runtime / infrastructure
Reusable implementations depend on the corresponding contracts and shared lower-level abstractions.

- **Framework.ECS.Runtime** — entity/component storage, world, entity lifecycle and queries. Storage architecture remains an explicit decision for the ECS task; do not assume current code is archetype/chunk ECS.
- **Framework.DI.Runtime** — container and lifecycle/scopes; Unity lifecycle integration remains outside this pure runtime.
- **Framework.Events.Runtime** — local event dispatch only. Scheduling, middleware, pooling, replay and network bridging remain separable responsibilities; avoid recreating the current monolithic EventBus.
- **Framework.Systems.Runtime** — system registry, lifecycle and scheduler; must not own MythHunter phase semantics.
- **Framework.Serialization.Runtime** — generic codec/registry infrastructure with stable schema identifiers; no CLR type names as a durable external protocol contract.
- **Framework.Entities** — generic entity factory/template primitives only; no Hero/Enemy/Item creation methods tied to MythHunter.
- **Framework.Logging** — default or replaceable logging implementation, behind the logging contract.
- **Framework.Validation.Runtime** — runtime validation only; Editor project analysis stays in tools.
- **Framework.Persistence** — save/load orchestration and persistence policies, separated from generic serialization and network encoding.

### C. Optional Framework runtime modules
These can be adopted by a game when needed. Framework Core must not require them.

- **Framework.Resources** — resource provider coordination and lifecycle.
- **Framework.SceneManagement** — platform-neutral scene transition contracts and orchestration.
- **Framework.Pooling** — generic object-pool contracts/implementation where pooling is useful.
- **Framework.Replay** — replay recording/playback contracts and runtime; depends on explicit event/serialization contracts.
- **Framework.Networking** — transport/session/protocol/replication boundaries. Client/server implementations in the current audit are stubs and must not be represented as production-ready.
- **Framework.Cloud.Contracts** — optional provider-neutral auth/data/analytics contracts if they are proven useful to multiple projects; provider implementations remain adapters.
- **Framework.Middleware** — only if the finalized event/network pipelines require reusable middleware contracts. Avoid introducing it as a separate runtime package solely for naming symmetry.

### D. Platform and provider adapters
Adapters implement Framework-facing contracts and may depend on their platform/provider SDK. Framework Core must never depend back on adapters.

- **Framework.Unity.RuntimeAdapters** — Unity implementations for scene loading, resource access, Unity object lifecycle, and other Unity-only runtime integrations.
- **Framework.Unity.Editor** — editor-only inspectors, project validation and editor extensions; must not be referenced by runtime assemblies.
- **Framework.Cloud.<Provider>** — concrete cloud/auth/data/analytics integrations per provider.
- **Framework.Networking.<Transport>** — concrete transports only when selected and implemented.
- **Framework.Diagnostics** — optional runtime diagnostics/profiling adapters that do not pollute core APIs.

### E. MythHunter Game Layer
These modules own the rules and content of this specific game. They can depend on Framework contracts/runtime and selected adapters.

- **MythHunter.Application** — GameBootstrapper/GameFlowManager replacement, application lifecycle and game-level orchestration.
- **MythHunter.Domain** — MythHunter rules and domain models/events where a domain grouping is useful.
- **MythHunter.Gameplay.Heroes** — hero lifecycle and game-specific race/class rules.
- **MythHunter.Gameplay.Lobby** — player slots, hero selection, readiness, timers and lobby orchestration.
- **MythHunter.Gameplay.Phases** — MythHunter phase enum, order, durations, and transitions.
- **MythHunter.Gameplay.Combat** — concrete combat rules, damage behavior, abilities, runes, AI and spawning policies; can be subdivided after responsibility-level design.
- **MythHunter.Content** — concrete archetypes/templates, hero/enemy/item definitions, settings and content configuration.
- **MythHunter.UI** — views, presenters, controllers, navigation and screens.
- **MythHunter.Services** — game-specific services and application settings that do not belong in generic Framework.
- **MythHunter.Composition** — the single Game composition root; registers Framework modules, game services, content, and chosen adapters.
- **MythHunter.Authoring** — MythHunter-specific content authoring tools, in Editor-only assemblies.

Do not create extra Game Layer packages merely to mirror folders. Exact grouping is resolved when assembly boundaries and concrete code dependencies are reviewed.

## Dependency map

```text
MythHunter.UI ───────────────┐
MythHunter.Application ──────┤
MythHunter.Domain/Gameplay ──┼──> Framework.Contracts + selected Framework modules
MythHunter.Content ──────────┤                   ▲
MythHunter.Composition ──────┘                   │ implements/uses contracts
                                                │
                                  Unity/provider adapters
                                                │
                                      Unity/platform/provider APIs

Framework Core must not reference:
- MythHunter.*
- Unity Editor APIs
- game UI or presentation
- concrete game scenes/resource keys/content
- concrete cloud providers or transport implementations
```

The diagram describes logical dependency ownership. At runtime, the composition root wires the selected implementations to their contracts; Framework modules should not resolve game or adapter implementations implicitly.

## Mapping current audit findings to target ownership

| Current audited area | Target owner | Required correction |
|---|---|---|
| Core/ECS | Framework.ECS.Contracts + Framework.ECS.Runtime | Separate contracts/runtime; decide storage later; isolate ComponentCache |
| Core/DI | Framework.DI.Contracts + Framework.DI.Runtime | Correct lifecycle/scopes; keep Unity lifecycle integration out |
| Core/Game, GameFlowManager, GameBootstrapper | MythHunter.Application / MythHunter.Composition | Remove game flow from Framework Core |
| concrete GameStateMachine/GameStates | MythHunter.Application | Keep generic state primitives separate if they are demonstrably reusable |
| SystemRegistry/SystemGroups | Framework.Systems.Runtime | Split registration, scheduling, lifecycle and domain-phase policy |
| PhaseSystem/phase rules | MythHunter.Gameplay.Phases | Remove phase semantics from generic registry |
| EventBus | Framework.Events.Runtime | Split dispatch from pipeline, middleware, pooling, replay and network bridge |
| concrete hero/lobby/game events | MythHunter.Domain or owning gameplay module | No concrete game events in Framework |
| EntityFactory/archetypes/templates | Framework.Entities + MythHunter.Content | Generic factories/templates separate from concrete entity definitions |
| generic serializer/registry | Framework.Serialization.* | Stable schema IDs/versioning; separate save policy and network protocol |
| persistence/save-load behavior | Framework.Persistence + MythHunter game schemas | Do not reuse wire-protocol identity as persistence schema identity |
| network client/server/messages/security | Framework.Networking + selected transport adapter | Treat current audited networking as incomplete/stubbed |
| ResourceManager/PreloadManager/pool | Framework.Resources / Framework.Pooling + Unity adapters + game config | Move hardcoded keys/phases to MythHunter configuration |
| Cloud/Auth/Data/Analytics | optional contracts + provider adapters | Never a mandatory Framework Core dependency |
| UI/presenters/controllers | MythHunter.UI | Domain/framework must not depend on presentation |
| Debug/profiling/inspectors | Framework.Diagnostics / Framework.Unity.Editor / Tools | Keep editor-only code out of runtime |
| Authoring/code generators | Framework.Unity.Editor or MythHunter.Authoring | Split generic tools from MythHunter-specific authoring |
| logging and validation | Framework.Logging.Contracts / Framework.Validation | Replace MythLogger behind abstraction; separate runtime and editor validation |

## Explicit non-decisions
This map does not yet decide:
- exact `.asmdef` names, package boundaries, or how many assemblies to create;
- exact namespaces/folder movement;
- final ECS storage (dictionary, sparse set, archetype/chunk or hybrid);
- concrete network transport/provider;
- whether every optional module warrants its own distributable package;
- final decomposition of large Combat/Events/Networking systems.

These decisions must not be guessed prematurely; they belong to the next architecture tasks or to their designated implementation epics.

## Technical debt and migration notes
- Existing code is mixed across Framework/Game/Unity/platform concerns; this map is a target, not a claim about current state.
- Reflection and automatic assembly scanning currently hide dependency edges; later work must replace implicit discovery with explicit registrations/codecs where appropriate.
- Existing networking is not production-ready according to the audit.
- EventBus and SystemRegistry currently own too many responsibilities and must be decomposed before extraction.
- `UnityEngine` types are acceptable in Unity adapters, presentation and game-specific code; they are forbidden only in platform-neutral Framework contracts/runtime groups.
- Optional modules must have no reverse dependency into Framework Core.
- No MythHunter source code was changed in this task.

## Conclusion
The target architecture contains five logical zones:
1. Framework contracts/primitives.
2. Framework runtime/infrastructure.
3. Optional Framework modules.
4. Platform/provider adapters and separate tooling.
5. MythHunter Game Layer and its composition root.

The target module map is complete enough to make the next decision: define assembly/package boundaries from concrete dependency evidence.
