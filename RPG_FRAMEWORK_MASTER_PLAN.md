# RPG Framework Master Plan

## Master Objective
Transform MythHunter into a reusable, modular RPG Framework plus a concrete Game Layer, then build universal RPG modules on top of that stable foundation.

## Working Rules
1. Exactly one roadmap item may be ACTIVE.
2. Prefer short, source-backed decisions that unblock implementation over repeated diagnostics.
3. Keep source evidence separate from architectural proposals.
4. A source task must define its target, owner, acceptance criteria and rollback approach.
5. Do not claim compilation or tests passed unless the relevant results demonstrate that.
6. Preserve existing user changes; never reset or discard local worktree changes.
7. Framework Core must not reference MythHunter, UnityEditor, game-specific policies or provider adapters.
8. Adapters depend on Framework; Framework never depends on adapters.
9. Avoid broad rewrites and speculative empty assemblies. Extract one small cohesive slice at a time.
10. After every source change, compile and run focused tests; do not treat repository documentation as implementation evidence.
11. Optional modules must not become mandatory Core dependencies without a documented reason.

# EPIC 01 — Full MythHunter Audit
**Status: COMPLETE**
- [x] Audit top-level structures: Core, Components, Entities, Systems, Events, Networking, Cloud, UI, Services, Utils
- [x] Audit Core: ECS, DI, Game, Installers, StateMachine, SceneManagement, Validation
- [x] Record Framework/Game candidates, technical debt, dependencies and unnecessary coupling

# EPIC 01.5 — Audit Completeness Check
**Status: COMPLETE**
- [x] Replay
- [x] Resources / Preload / Pool
- [x] Cloud / External Services
- [x] Debug / Developer Tools
- [x] Authoring / Editor Tooling
- [x] Utils / Logging / Validation
- [x] Final completeness verdict

# EPIC 02 — Architecture Rebuild
**Status: COMPLETE**
- [x] Framework/Game boundary and target module map
- [x] Assembly/package boundary principles
- [x] Dependency direction and forbidden references
- [x] Composition-root and migration strategy
- [x] Final architecture baseline

# EPIC 02.9 — Post-Architecture Roadmap Reassessment
**Status: COMPLETE**
- [x] Remove overlapping work across former EPIC 03–14
- [x] Keep optional capabilities out of Framework Core
- [x] Establish dependency-ordered milestones and exit criteria
- [x] Align execution pointer with STATUS.md
- [x] Record conclusion in `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`

# EPIC 03 — First Reusable Framework Slice
**Goal:** Begin extracting reusable code with a small, source-backed ECS boundary instead of extending baseline diagnostics indefinitely.

## 3.1 — Source baseline
**Status: EVIDENCE RECORDED — PARTIAL**
- [x] Record remote/local-reported branch and commit: `dev` / `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`
- [x] Record Unity version: `6000.0.45f1`
- [x] Inventory assembly definitions: six under UniTask, none under `Assets/_MythHunter`
- [x] Record local test result: one third-party Addressables EditMode stub passed; PlayMode discovered zero tests
- [x] Record limitations: clean forced compilation not demonstrated; no existing MythHunter C# tests discovered; local working tree has setup changes
- [x] Preserve local recovery materials under `D:\RPG-Framework-Baseline` (partial patch/report bundle; not represented as full backup)
- [x] Confirm no MythHunter source was changed during the audit

Conclusion: do not spend additional time on baseline investigation unless a specific error blocks the next bounded code change. Compile and create focused tests during implementation. Preserve local changes and do not reset the worktree.

## 3.2 — First Framework slice decision and usage map
**Status: EVIDENCE RECORDED — READY TO START FIRST SOURCE CHANGE**
- [x] Review current ECS contracts and immediate dependencies
- [x] Record first-slice decision in `Audits/EPIC_03.2_First_Framework_Slice_Decision.md`
- [x] Map observed usages of `Entity`, raw entity IDs, `IComponent`, `IEntityManager`, `IEcsWorld`, and `ISystemRegistry`
- [x] Record current behavior and compatibility constraints
- [x] Decide initial identity strategy: preserve integer IDs; defer any Entity value-type migration
- [x] Define first slice: canonical `IComponent` + `IEntityManager` contracts, no game/system/runtime implementation
- [x] Specify compile/test/rollback acceptance criteria in `Audits/EPIC_03.2_ECS_Contract_Usage_Map.md`

Conclusion: usage map and API constraints recorded. Proceed to EPIC 03.3. Unity is not required for design or GitHub source inspection; it is required only when validating source/assembly integration.

## 3.3 — Assembly boundary reassessment
**Status: COMPLETE — original two-interface extraction approach rejected as incomplete**
- [x] Identify that the existing predefined assembly cannot directly reference the new user asmdef
- [x] Identify that the MythHunter tree mixes UnityEngine runtime code and Editor-specific files/direct imports
- [x] Record correction in `Audits/EPIC_03.3_Assembly_Boundary_Reassessment.md`

Conclusion: do not merge the current draft as a completed extraction. The assembly plan needs a source-backed map of runtime/editor compilation boundaries before the first integrated assembly is added.

## 3.4 — Compile-time dependency map and minimal assembly cut
**Status: ACTIVE**
- [x] Keep the initial extraction isolated in draft PR #18; do not merge it
- [x] Record the assembly-reference limitation and Editor/runtime contamination in `Audits/EPIC_03.3_Assembly_Boundary_Reassessment.md`
- [x] Record current dependency clusters and known Editor crossings in `Audits/EPIC_03.4_Assembly_Dependency_Inventory.md`
- [x] Record a partial source-backed inventory of cross-boundary dependencies in `Audits/EPIC_03.4_Assembly_Dependency_Inventory.md`
- [ ] Complete the file-level runtime/editor assembly inventory and choose the smallest acyclic layout that existing consumers can actually reference
- [ ] Replace the contract-only PR #18 staging experiment with the selected integrated assembly cut; preserve source asset GUIDs and related metadata
- [ ] Add focused tests; confirm a real compile in Unity before merge

**Exit gate:** there is a valid, documented assembly dependency graph; the proposed source moves form a coherent slice and can be verified in Unity.

# EPIC 04 — Framework Core Contracts and Shared Primitives
- [ ] Establish canonical ownership and callers before moving or renaming types
- [ ] Keep UnityEditor, game phases, MythHunter types, concrete game events and providers out of neutral contracts
- [ ] Avoid duplicate CLR types and update consumers coherently
- [ ] Add compile/test/forbidden-reference validation

# EPIC 05 — Framework Runtime Modules
## 5.1 — Logging and validation
- [ ] Separate neutral abstractions from MythHunter/Unity implementations
- [ ] Choose owners and contracts from actual consumers
- [ ] Add focused tests and compile

## 5.2 — Dependency injection
- [ ] Specify registration, lifetime, scope, resolution, disposal and async lifecycle
- [ ] Remove concrete game/logger dependencies from neutral surfaces
- [ ] Extract runtime module and test lifecycle/error cases

## 5.3 — Event dispatch
- [ ] Separate generic event contracts from concrete MythHunter events
- [ ] Specify ordering, exception handling, subscription disposal and cancellation
- [ ] Keep network/replay bridges and middleware as explicit adapters
- [ ] Remove duplicate EventBus only after all call paths are mapped

## 5.4 — System lifecycle and scheduling
- [ ] Separate generic scheduling/lifecycle from MythHunter phase policy
- [ ] Specify initialization, update, shutdown and failure semantics
- [ ] Decouple ECS world from the Game-owned system registry where required
- [ ] Test lifecycle order and disposal

## 5.5 — ECS runtime
- [ ] Decide identity, lifecycle, storage, archetype and query model before extraction
- [ ] Resolve ComponentCache ownership with evidence
- [ ] Extract cohesive slices with correctness tests
- [ ] Evaluate performance only after API correctness and stability

# EPIC 06 — Serialization Foundation
- [ ] Audit component/entity/versioned/delta/network serialization separately
- [ ] Define stable schema/type identifiers when data crosses version/process boundaries
- [ ] Separate generic serialization from persistence policy and network wire formats
- [ ] Extract only proven-reusable contracts/runtime and test compatibility

# EPIC 07 — MythHunter Game Layer and Composition
- [ ] Map bootstrap/installers/registries/startup paths
- [ ] Establish one authoritative Game Composition Root
- [ ] Keep concrete states, phase policies, Hero/Lobby/Combat rules, domain events, archetypes, content and game UI in the Game Layer
- [ ] Separate Framework registration, adapter binding and game registrations
- [ ] Validate startup, shutdown and the working game path

# EPIC 08 — Dependency Debt and Lifecycle Stabilization
- [ ] Remove remaining cycles and unnecessary dependencies one at a time
- [ ] Avoid repeating fixes already handled during module extraction
- [ ] Stabilize async cancellation/shutdown and reflection paths when evidence warrants it
- [ ] Enforce forbidden references mechanically where feasible

# EPIC 09 — Optional Runtime Services and Adapters
- [ ] Resources/preload/scene loading
- [ ] Pooling (optional; only where a consumer needs it)
- [ ] Replay and developer diagnostics
- [ ] Cloud/provider integrations
- [ ] Keep Unity/platform/provider implementations outside neutral contracts

# EPIC 10 — RPG Foundation Domain Modules
- [ ] Stats and attributes
- [ ] Resources
- [ ] Statuses, buffs and debuffs
- [ ] Damage and resistances
- [ ] Death/life-state rules
- [ ] Define interactions and test domain behavior without MythHunter content

# EPIC 11 — Optional Gameplay Modules
- [ ] Inventory, items and equipment
- [ ] Loot, abilities and combat
- [ ] Progression and cooldowns
- [ ] Define explicit dependencies and test selectable combinations

# EPIC 12 — Persistence
- [ ] Define save/load lifecycle, storage abstraction, schema versioning and migrations
- [ ] Test roundtrip, compatibility and interrupted operations
- [ ] Keep network protocol concerns outside persistence

# EPIC 13 — Optional Networking
- [ ] Audit actual client/server implementation before reuse
- [ ] Define transport/session lifecycle, commands/snapshots/replication and stable wire schema
- [ ] Define security boundaries and tests
- [ ] Prove offline games can omit networking

# EPIC 14 — Tools, Sample and Release
- [ ] Dependency/module graph and relevant inspectors/diagnostics
- [ ] Runtime/editor assembly separation
- [ ] Framework API documentation and new-project/module templates
- [ ] Small sample RPG independent of MythHunter game content
- [ ] Automated architecture/build/test validation and release-candidate review

---

## Milestone map
- **M0 — Baseline evidence:** EPIC 03.1 recorded; no additional diagnostic loop.
- **M1 — First reusable boundary:** EPIC 03.2 recorded, EPIC 03.3 active.
- **M2 — Reusable Core:** EPIC 04–06 for selected scope.
- **M3 — MythHunter integration:** EPIC 07–08.
- **M4 — Optional capabilities:** EPIC 09, 12 and 13 as selected.
- **M5 — RPG modules and adoption:** EPIC 10–11 and 14.

## Dependency and validation rules
- Exactly one active item, tracked in STATUS.md.
- Every source change preserves unrelated local changes and has a rollback plan.
- Compile and run relevant focused tests after each cohesive source change; report actual results.
- A documentation commit is not source implementation.
- Framework/Core must not reference MythHunter, UnityEditor or adapters.
- Adapters depend on Framework; Framework never depends on adapters.
- Do not create speculative empty modules or perform a broad ECS rewrite.
- Keep persistence schema separate from network wire serialization.
