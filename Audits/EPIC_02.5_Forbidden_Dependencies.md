# EPIC 02.5 — Forbidden Dependencies and Enforcement Rules

## Status
DONE

## Purpose
Turn the allowed dependency-direction policy into an explicit list of forbidden references and checks. These are target rules for extraction and future CI enforcement; they are not a claim that the current MythHunter source already passes them.

## Hard prohibitions

### Framework Core / contracts
The following are forbidden references from platform-neutral Framework contracts/primitives:
- Any `MythHunter.*` namespace or assembly.
- Game-specific heroes, lobby slots, class/race definitions, game phases, abilities, combat rules, concrete game events or content.
- `UnityEngine` and `UnityEditor` APIs.
- UI, views, presenters, controllers, or other presentation assemblies.
- Concrete cloud, analytics, authentication or transport provider SDKs.
- Concrete Unity runtime adapter implementations.
- Save-game or network protocol policies that belong to higher-level modules.

### Framework runtime
The following are forbidden references from reusable Framework runtime modules:
- Any `MythHunter.*` assembly.
- UI/presentation and concrete game content.
- `UnityEditor` in any runtime assembly.
- Concrete platform/provider adapters that should instead implement a Framework-facing contract.
- Hard-coded MythHunter scene names, resource keys, phase IDs or content identifiers.
- Unbounded implicit discovery that automatically imports concrete game types into Framework registries.

### Dependency inversion violations
The following reference directions are forbidden:
- Framework → Game Layer.
- Framework contracts → Framework implementations.
- Abstraction/contracts → adapter implementation.
- Domain/gameplay rules → presentation/UI implementation.
- Serialization → Persistence or Networking.
- ECS runtime → concrete gameplay systems/entities.
- Generic Systems registry/scheduler → MythHunter phase policy.
- Generic EventBus → concrete MythHunter domain events.
- Generic networking → concrete game event/content types.
- Runtime assemblies → any `*.Editor` assembly.
- Optional module → unrelated optional module without an explicit, justified contract.
- Any circular assembly/module reference.

### Ownership violations
The following are forbidden even where a reference could be technically compiled:
- Adding a game-specific concept to Framework merely to avoid fixing a dependency.
- Moving a contract into a general-purpose “Common” assembly without clear ownership and more than one justified consumer.
- Making Resources, Pooling, Replay, Networking, Cloud, UI, or Tools mandatory dependencies of Framework Core.
- Using static global service locators to bypass the composition root.
- Treating ComponentCache as a second authoritative ECS store.
- Using CLR type names or per-call generated identifiers as durable save/network schema identity.
- Allowing Editor validation/code generation to execute as a runtime dependency.

## Allowed exceptions
An exception is allowed only when all conditions are met:
1. The dependency has a named owner and documented purpose.
2. It does not create a cycle or reverse the Framework/Game boundary.
3. It is isolated in a clearly named adapter/integration module.
4. Runtime vs Editor separation remains enforceable.
5. The exception is documented in an architecture decision record before implementation.

The exception process must not be used to permit Framework Core → MythHunter or Framework runtime → Unity Editor.

## Enforcement checklist

For each new or changed assembly/module:
- [ ] Check direct references against the forbidden matrix.
- [ ] Check transitive dependencies where tooling supports it.
- [ ] Verify that Framework runtime has no `MythHunter.*` reference.
- [ ] Verify that Framework contracts/core do not reference `UnityEngine` or `UnityEditor`.
- [ ] Verify runtime assemblies do not reference `*.Editor`.
- [ ] Verify that UI/presentation is not referenced by Framework or domain rules.
- [ ] Verify that no cycles have been introduced.
- [ ] Verify optional modules remain optional.
- [ ] Verify explicit registrations replace broad implicit assembly scanning where appropriate.
- [ ] Record accepted exceptions and why a lower-level owner cannot safely own the dependency.

This checklist becomes mechanically enforceable only after assembly definitions and/or architecture tests are added. Until then, it is a review policy, not a passing test result.

## Based on audit findings
Known source areas that must be checked against these prohibitions during migration:
- EventBus references to concrete game events and Unity Editor.
- EcsWorld/SystemRegistry direction and game phase semantics.
- Core/Game and GameBootstrapper composition concerns.
- Installers and static InstallerRegistry mixing framework/game registrations.
- NetworkEventBus and network serialization identity coupling.
- PreloadManager concrete phase/scene/resource keys.
- DI validation mixing runtime and Editor concerns.
- ComponentCache duplicating ECS state.
- UnityEngine types found in low-level code candidates.

These are recorded audit findings and migration targets; they are not represented as fixed by this document.

## Conclusion
The prohibitions now state what cannot be referenced, who owns the integration boundary, how exceptions are reviewed, and what future automated checks must verify. No MythHunter source code was changed.
