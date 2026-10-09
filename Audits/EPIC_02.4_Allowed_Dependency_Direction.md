# EPIC 02.4 — Allowed Dependency Direction

## Status
DONE

## Purpose
Define the allowed direction of compile-time and runtime dependencies between logical modules and assemblies. This policy follows the target module map and proposed assembly boundaries.

## Fundamental rule

Dependencies point inward toward stable abstractions, and never upward from reusable infrastructure into the concrete game.

```text
MythHunter UI / Application / Gameplay / Content
                    |
                    v
       Framework abstractions and modules
                    ^
                    |
       Platform/provider adapters implement
       Framework-facing contracts at composition
```

Interpretation:
- Game code may call Framework APIs.
- Adapter implementations may implement Framework contracts.
- The Game composition root wires both sides together.
- Framework must not discover or resolve MythHunter services implicitly.
- This is a compile-time reference policy; runtime objects may be connected by dependency injection at the composition root.

## Layer rules

### L0 — Platform-neutral Framework contracts and primitives
Examples: ECS-facing contracts, event contracts, DI contracts, serialization contracts, logging abstractions, minimal shared primitives.

Allowed references:
- language/runtime standard libraries;
- other lower-level neutral contracts only where required and cycle-free.

Forbidden references:
- MythHunter namespaces or assemblies;
- UnityEngine / UnityEditor;
- UI/presentation;
- concrete runtime infrastructure implementations;
- provider SDKs;
- concrete game events, phase enums, content IDs or scene names.

Rule: keep these contracts small and owned by the module that defines the relevant concept. Do not create dependencies only to share trivial types.

### L1 — Framework runtime and reusable modules
Examples: ECS runtime, DI runtime, Events runtime, Systems runtime, generic entities/templates, Serialization, Persistence, Resources, Pooling, Replay, Networking.

Allowed references:
- the contracts it implements;
- required lower-level Framework contracts/modules;
- standard runtime libraries.

Forbidden references:
- MythHunter.Game / MythHunter.UI / MythHunter.Content / MythHunter.Application;
- Unity Editor APIs in runtime code;
- concrete Unity/provider adapters;
- concrete game-specific phases/events/heroes;
- UI or presentation;
- implicit assembly scans that pull unknown game types into the module.

Rule: dependencies must match the module's responsibility. For example, Persistence may use Serialization, Replay may use Events + Serialization contracts, and an ECS scheduler may use system/ECS contracts. Reverse dependencies are forbidden.

### L2 — Platform and provider adapters
Examples: Unity runtime adapters, Unity resource provider, Unity scene loader, provider-specific cloud adapter, concrete transport adapters.

Allowed references:
- the Framework contracts/modules they implement or integrate;
- their platform/provider SDK;
- game composition APIs only when needed to register concrete implementations and without creating a cycle.

Forbidden references:
- forcing Framework contracts/runtime to reference the adapter;
- Editor APIs from runtime adapter assemblies;
- concrete game policies inside generic adapter implementations unless isolated in a game-specific adapter.

Rule: implementation dependencies point from adapter to abstraction, never from abstraction back to adapter.

### L3 — MythHunter domain, gameplay, application and content
Examples: heroes, lobby, phases, combat rules, domain events, game flow, concrete entities/templates/content, game settings.

Allowed references:
- Framework contracts and the selected Framework runtime/optional modules;
- game-owned lower-level domain/application contracts;
- selected platform adapter APIs only at integration/composition edges.

Forbidden references:
- placing game rules in Framework to avoid a Game dependency;
- making generic modules depend on concrete game events/components;
- domain rules depending on presentation;
- a lower-level game domain module depending on a high-level UI/application module unless the relationship is inverted through an interface/event.

Rule: gameplay may specialize Framework behaviour, but Framework cannot specialize itself for MythHunter.

### L4 — Presentation and Editor tooling
Presentation:
- UI can depend on public Game/Application/Domain APIs and Framework public APIs needed for display.
- Domain and Framework must not depend on UI, presenters, controllers, or view types.
- UI communicates commands into application/game APIs or consumes public notifications; avoid a broad reverse dependency on implementation details.

Editor tooling:
- Unity Editor assemblies may inspect runtime/game types, but runtime assemblies must never reference editor assemblies.
- Generic Framework Editor tooling must not place MythHunter content/rules into Framework Core.
- MythHunter authoring tools belong to MythHunter.Editor unless demonstrably generic.

## Specific dependency matrix

| From | May depend on | Must not depend on |
|---|---|---|
| Framework contracts | standard runtime + lower-level neutral contracts | UnityEngine/Editor, MythHunter, UI, providers |
| Framework runtime | own contracts + required Framework contracts/modules | MythHunter, UI, concrete adapters, Editor APIs |
| Optional Framework module | declared lower-level Framework contracts/modules | game-specific content/events, unrelated optional modules by default |
| Unity/provider adapter | Framework contracts/modules + platform/provider SDK | reverse references from Framework; Editor APIs at runtime |
| MythHunter domain/gameplay | Framework public APIs + game-owned domain contracts | UI implementation, Editor tooling |
| MythHunter application/composition | game modules + selected Framework modules + adapters | dependency inversion violations into Framework |
| MythHunter UI | game-facing public APIs + display contracts | being referenced by Domain or Framework |
| Editor tools | runtime/game public APIs + UnityEditor | runtime referencing Editor assemblies |

## Required special cases

### Events
- Framework.Events owns generic event abstractions and dispatch.
- Concrete lobby, hero, phase and combat events are game-owned.
- Event middleware/pooling/replay/network bridging must depend on event contracts, not force EventBus to depend on all those modules.
- A generic event contract must not encode MythHunter phase or hero semantics.

### ECS and Systems
- ECS contracts/runtime cannot depend on gameplay systems or concrete game entities.
- Generic Systems runtime may use ECS and event abstractions only through stable contracts.
- MythHunter phases are game policy; they must not become a generic scheduler dependency.
- ComponentCache must not become a second source of truth alongside ECS storage.

### Serialization, Persistence and Networking
- Serialization owns codecs and schema metadata.
- Persistence owns save/load lifecycle and save schema policies; it may depend on Serialization.
- Networking owns protocol/wire schemas and stable wire identifiers; it must not make Persistence own network format.
- No durable external format may rely only on CLR type names or an unstable generated ID.
- The exact network protocol/security design remains for the Networking epic.

### Resources, Pooling and Replay
- Generic resource and pooling contracts must not carry concrete MythHunter keys, phases, scenes or assets.
- Unity resource/scene implementations live in Unity adapters.
- Replay consumes explicitly defined event/serialization contracts and remains optional.
- No optional resource/pooling/replay module is a mandatory dependency of Framework Core.

### Logging and Validation
- Framework depends on neutral logging/validation abstractions.
- A replaceable logger implementation may live in Framework runtime or adapters depending on its dependencies.
- Runtime validation stays separate from editor project scans.
- No static global service locator should be introduced as a workaround for dependency direction.

## Cycle policy

A dependency cycle between assemblies/modules is forbidden.

When A requires a service currently owned by B, resolve it using the appropriate method:
1. move the contract to the lowest stable owner that actually owns the concept;
2. inject the interface/contract from the composition root;
3. publish a neutral event/command where that is the correct domain relationship;
4. split a module only when the split gives a real, enforceable boundary.

Do not break cycles by moving game-specific interfaces into Framework Core without ownership justification.

## Enforcement expectations

During extraction:
- use `.asmdef` references to make boundaries enforceable;
- keep Unity Editor references restricted to Editor assemblies;
- make registration explicit rather than scanning all assemblies;
- inspect new references as part of each pull request/change;
- validate that the Framework runtime has no reference to `MythHunter.*`;
- validate that runtime assemblies do not reference `*.Editor`;
- build/test after each boundary change.

Automated architecture tests or a dependency-graph check should be added when extraction starts. This document specifies policy, but the source repository has not yet been mechanically checked against it.

## Conclusion
The allowed direction is: **game/application and presentation consume Framework APIs; adapters implement Framework contracts; Framework does not reference the Game Layer, UI, Unity Editor, or concrete provider implementations.** All modules must remain acyclic, with integration performed explicitly by composition roots.

No source code changes are made in this task.
