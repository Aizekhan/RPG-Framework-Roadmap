# EPIC 02.8 — Final Architecture Baseline

## Status
DONE

## Purpose
Consolidate EPIC 02 architecture decisions into a single operational baseline for extraction. This document links the ownership map, assembly/package grouping, dependency policy, composition-root strategy and migration strategy into one consistent reference.

## Scope and current-state caveat
This is the **target architecture**, not a claim that MythHunter already implements it. Source code remains in `Aizekhan/MythHunter`, branch `dev`. No source code was changed by this baseline task.

## 1. Architecture zones

### Zone A — Platform-neutral Framework contracts/primitives
Owns reusable abstractions that can exist without MythHunter and Unity:
- ECS/entity/component contracts and identity primitives;
- system/lifecycle contracts;
- event contracts;
- DI contracts;
- serialization/codec contracts and schema metadata;
- logging and validation abstractions;
- resource and scene-loading abstractions where truly platform-neutral.

Rules:
- no `MythHunter.*`;
- no UI/presentation;
- no UnityEngine/UnityEditor in platform-neutral contract assemblies;
- no concrete providers or adapter implementations;
- no game-specific phases/events/content.

### Zone B — Framework runtime and reusable infrastructure
Owns implementations of ECS, DI, event dispatch, system lifecycle/scheduling, generic entity/template infrastructure, serialization, persistence, logging/validation runtime and other reusable runtime capabilities.

Rules:
- depend on stable Framework contracts/lower-level modules;
- no references to MythHunter, UI/presentation, Unity Editor or concrete provider adapters;
- no implicit global discovery of game types;
- each module has an explicit responsibility and lifecycle/error semantics.

### Zone C — Optional Framework modules
Resources, scene management, pooling, replay, networking, cloud abstractions and developer diagnostics are opt-in capabilities.

Rules:
- Framework Core must not require them;
- each declares explicit dependencies;
- provider/platform implementations stay in adapters;
- Networking and Persistence use distinct protocol/schema responsibilities;
- current audited networking is incomplete and must not be treated as production-ready.

### Zone D — Platform/provider adapters and tools
Unity runtime adapters implement Framework-facing contracts using Unity APIs. Provider adapters integrate selected cloud/network services. Editor tooling is isolated in Editor-only assemblies.

Rules:
- dependencies point from adapter to Framework abstraction;
- Framework never references adapters;
- runtime assemblies never reference Editor assemblies;
- generic tools and MythHunter authoring are distinguished.

### Zone E — MythHunter Game Layer
Owns the concrete application, domain, gameplay rules, content, UI and Game composition root:
- application/game flow and concrete states;
- heroes, lobby, phase policy, combat and abilities;
- concrete game events and entities/archetypes/templates;
- game services/settings/content;
- UI/presentation;
- MythHunter-specific authoring tools;
- composition of selected Framework modules and adapters.

The Game Layer may depend on Framework APIs; the Framework must never depend on MythHunter.

## 2. Initial assembly/package policy
Use a small number of enforceable assemblies rather than one assembly/package per logical module.

Proposed groups:
- `RPGFramework.Contracts` — minimal shared neutral contracts only.
- `RPGFramework.ECS`
- `RPGFramework.Systems`
- `RPGFramework.DI`
- `RPGFramework.Events`
- `RPGFramework.Serialization`
- `RPGFramework.Persistence`
- optional modules such as `RPGFramework.Resources`, `RPGFramework.Pooling`, `RPGFramework.Replay`, `RPGFramework.Networking`.
- `RPGFramework.Unity` — runtime adapters.
- `RPGFramework.Unity.Editor` — Editor-only tooling.
- `MythHunter.Game`, optionally split into `MythHunter.UI`, `MythHunter.Unity`, and `MythHunter.Editor` where actual references justify it.

These are proposed names. Final .asmdef placement must be based on inspection of the actual project structure and current references. Logical module boundaries do not automatically imply separate assemblies or separate UPM packages.

## 3. Dependency rules

### Allowed
- MythHunter Game/UI/Application → public Framework APIs.
- Runtime module → its own contracts and required lower-level Framework modules.
- Unity/provider adapter → Framework contracts/modules + corresponding platform/provider SDK.
- Persistence → Serialization.
- Replay → explicit event/serialization contracts.
- Game composition root → selected Framework modules, adapters and MythHunter registrations.
- Editor tooling → runtime/public types for inspection, while runtime remains isolated from Editor.

### Forbidden
- Framework → any MythHunter assembly/type.
- Framework contracts/core → UI/presentation or Unity Editor.
- Platform-neutral Framework contracts/runtime → UnityEngine, unless isolated in a specifically Unity-named adapter module.
- Domain/gameplay → UI implementation.
- Generic EventBus → concrete MythHunter events.
- Generic system registry/scheduler → MythHunter phase policy.
- ECS runtime → concrete gameplay systems/entities.
- Serialization → Persistence or Networking.
- Networking → concrete game events/content or unstable CLR type identity as a durable wire ID.
- Runtime → Editor assembly.
- Any cyclic assembly/module reference.
- Static service locators or global automatic scans used to bypass explicit ownership/dependency injection.

Exceptions require a documented owner, reason and proof that direction/cycle/platform rules remain intact.

## 4. Composition root
MythHunter has one authoritative Game Composition Root.

It must:
1. configure DI/lifetimes;
2. register Framework modules;
3. select optional modules;
4. bind Unity/provider adapters;
5. register MythHunter application/domain/gameplay/content/presentation services;
6. validate registrations and dependency direction;
7. start the game exactly once;
8. coordinate cancellation, shutdown and disposal.

Framework modules may expose explicit registration functions but do not own application startup, discover MythHunter services or create a competing root. Unity bootstrap host delegates to the Game Composition Root.

## 5. Migration strategy
Migrate incrementally with a compiling checkpoint after each cohesive step.

1. **Baseline:** record source commit, Unity version, working-tree state, assembly definitions, dependencies, compile and test results. Preserve a rollback point.
2. **Enforce first boundary:** isolate the first neutral Framework slice and Editor-only assemblies. Confirm no Framework → MythHunter reference.
3. **Extract contracts/primitives:** only where ownership and dependencies are clear.
4. **Extract runtime modules:** logging/validation, DI, events, systems, ECS after its storage/identity decisions, serialization, generic entity/template infrastructure.
5. **Extract adapters:** Unity resources/scene integration, provider integrations and Editor tooling.
6. **Establish composition root:** separate Framework registration, adapter binding and MythHunter registration.
7. **Move Game Layer behaviour:** application flow, concrete states, lobby, heroes, phases, combat, events, content and UI.
8. **Remove debt:** ComponentCache duplication, duplicate EventBus, phase ownership overlap, serialization/protocol coupling, unstable identifiers/reflection paths, async lifecycle problems.
9. **Harden:** automated dependency checks, tests and a working MythHunter path.

Do not proceed after a new compilation/test regression until it is repaired or explicitly understood. Keep changes small, reviewed and reversible; preserve unrelated user changes. Do not perform bulk moves based only on folder names.

## 6. Required validation
Before considering the architecture implemented:
- Framework runtime builds without MythHunter references.
- Platform-neutral contracts do not depend on Unity APIs.
- Runtime assemblies do not reference Editor assemblies.
- No assembly cycles exist.
- Game composition is explicit and unique.
- Adapter bindings and module registrations are explicit.
- Optional modules remain optional.
- Relevant tests and Unity compilation pass, with baseline failures separated from regressions.
- Dependency rules are enforced mechanically where feasible, not only written in documentation.

## 7. Audit-driven migration targets
The following remain known work items, not resolved issues:
- mixed Core/Game startup responsibility;
- installer/registry ownership and hardcoded discovery;
- EventBus imports concrete events and combines too many pipeline roles;
- EcsWorld/SystemRegistry dependency and lifecycle problems;
- ComponentCache duplication;
- phase policy embedded in generic system infrastructure;
- generic entity/factory code mixed with hero/enemy/item content;
- serializer registry and network IDs/versioning;
- networking client/server stubs and incomplete security;
- PreloadManager hardcoded phases/scenes/resource keys;
- DI lifecycle/scopes and async startup/cancellation;
- Unity runtime vs Editor tooling boundaries.

## 8. What is deliberately not decided
- final ECS storage model;
- final public APIs for Framework modules;
- concrete networking transport/security implementation;
- which optional modules are independent public packages;
- exact Unity assembly definition placement until current project references are inspected;
- detailed decomposition of large systems beyond the boundaries supported by current evidence.

Those decisions belong to their designated implementation tasks and must not be guessed merely to make the map look complete.

## Conclusion
The target architecture is now consolidated: **platform-neutral Framework contracts/runtime, optional modules, one-way platform/provider adapters, and a concrete MythHunter Game Layer composed through one Game Composition Root**. The next phase may begin only after the existing MythHunter source is inspected against this baseline and the baseline compile/test state is recorded.

No MythHunter source code was changed in EPIC 02.
