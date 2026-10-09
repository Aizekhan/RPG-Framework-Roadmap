# RPG Framework Master Plan

## Master Objective
Transform MythHunter into a reusable, modular RPG Framework plus a concrete Game Layer, then build universal RPG modules on top of that stable foundation.

## Working Rules
1. Work strictly top to bottom through the currently ACTIVE item.
2. Exactly one roadmap item may be ACTIVE at a time.
3. Audit first, architecture second, implementation third.
4. Keep source-code evidence separate from architecture proposals and external reference material.
5. Every completed item requires a documented conclusion and applicable validation evidence.
6. This file defines execution order; `STATUS.md` is the exact execution pointer.
7. Re-plan after material architecture or source/build discoveries.
8. Do not generalize systems before ownership, dependencies and contracts are stable.
9. Every source migration step needs a compiling checkpoint, relevant tests, and a rollback point.
10. A file move, namespace change, or assembly definition alone does not prove extraction is complete.
11. Optional modules must not become mandatory Framework Core dependencies without a documented reason.
12. Do not mark source implementation complete based only on GitHub document edits; Unity compile/test evidence must be recorded.

# EPIC 01 — Full MythHunter Audit
**Status: COMPLETE**

Goal: Understand the actual source architecture and identify Framework/Game ownership.

### 1.1 — Top-level structures
- [x] Core
- [x] Components
- [x] Entities
- [x] Systems
- [x] Events
- [x] Networking
- [x] Cloud
- [x] UI
- [x] Services
- [x] Utils

### 1.2 — Core
- [x] ECS
- [x] DI
- [x] Game
- [x] Installers
- [x] StateMachine
- [x] SceneManagement
- [x] Validation

### 1.3 — Components
- [x] Core
- [x] Character
- [x] Combat
- [x] Movement
- [x] Lobby
- [x] Other components

### 1.4 — Entities
- [x] EntityFactory
- [x] Archetypes
- [x] Templates
- [x] Serialization Registry

### 1.5 — Systems
- [x] Core Systems
- [x] Gameplay Systems
- [x] Hero Systems
- [x] Lobby Systems
- [x] Phase Systems
- [x] System Groups

### 1.6 — Events
- [x] EventBus
- [x] Event Types
- [x] Event Pipeline
- [x] Middleware
- [x] Network Events

### 1.7 — Networking
- [x] Client
- [x] Server
- [x] Messages
- [x] Serialization
- [x] Security

### 1.8 — Final audit report
- [x] Framework modules
- [x] Game Layer modules
- [x] Technical debt list
- [x] Dependency map
- [x] Unnecessary dependency list
- [x] Refactoring candidates
- [x] Core/Game
- [x] Gameplay installers
- [x] Phase systems
- [x] CombatSystem
- [x] ComponentCache
- [x] EntityManager storage layer

# EPIC 01.5 — Audit Completeness Check
**Status: COMPLETE**

Goal: Include major structures missed by the initial detailed audit.
- [x] Replay
- [x] Resources / Preload / Pool
- [x] Cloud / External Services
- [x] Debug / Developer Tools
- [x] Authoring / Editor Tooling
- [x] Utils / Logging / Validation
- [x] Final completeness verdict

# EPIC 02 — Architecture Rebuild
**Status: COMPLETE**

Goal: Define the target architecture and migration policy before source implementation.
- [x] Freeze Framework/Game boundary
- [x] Define target module map
- [x] Define assembly/package boundaries
- [x] Define allowed dependency direction
- [x] Define forbidden dependencies
- [x] Define composition-root strategy
- [x] Define migration strategy
- [x] Final architecture baseline

# EPIC 02.9 — Post-Architecture Roadmap Reassessment
**Status: ACTIVE — planning gate; no source implementation may pass this gate until completed.**

Goal: Rewrite and sequence the remaining epics to remove duplicated scope and establish dependency-ordered milestones.

- [x] Identify overlap between EPIC 03, 05, 06, and 07
- [x] Identify optional capabilities that must not block Framework Core
- [x] Define milestone-level execution approach
- [ ] Map every remaining task to owner, prerequisite, evidence, and exit criteria
- [ ] Replace old EPIC 03–14 ordering with the revised execution sequence
- [ ] Align `STATUS.md` with the final revised plan
- [ ] Verify roadmap consistency and record conclusion

# EPIC 03 — Baseline and Enforceable Boundary
**Goal:** Establish a known build/test baseline and create the first small, enforceable Framework/Game boundary.

### 3.1 — Source baseline
- [ ] Record MythHunter source branch and commit
- [ ] Record working-tree state and preserve unrelated user changes
- [ ] Record Unity version and package manifest/lockfile state
- [ ] Inventory all existing assembly definitions and compilation exclusions
- [ ] Run Unity compilation and available tests; record baseline failures separately
- [ ] Create a recoverable checkpoint before source changes

**Exit gate:** baseline compilation/test state and rollback point are documented. If local Unity validation is unavailable, stop source migration and record the blocker.

### 3.2 — Boundary design against actual project constraints
- [ ] Build a source-backed dependency map for the first candidate slice
- [ ] Verify Unity assembly-definition and package constraints against the actual project
- [ ] Choose the smallest viable first Framework-owned assembly/slice
- [ ] Confirm no Framework-to-MythHunter reference is required by the proposed boundary
- [ ] Record the chosen boundary and alternatives rejected

**Exit gate:** the first slice is justified by source evidence; no code has been moved yet in this task.

### 3.3 — First enforceable boundary
- [ ] Add the minimal required assembly definition(s)
- [ ] Isolate Editor-only scripts where required for the boundary
- [ ] Compile Unity project
- [ ] Run relevant tests
- [ ] Check for assembly cycles and forbidden references
- [ ] Document a rollback point and outcome

**Exit gate:** the first boundary compiles, tests pass or baseline failures are explicitly separated, and Framework code does not reference MythHunter.

# EPIC 04 — Framework Core Contracts and Shared Primitives
**Goal:** Extract only stable platform-neutral contracts needed by the first reusable Framework slice.

- [ ] Define minimal shared primitives and ownership
- [ ] Identify canonical types and all callers before moving/renaming
- [ ] Extract contracts into the owning Framework assembly without creating duplicate CLR types
- [ ] Keep UnityEngine/UnityEditor, MythHunter types, game phases, concrete game events and providers out of neutral contracts
- [ ] Update consumers and source generators/codegen templates coherently
- [ ] Compile and run focused tests after each cohesive migration step
- [ ] Enforce forbidden-reference rules mechanically where feasible

**Exit gate:** contracts compile inside their intended Framework ownership, consumers use the canonical types, and no duplicate type/assembly-cycle problem is introduced.

# EPIC 05 — Framework Runtime Modules
**Goal:** Extract reusable runtime implementations after their contracts, ownership, and required dependencies are stable.

## 5.1 — Logging and validation primitives
- [ ] Separate neutral logging/validation abstractions from MythHunter/Unity implementations
- [ ] Decide owner and boundary of existing logger and validation utilities
- [ ] Add focused tests and compile

## 5.2 — Dependency injection
- [ ] Audit registration, lifetime, scope, resolution, disposal, and async lifecycle behaviour
- [ ] Stabilize DI contracts and remove concrete game/logger dependencies from neutral surfaces
- [ ] Extract DI runtime into the chosen assembly
- [ ] Test registration errors, lifetimes, scopes, disposal and cancellation

## 5.3 — Event dispatch
- [ ] Separate generic event contracts from concrete MythHunter events
- [ ] Decide dispatch ordering, error handling, subscriptions and disposal semantics
- [ ] Separate local dispatch from network/replay bridges and middleware ownership
- [ ] Remove duplicate EventBus only after mapping all call paths
- [ ] Test ordering, exceptions, subscription disposal and async cancellation

## 5.4 — Systems lifecycle and scheduler
- [ ] Separate generic system lifecycle/scheduling from MythHunter phase policy
- [ ] Decide initialization/update/shutdown ordering and failure semantics
- [ ] Break EcsWorld/SystemRegistry coupling where needed
- [ ] Test lifecycle ordering, cancellation and disposal

## 5.5 — ECS runtime
- [ ] Decide identity, entity/component lifecycle, storage, archetype and query model before extraction
- [ ] Map current callers and cache/registry behaviour
- [ ] Resolve ComponentCache ownership/duplication with evidence
- [ ] Extract ECS contracts/runtime one cohesive slice at a time
- [ ] Test entity/component operations, storage, query correctness and lifecycle
- [ ] Validate performance only after correctness and API stability

**Exit gate:** each selected runtime module has explicit dependencies, focused tests and a successful compiling checkpoint. ECS storage/identity decisions are recorded before their implementation is moved.

# EPIC 06 — Serialization Foundation
**Goal:** Separate generic codecs/schema identity from persistence orchestration and network wire protocols.

- [ ] Audit current component, entity, versioned, delta, persistence and network serializers separately
- [ ] Define stable schema/type identifiers independent of CLR names where data crosses process/version boundaries
- [ ] Separate generic serialization contracts from persistence save/load policy
- [ ] Separate network wire format/versioning from persistence serialization
- [ ] Extract only the contract/runtime portion proven reusable
- [ ] Add compatibility/version/migration tests as applicable
- [ ] Compile and validate consumers

**Exit gate:** persistence and networking do not own or redefine the generic serializer contract inconsistently; schema evolution risks and migration limits are documented.

# EPIC 07 — MythHunter Game Layer and Composition
**Goal:** Move concrete application/domain/gameplay ownership into MythHunter-owned assemblies and integrate through one authoritative composition root.

## 7.1 — Composition root
- [ ] Map existing bootstrap, installers, registries and startup paths
- [ ] Define one authoritative Game Composition Root
- [ ] Separate Framework module registration, adapter binding and MythHunter registrations
- [ ] Define deterministic initialization failure, cancellation and disposal
- [ ] Validate missing/duplicate registrations and compile/run the game path

## 7.2 — Application and content ownership
- [ ] Extract concrete states, scene flow and application/game flow
- [ ] Extract Lobby and Hero rules
- [ ] Extract MythHunter phase rules from generic systems infrastructure
- [ ] Extract Combat/Abilities and other concrete gameplay rules
- [ ] Keep concrete domain events, archetypes, templates, settings and content in Game Layer
- [ ] Move game-specific UI/presentation and authoring to their appropriate boundaries
- [ ] Compile and validate game behaviour at each cohesive step

**Exit gate:** MythHunter depends on public Framework APIs; Framework does not depend on MythHunter; game behaviour is preserved unless a separately approved task changes it.

# EPIC 08 — Dependency Debt and Lifecycle Stabilization
**Goal:** Remove audited coupling after relevant ownership boundaries are established, rather than mixing bulk cleanup into extraction.

- [ ] Remove cyclic and unnecessary dependencies one at a time
- [ ] Complete duplicate EventBus cleanup if not already resolved in EPIC 05.3
- [ ] Complete ComponentCache cleanup if not already resolved in EPIC 05.5
- [ ] Replace implicit assembly/service discovery with explicit registration where required
- [ ] Stabilize reflection/dynamic hot paths when evidence justifies change
- [ ] Stabilize async lifecycle, cancellation and shutdown
- [ ] Validate dependency graph and prohibited references automatically
- [ ] Document each debt item outcome and regression/rollback evidence

**Exit gate:** each cleanup has source evidence, tests/compile validation and no untracked regression. Do not repeat tasks already completed in EPIC 05.

# EPIC 09 — Optional Runtime Services and Adapters
**Goal:** Extract reusable services as opt-in capabilities, keeping provider/platform implementations outside neutral Framework modules.

## 9.1 — Resources, preload and scene loading
- [ ] Define resource and scene-loading abstractions that are genuinely platform-neutral
- [ ] Separate MythHunter resource keys, scene names, phase policy and content configuration
- [ ] Implement/retain Unity resource and scene adapters in Unity-specific assemblies
- [ ] Test error, cancellation and preload lifecycle

## 9.2 — Pooling
- [ ] Define pooling abstraction only if a real consumer requires it
- [ ] Keep pooling optional and avoid making ECS/Core require it
- [ ] Test acquire/release and lifecycle semantics

## 9.3 — Replay and developer diagnostics
- [ ] Define replay ownership against stable event and serialization contracts
- [ ] Keep debug tools out of runtime Framework Core
- [ ] Separate generic Framework tooling from MythHunter authoring
- [ ] Add tests/validation only for implemented capabilities

## 9.4 — Cloud/provider integrations
- [ ] Keep authentication, cloud data and analytics optional
- [ ] Define provider-neutral boundaries only where more than one game/provider needs them
- [ ] Isolate concrete SDK integrations in provider adapters
- [ ] Test adapter boundary without making Core depend on providers

**Exit gate:** every shipped service is optional, has an explicit dependency contract and lives in the appropriate runtime/adapter/game/editor owner.

# EPIC 10 — RPG Foundation Modules
**Goal:** Build universal RPG domain modules above the stable Framework, not inside the low-level infrastructure.

- [ ] Stats and attributes
- [ ] Resources
- [ ] Statuses
- [ ] Buffs and debuffs
- [ ] Damage and resistances
- [ ] Death/life-state rules
- [ ] Define data/API ownership and interactions before implementation
- [ ] Add deterministic domain tests
- [ ] Validate modules without MythHunter-specific content

**Exit gate:** modules remain game-agnostic, depend only on documented lower-level APIs and have tests for domain rules.

# EPIC 11 — Gameplay Modules
**Goal:** Create reusable gameplay capabilities as optional domain modules.

- [ ] Inventory
- [ ] Items
- [ ] Equipment
- [ ] Loot
- [ ] Abilities
- [ ] Combat
- [ ] Progression
- [ ] Cooldowns
- [ ] Define explicit module dependencies and avoid cycles
- [ ] Add domain tests and integration tests for chosen combinations

**Exit gate:** modules can be selected independently where designed, do not force unrelated capabilities into Core, and do not contain MythHunter-specific policies/content.

# EPIC 12 — Persistence
**Goal:** Implement durable save/load functionality using the serialization foundation, with explicit schemas and migration policy.

- [ ] Define save/load lifecycle, storage abstraction and failure semantics
- [ ] Define schema versioning and migration strategy
- [ ] Implement storage adapters only for selected platforms/providers
- [ ] Test roundtrip, compatibility, interrupted operations and migration
- [ ] Keep network packet/protocol responsibilities outside persistence
- [ ] Document guarantees and unsupported cases

**Exit gate:** persistence is tested independently and does not require a networking module.

# EPIC 13 — Networking
**Goal:** Build networking as an independent optional Framework module with explicit wire contracts and security boundaries.

- [ ] Audit the actual status of existing client/server/transport code before reuse
- [ ] Define transport/session lifecycle
- [ ] Define commands, snapshots and replication responsibilities
- [ ] Define stable network IDs and wire schema/versioning
- [ ] Define authentication/authorization and replay-protection requirements where applicable
- [ ] Keep concrete game-domain events/content outside generic networking
- [ ] Add protocol, lifecycle and security tests
- [ ] Validate networking can be omitted from offline games

**Exit gate:** networking is optional, wire identity is stable, security limitations are documented, and it does not depend on persistence implementation details.

# EPIC 14 — Tools, Sample and Release
**Goal:** Make the Framework understandable, verifiable and reusable in a new RPG project.

## 14.1 — Tooling
- [ ] Framework dependency/module graph
- [ ] Entity/component/system inspectors where supported by public APIs
- [ ] Event/network/save diagnostics only for modules that exist
- [ ] Isolate Unity Editor code from runtime assemblies
- [ ] Keep MythHunter-specific authoring tools Game-owned

## 14.2 — Adoption and release
- [ ] Framework and API documentation
- [ ] New-project template
- [ ] Module template guidance
- [ ] Small sample RPG independent of MythHunter game content
- [ ] Automated architecture/build/test validation
- [ ] Final dependency and API review
- [ ] Release candidate and known-limitations document

**Exit gate:** a new project can consume the Framework and selected modules without copying MythHunter-specific code; published capabilities are tested and documented.

---

## Milestone map
- **M0 — Baseline:** EPIC 03.1 complete.
- **M1 — Enforceable boundary:** EPIC 03.2–03.3 complete.
- **M2 — Reusable Core:** EPIC 04–06 complete for the explicitly selected Core scope.
- **M3 — MythHunter integration:** EPIC 07–08 complete with a working game path.
- **M4 — Optional capabilities:** EPIC 09, 12 and 13 as selected; no optional module blocks Core.
- **M5 — RPG modules and adoption:** EPIC 10–11 and 14 complete for the intended release scope.

## Dependency and validation rules
- Every epic/task must have an owner, prerequisites, source evidence, tests/compile requirements and an exit gate.
- A task can be marked complete only if its documented outcome and applicable compile/test evidence exist.
- Run Unity compilation and relevant tests after each cohesive source change.
- Stop after a migration regression; repair or revert the specific step before continuing.
- Framework contracts/Core must not reference MythHunter, UI, UnityEditor or platform/provider implementations.
- Adapters depend on Framework abstractions; Framework never references adapters.
- Optional modules remain optional; avoid speculative empty assemblies/packages.
- Keep persistence and network wire serialization separate.
- Any future reordering must be documented with evidence and reflected in this file and STATUS.md.
