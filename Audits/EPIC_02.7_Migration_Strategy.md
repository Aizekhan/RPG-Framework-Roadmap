# EPIC 02.7 — Migration Strategy

## Status
DONE

## Purpose
Define a controlled, reversible migration path from the current mixed MythHunter codebase to the target Framework/Game/Adapter boundaries. This is an execution strategy, not a record of completed source changes.

## Core decision
Use **incremental extraction with a compiling checkpoint after every small step**. Do not rewrite the whole codebase, perform a one-shot folder move, or extract modules based on folder names alone.

The source repository remains the implementation target:
- Repository: `Aizekhan/MythHunter`
- Branch: `dev`

This strategy document lives in `Aizekhan/RPG-Framework-Roadmap`. Architecture planning must not be mistaken for source implementation.

## Migration safety rules
1. Start from the exact current source branch and inspect the current Unity version, existing `.asmdef` files, package dependencies, project settings and test setup before changing code.
2. Preserve current gameplay behaviour unless a separate task explicitly changes it.
3. Establish a known-good baseline: compile, run available tests, record existing failures and save a recoverable Git checkpoint.
4. Change one dependency boundary or one cohesive module at a time.
5. After each step, compile the Unity project and run relevant tests; do not continue after a new failure without recording and resolving it.
6. Avoid bulk file/folder moves before ownership and references are understood.
7. Keep source-code evidence separate from architecture proposals and external reference material.
8. Never mark an extraction complete solely because files moved or an assembly definition was created.
9. Keep changes reversible through small commits and clearly stated rollback points.
10. Do not make Framework API/public compatibility promises before the contracts stabilize.

## Migration phases

### Phase 0 — Baseline and safeguards
- Record the current source branch/commit and working-tree state.
- Verify the project compiles in its existing Unity environment.
- Run existing tests and record baseline failures separately.
- Capture current assembly definitions, package dependencies, scripts excluded from compilation, and editor-only code boundaries.
- Create a known-good checkpoint before the first source-code modification.

Exit condition: current build/test status is known and recoverable.

### Phase 1 — Create enforceable boundaries
- Add only the minimum assembly definitions needed to isolate the first neutral Framework slice.
- Avoid creating one assembly per logical module by default.
- Put Unity Editor-only code behind Editor-only assembly boundaries.
- Confirm Framework runtime has no reference to MythHunter assemblies before further extraction.

Exit condition: the first boundary compiles and the forbidden-reference policy can be checked.

### Phase 2 — Extract pure contracts and primitives
- Identify genuine reusable contracts from the audit (for example, ECS, DI, events and serialization contracts).
- Move or redefine contracts only when their ownership is clear and their dependencies are neutral.
- Do not move concrete MythHunter event types, phase enums, hero rules, content or UI into Framework.
- Add focused tests for contract expectations where meaningful.

Exit condition: selected contracts compile independently of MythHunter and Unity-specific implementations, where platform-neutrality is required.

### Phase 3 — Extract runtime implementations one module at a time
Recommended sequence, adjusted only when real dependency evidence requires it:
1. logging and small validation primitives;
2. DI contracts/runtime after verifying lifecycle and scope behaviour;
3. event contracts and local event dispatch, separating middleware/pooling/network bridges as needed;
4. system contracts/lifecycle/scheduler without MythHunter phase policy;
5. ECS contracts/runtime after the storage and identity decisions are made;
6. serialization infrastructure with stable schema identity;
7. generic entity/template infrastructure after separating MythHunter content.

Each module must declare its dependencies, have tests for its key behaviour and pass the agreed validation before the next module is migrated.

Exit condition: each extracted module is independently understandable, has explicit dependencies, and passes available tests/build checks.

### Phase 4 — Establish adapter boundaries
- Move UnityEngine-dependent implementations to the appropriate Unity adapter or game-specific Unity assembly.
- Keep Unity resource keys, scene names, prefabs and content configuration in MythHunter-owned code.
- Keep cloud/auth/analytics SDKs behind optional provider adapters.
- Keep Editor tools and code generation out of runtime assemblies.
- Introduce adapters only for needed integrations; do not create speculative empty packages.

Exit condition: Framework runtime does not reverse-reference Unity/provider adapters; runtime-to-Editor references are absent.

### Phase 5 — Establish the Game composition root
- Introduce one authoritative MythHunter Game Composition Root.
- Separate Framework registrations, adapter bindings, and MythHunter registrations.
- Replace hardcoded static installer discovery with explicit, documented registration.
- Consolidate overlapping startup responsibilities only after mapping existing behaviour.
- Define initialization order, failure semantics, cancellation and disposal.
- Keep Unity bootstrap code thin and delegated to the Game Composition Root.

Exit condition: the game has one explicit composition path, missing/duplicate registrations are handled deterministically, and the build runs.

### Phase 6 — Move concrete Game Layer responsibilities
Move game-specific responsibilities from the mixed code only after the Framework contracts they consume are stable:
- application/game flow and concrete states;
- lobby and hero rules;
- concrete phase rules;
- combat and abilities;
- concrete domain events;
- game archetypes/templates/content;
- UI/presentation and game-specific settings;
- MythHunter authoring tooling.

Do not move these simply because a folder name matches the target map. Follow actual ownership and dependencies.

Exit condition: Framework no longer contains MythHunter domain behaviour, and the Game Layer compiles against public Framework APIs.

### Phase 7 — Remove duplicated or invalid ownership
Address audited debt in controlled steps:
- remove/isolate ComponentCache after proving how current callers use it;
- remove duplicate event bus abstractions only after behaviour and call paths are mapped;
- remove duplicated phase ownership between scheduler/registry and game phase systems;
- separate persistence serialization from network wire serialization;
- replace unstable external identifiers and inappropriate reflection discovery;
- stabilize asynchronous initialization, cancellation and shutdown;
- remove unnecessary references and validate the dependency graph.

These are separate refactoring tasks; do not combine them into a single large cleanup commit.

Exit condition: each debt item has evidence, tests/validation and a documented outcome.

### Phase 8 — Harden and validate the target architecture
- Add automated checks for prohibited project/assembly references.
- Test the Framework without MythHunter-specific assemblies.
- Build and run the MythHunter sample/game path.
- Test module registration and lifetimes.
- Verify optional modules are not required by the Framework Core.
- Record known limitations and unresolved technical debt.

Exit condition: the architecture is enforced by build-time and/or automated checks rather than documentation alone.

## Per-step checklist
A migration step cannot be marked complete until all applicable items are satisfied:

- [ ] Intent and target owner documented.
- [ ] Current callers/references identified.
- [ ] Expected API/behaviour changes stated.
- [ ] Small source diff reviewed.
- [ ] Unity compile succeeds.
- [ ] Relevant automated/manual tests pass, or pre-existing failures are explicitly distinguished.
- [ ] No new forbidden dependency or cycle is introduced.
- [ ] Rollback point exists.
- [ ] Roadmap and `STATUS.md` updated.

## Rollback and failure policy
- If compilation or a relevant test fails due to the new change, stop the migration sequence.
- Revert or repair that specific change before proceeding; do not stack more uncertain changes on top.
- Preserve unrelated user changes.
- Do not reset or force-push the source branch as a routine rollback.
- Record any baseline failure before attributing it to migration.
- If the intended boundary cannot compile without an undesirable dependency, revisit ownership/contracts instead of forcing a circular reference.

## Parallel work policy
Architecture documents may be drafted in parallel only when they do not change the active execution pointer. Source migrations that touch shared contracts, installers or assemblies should remain sequential unless module ownership and APIs are already frozen and work can be integrated safely.

## Immediate next actions after architecture baseline
1. Complete the remaining EPIC 02 architecture tasks.
2. Review the consolidated architecture baseline for contradictions and missing owners.
3. Inspect actual `.asmdef` and dependency evidence in MythHunter before changing source code.
4. Rebaseline the plan if actual project constraints differ from the current architecture proposal.
5. Begin Framework Extraction only after the architecture baseline is accepted and source baseline is known.

## Risks and caveats
- The audit reveals intertwined responsibilities, reflection-based discovery, duplicate abstractions and incomplete networking; incremental extraction is necessary to limit regressions.
- The current source's exact Unity version, compilation status, assembly-definition layout and test coverage must be checked at migration start; this document does not claim those facts have already been validated.
- Some target module boundaries may need to be adjusted to match actual compile-time dependencies.
- Networking, replay, cloud integrations and developer tools remain optional and must not delay the first reusable Framework Core extraction unless they are required by a selected game path.

## Conclusion
Migrate incrementally from a known-good baseline, extract stable contracts first, introduce enforceable boundaries, then move implementations and game-specific ownership in small compiling steps. Each step must be testable, reversible and documented. No MythHunter source code was changed by this planning task.
