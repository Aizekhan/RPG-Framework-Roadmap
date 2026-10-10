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
**Status: COMPLETE — assembly-reference direction verified**
- [x] Verify that predefined Unity assemblies can reference project asmdef assemblies when Auto Referenced is enabled.
- [x] Record the reverse restriction: an asmdef assembly cannot depend on types in Unity's predefined Assembly-CSharp.
- [x] Confirm that a broad MythHunter.Runtime.asmdef is not needed simply to introduce a pure Framework assembly.
- [x] Record the corrected rule in `Audits/EPIC_03.3_Assembly_Boundary_Reassessment.md`.

Conclusion: keep the initial Framework module dependency-free and do not move the entire MythHunter tree just to reference it.

## 3.4 — Compile-time dependency map and minimal assembly cut
**Status: COMPLETE — static map and first slice selected**
- [x] Map current IComponent/IEntityManager consumer clusters and the EcsWorld-to-ISystemRegistry game dependency.
- [x] Review the current source layout: MythHunter has no asmdef; Editor-sensitive files are mixed into runtime-looking folders.
- [x] Choose a standalone, pure .NET Framework ECS runtime under `Assets/_Framework/ECS/Runtime`, not a premature whole-game assembly migration.
- [x] Define a separate Editor-only test assembly under `Assets/_Framework/ECS/Tests`.
- [x] Record the transitional policy: legacy MythHunter ECS remains intact until a later consumer migration, then its duplicate implementation must be removed.
- [x] Record dependency inventory and boundary rationale in `Audits/EPIC_03.4_Assembly_Dependency_Inventory.md`.

Conclusion: static dependency mapping is sufficient to start the isolated runtime module. The source migration and Unity compile are tracked separately below.

## 3.5 — Implement and validate first Framework ECS runtime
**Status: MERGED to `dev` — PR #18, merge commit `b6e046895938d2dd72026827c4f72fd44ced1fdf`**
- [x] Add `RPGFramework.ECS.Runtime.asmdef` with no references and `noEngineReferences: true`.
- [x] Add initial `IComponent`, `IEntityManager` and `EntityManager` implementation in namespace `RPGFramework.ECS`.
- [x] Add eight focused NUnit tests for entity identity, component add/query/get/remove, destroy, missing components, and rejecting invalid entity IDs.
- [x] Add a .NET 8 test project that compiles the pure runtime/tests without loading Unity.
- [x] Add a GitHub Actions workflow for the framework runtime.
- [x] Add a static boundary validator for asmdef JSON, forbidden namespace references, Unity asset metadata and GUID uniqueness.
- [x] Preserve the original MythHunter interface files and their Unity GUIDs; assign distinct GUIDs to new Framework assets.
- [x] Run .NET CI after nullable cleanup: 7 passed, 0 failed, no compiler warnings.
- [x] Harden the new Framework EntityManager so AddComponent rejects unknown/destroyed IDs instead of creating phantom entities; add focused regression test.
- [x] Re-run .NET CI after the behavior fix: 8 passed, 0 failed, 0 skipped (including unknown and destroyed entity IDs).
- [x] Run static boundary validation in CI: OK; 3 runtime C# files, 1 test file, 7 Framework/test asset GUIDs checked.
- [x] Re-run GitHub Actions on current PR head `3917e6a48344d4469fd95a2049b1a2e60d983223`: static boundary OK; .NET compile and NUnit 8 passed, 0 failed, 0 skipped.
- [x] Remove five unrelated legacy Editor/runtime portability edits from this ECS PR.
- [x] Review current changed-file list: only Framework runtime/test/CI assets and expected legacy metadata/newline normalization remain.
- [x] Run Unity CLI EditMode invocation and inspect XML: the run passed but discovered only `AddressableAssets.DocExampleCode.TestStub.RequiredTest` (1 passed). This is unrelated to Framework ECS and does not satisfy the ECS Unity test gate by itself.
- [x] Run Unity CLI full EditMode tests on exact PR head `3917e6a48344d4469fd95a2049b1a2e60d983223` using Unity `6000.0.45f1`; XML `D:\\MythHunter-Git\\FrameworkECS-TestResults.xml` reports total 9 passed, 0 failed, 0 skipped.
- [x] Confirm `RPGFramework.ECS.Runtime.Tests.dll` was discovered and all eight `RPGFramework.ECS.Tests.EntityManagerTests` passed.
- [x] Complete final changed-file review: 20 files, limited to new Framework runtime/tests/CI/harness/validator and legacy metadata/newline normalization; no legacy ECS behavior cutover.
- [x] Document Unity EditMode results and follow-up migration in README/PR description; current PR-head CI runs #19 and #236 pass.

**Exit gate:** .NET tests pass; Unity imports and compiles the branch; the Unity test assembly runs; legacy MythHunter code has not been silently switched; the follow-up migration is explicit. Do not merge before Unity validation.

## 3.6 — Migrate MythHunter ECS consumers to Framework runtime
**Status: COMPLETE — PR #19 merged to `dev`, merge commit `d3b81484f42d084cae150b5b60fc64185cfb70c2`**
- [x] Create a dedicated migration branch and stacked draft PR #19 based on the 3.5 branch.
- [x] Move identified ECS consumers to import the Framework contract and manager APIs.
- [x] Change the MythHunter composition root to bind the Framework manager; remove duplicate legacy interface/manager source files on the migration branch.
- [x] Update editor code generation and static boundary validation for the new canonical namespace.
- [x] Run migration CI on preliminary head `aef44c3d9b393219eb299b4a1fb4870520e65131`: static boundary/import checks and .NET ECS tests passed.
- [x] Review compiler-facing references and remove legacy namespace imports only where unused, preserving imports for game-layer types.
- [x] Update the plain-component code-generation template to emit `using RPGFramework.ECS;` without the removed legacy component namespace.
- [x] Expand the ECS workflow path filters to include `Assets/_MythHunter/**` on the migration branch.
- [x] Run CI on migration head `f5ed3325a930c61eeab90e287898eb87355d886f` ([run #34](https://github.com/Aizekhan/MythHunter/actions/runs/38073434986)): static boundary/import checks and .NET ECS tests passed.
- [x] Include `Assets/_MythHunter/**` in both push and pull-request workflow path filters.
- [x] Run CI on latest migration head `21d8d89938bc4c165aa485ae3abf96e003744cc1` ([run #35](https://github.com/Aizekhan/MythHunter/actions/runs/38073513821)): static boundary/import checks and .NET ECS tests passed.
- [x] Update PR #19 description with current scope and validation gaps.
- [x] Inspect pre-cleanup `EPIC-03-6-EditMode.xml`: Framework ECS test assembly discovered; 8 ECS tests passed; total EditMode 9 passed, 0 failed, 0 skipped. This file is timestamped 17:36Z, before later source cleanup commits.
- [x] Rerun Unity EditMode on latest migration head `21d8d89938bc4c165aa485ae3abf96e003744cc1`: `EPIC-03-6-EditMode-latest.xml` at `2026-10-10 18:12:11Z`, total 9 passed, 0 failed, 0 skipped; all 8 Framework ECS tests discovered and passed.
- [x] User confirmed full MythHunter compilation/Unity Console is clean.
- [x] User confirmed game bootstrap/ECS smoke check is OK.
- [x] No reported Unity compile/import errors; latest-head EditMode passed.
- [x] Game bootstrap/ECS smoke reported OK; test suite covers unknown/destroyed-ID rejection.
- [x] After PR #18 merged, retarget PR #19 to `dev` and align branch ancestry in `fde1beba9940dc6647ac18244ee78fa973c8d31c` without changing the source tree.
- [x] Static boundary validator confirms no duplicate legacy ECS contracts, no explicit legacy interface references, and Framework imports for current consumers; preserve targeted manual review for future changes.
- [x] Refreshed migration CI #37/#238 passed; PR #19 merged, then post-merge CI #38/#239 passed. Recorded in `STATUS.md`.

**Outcome:** Exactly one ECS component contract, entity-manager contract and active entity store remain. Game-owned world/system lifecycle APIs stayed in MythHunter.
Detailed scope, acceptance criteria and rollback plan: [`Tasks/EPIC_03.6_ECS_Consumer_Migration.md`](Tasks/EPIC_03.6_ECS_Consumer_Migration.md).

# EPIC 04 — Framework Core Contracts and Shared Primitives
**Status: COMPLETE — source-backed candidate audit; no speculative shared assembly created**
- [x] Establish canonical ownership and callers before moving or renaming types
- [x] Keep UnityEditor, game phases, MythHunter types, concrete game events and providers out of neutral contracts
- [x] Avoid duplicate CLR types and update consumers coherently
- [x] Record audit at `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`.
- [x] Decision: don't create an empty `RPGFramework.Contracts`; current reusable ECS contracts have a canonical owner, while DI/events/logging/serialization/validation candidates need bounded, source-backed extractions.

# EPIC 05 — Framework Runtime Modules
## 5.1 — Logging and validation
**Status: COMPLETE — logging extracted and merged; generic validation extraction deliberately deferred by source audit**
- [x] Map `IMythLogger`, `MythLogger`, composition-root binding and consumer compatibility requirements.
- [x] Add `RPGFramework.Logging.ILogger` and `LogSeverity` as a separate no-engine-reference assembly.
- [x] Keep `IMythLogger` source-compatible; map severity/exception behavior in the adapter; bind the same logger instance under both interfaces.
- [x] Add focused Framework logging test; extend .NET harness, static boundary/GUID validator and CI path filters.
- [x] CI on source head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`: static boundary OK; 13 asset GUIDs checked; .NET 9 passed, 0 failed, 0 skipped; no compiler warnings.
- [x] Unity CLI EditMode `D:\\MythHunter-Git\\EPIC-05-1-Logging-EditMode.xml`, Unity `6000.0.45f1`: 10 passed, 0 failed, 0 skipped; `LoggerContractTests.Log_PreservesSeverityMessageCategoryAndException` passed.
- [x] User confirmed full project compile/Console and game bootstrap smoke are clean.
- [x] Merge PR #20 to `dev`: `ec967a2944ba83db69655d4415d416f49a2ce122`.
- [x] Audit `IValidator<T>`, `Validator<T>`, and `ValidationResult`; see `Audits/EPIC_05.1_Validation_API_Candidate_Audit.md`.
- [x] Decision: no Framework Validation extraction yet. Source search finds no real external production callers/construction sites; other validation helpers have different semantics. Do not create an unused module or migrate existing files.
- [x] Preserve a reopen condition: concrete caller demand, cross-module reuse, and explicit error/criticality/immutability semantics must be demonstrated first.

**Outcome:** EPIC 05.1 is closed. Proceed to EPIC 05.2.

## 5.2 — Dependency injection
**Status: ACTIVE — source audit recorded; characterization tests are next**
- [x] Map the current public surface and coupled concepts (`IDIContainer`, `DIScope`, `LazyDependency<T>`, `IDIInstaller`, lifecycle manager).
- [x] Record source risks: scoped resolution path, Type-based resolution/registration checks, scope hierarchy, disposal ownership, and current-scope concurrency.
- [x] Define bounded execution/acceptance criteria in `Tasks/EPIC_05.2_DI_Behavior_Characterization.md`.
- [ ] Add characterization/regression tests for singleton, transient, scoped, lazy and Type-based API behavior.
- [ ] Confirm defects with tests and fix the smallest set while preserving current MythHunter-facing APIs.
- [ ] Define lifetime, scope, disposal, injection and failure semantics before creating a neutral Framework DI module.
- [ ] Extract and migrate only after tests, compile, Unity EditMode and bootstrap validation pass.

Audit: `Audits/EPIC_05.2_DI_Candidate_Audit.md`.

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
- **M1 — First reusable boundary:** EPIC 03.2–03.4 complete; EPIC 03.5 validation active.
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
