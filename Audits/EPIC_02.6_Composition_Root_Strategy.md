# EPIC 02.6 — Composition-Root Strategy

## Status
DONE

## Purpose
Define where Framework infrastructure and MythHunter-specific services are created, configured, registered and started. This is an architecture decision only; existing source code is not changed in this task.

## Decision
MythHunter has one authoritative **Game Composition Root**. Framework modules may expose registration/configuration functions, but must not independently start the game, scan the whole application for types, or resolve MythHunter services.

Framework configuration and Game configuration are separate responsibilities, connected explicitly by the Game Composition Root.

## Responsibilities

### Framework module registration
Each Framework module may expose an explicit registration entry point, for example:
- ECS runtime registration;
- DI runtime registration;
- event dispatch registration;
- systems/scheduler registration;
- serialization registration;
- selected optional modules (Resources, Pooling, Replay, Networking, Persistence).

A module registration entry point must:
- register only services owned by that module;
- state its required dependencies;
- avoid registering MythHunter-specific types;
- avoid hidden global/static side effects;
- fail visibly and deterministically when required dependencies/configuration are missing;
- not start gameplay or load concrete game content unless that is explicitly the responsibility of a higher layer.

These examples establish the pattern; they do not assert that such registration APIs already exist.

### Framework composition
A Framework composition function/root:
- creates/configures Framework infrastructure only;
- chooses only Framework modules and implementations;
- uses explicit registration rather than broad assembly scanning;
- does not reference or resolve MythHunter services, game rules, UI or content;
- does not own Unity scene flow or the game lifecycle.

The Framework can provide a composition helper, but it must not become a second competing application entry point.

### MythHunter Game Composition Root
The Game Composition Root owns the final application graph and startup sequence. It is responsible for:
1. creating/configuring the DI container or selected composition mechanism;
2. registering Framework core modules;
3. selecting optional Framework modules;
4. registering Unity/provider adapters;
5. registering MythHunter application, domain, gameplay and presentation services;
6. binding interfaces to concrete implementations;
7. validating required registrations and the dependency graph;
8. starting the game runtime exactly once;
9. coordinating shutdown/disposal in a defined order.

The Game Composition Root is the only place allowed to know the full set of concrete Framework, adapter and MythHunter implementations.

## Recommended logical startup flow

```text
Unity entry point / bootstrap host
             |
             v
MythHunter Game Composition Root
             |
             +--> configure DI/lifetime
             +--> register Framework contracts + implementations
             +--> register selected optional modules
             +--> register Unity/provider adapters
             +--> register MythHunter services, domain and gameplay
             +--> validate registrations/dependencies
             |
             v
Start application/game lifecycle
             |
             v
Controlled shutdown and disposal
```

The Unity bootstrap host is an adapter/entry mechanism. It delegates composition to the Game Composition Root and must not duplicate registrations or run an independent parallel startup path.

## Lifetime and lifecycle rules
- Each long-lived service must have an explicitly defined lifetime (application, session/game, or transient) where the DI implementation supports lifetimes.
- Session/game-scoped services must be disposed or reset when the corresponding session ends.
- Shutdown, cancellation and disposal order must be deterministic.
- Async initialization must have observable completion/failure and cancellation; avoid `async void` for orchestration.
- Do not hide registration/initialization failures and continue in a partially composed state unless a documented recovery policy explicitly allows it.
- Unity main-thread requirements must be handled by Unity adapters/application lifecycle, not inserted into platform-neutral Framework Core.
- No static global service locator or hidden singleton registry may replace explicit composition.

## Installers and migration target
The audit found that existing installers mix Framework and game registrations, the static InstallerRegistry hardcodes installers, and CoreInstaller includes mixed responsibilities.

Migration target:
- **Framework module registration:** registers Framework-owned services only.
- **MythHunter registrations:** registers game rules, content and application/presentation services.
- **Adapter registrations:** bind platform/provider implementations to abstractions.
- **Game Composition Root:** invokes these registrations in one documented order and performs validation/startup.
- **Unity entry host:** only delegates to the composition root and forwards lifecycle/shutdown where necessary.

Do not perform a bulk installer rewrite in this architecture task. In the implementation phase, migrate one registration group at a time and keep the project compiling after each step.

## Error and observability requirements
- Duplicate registrations must have defined behaviour (reject, replace explicitly, or multi-bind deliberately); no accidental last-registration-wins.
- Missing required registrations must fail fast with the service/module name and dependency context.
- Optional module registration may be skipped only when it is genuinely optional and not required by selected game configuration.
- Startup errors must be logged via the configured abstraction and surfaced to the caller/UI as appropriate.
- Registration order must be explicit where order affects behavior.
- Composition validation must not depend on Unity Editor assemblies at runtime.

## Forbidden patterns
- Framework code resolving `MythHunter.*` services.
- Multiple independent roots starting the same game.
- `GameBootstrapper`, `GameFlowManager`, and installers each independently acting as the authoritative composition/startup owner.
- Static registration lists that hardcode every future game and optional module.
- Global assembly scans that silently register unrelated types.
- Registration constructors with hidden side effects.
- Resolving concrete services from deep inside domain/gameplay code when they should be injected.
- Swallowing registration/initialization errors and continuing with a partial graph.

## Validation criteria
The strategy is considered implemented only when source code later demonstrates:
- one authoritative Game Composition Root;
- Framework registration is separate from MythHunter registration;
- adapter binding is explicit;
- no Framework → Game references;
- no runtime → Editor references;
- deterministic missing/duplicate registration semantics;
- deterministic initialization, cancellation and disposal;
- tests or validation checks for the composed dependency graph.

This document defines the target strategy; it does not claim these criteria currently pass.

## Technical debt / migration risks
- Existing overlapping startup/game-flow ownership must be reconciled during implementation.
- Existing installer registry and installer classes mix registration responsibilities.
- Async lifecycle and swallowed errors could conceal partial startup.
- DI scopes/lifecycle need validation; do not assume the current container fully satisfies the target.
- Unity-specific startup must remain a thin adapter around the application composition root.

## Conclusion
Use one authoritative MythHunter Game Composition Root. Framework modules provide explicit, isolated registrations; adapters bind platform-specific implementations; game code registers MythHunter behaviour. Composition is validated before the runtime starts, and startup/shutdown have explicit lifecycle and failure semantics.

No MythHunter source code was changed in this task.
