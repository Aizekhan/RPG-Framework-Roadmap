# EPIC 03.6 — Migrate MythHunter ECS consumers to Framework runtime

## Status
**COMPLETE.** PR #18 and PR #19 have both merged to `dev`; PR #19 merge commit `d3b81484f42d084cae150b5b60fc64185cfb70c2`. Refreshed migration CI #37/#238 and post-merge CI #38/#239 passed. Unity CLI EditMode on migration source `21d8d89938bc4c165aa485ae3abf96e003744cc1`, Unity `6000.0.45f1`: 9 passed, 0 failed, 0 skipped. User confirmed full Unity Console/project compile and game bootstrap/ECS smoke are OK. Historical source checks below remain as evidence.

## Parent and target
- Parent: EPIC 03 — First Reusable Framework Slice
- Depends on: EPIC 03.5 merged after Unity import/compile/EditMode tests pass.
- Target repository: Aizekhan/MythHunter
- Target starting point: updated `dev` after PR #18 merge; create a separate feature branch for this task.
- Do not modify `dev` directly.

## Implementation checkpoint (2026-10-10)
- Branch: `feature/epic-03-6-ecs-consumer-migration`.
- Draft stacked PR: [#19 — Migrate MythHunter ECS consumers to Framework runtime](https://github.com/Aizekhan/MythHunter/pull/19), now based on `dev` after PR #18 merged; diff isolates the migration.
- Completed source slice: added Framework imports to identified consumers (component cache/factory, world, systems, archetype/template code, serializers and entity factories); switched the composition-root binding to the Framework manager by removing the legacy class; removed duplicate `MythHunter.Core.ECS.IComponent` and `IEntityManager` definitions.
- Updated the Editor code generator to emit `using RPGFramework.ECS;`.
- Static validator now checks for duplicate legacy contract/runtime files, orphaned legacy metadata, explicit legacy contract references, and missing Framework imports in ECS consumers.
- CI run [#21](https://github.com/Aizekhan/MythHunter/actions/runs/38071622972) passed static checks and .NET ECS tests on the preliminary migration head. Follow-up review removed legacy ECS imports only where they were unused, preserving game-layer imports for `EcsWorld`, `IEcsWorld`, `ISystem`, `SystemBase`, and `ISerializableComponent`; the code generator's plain-component template now targets Framework `IComponent`. Workflow triggering was expanded to cover `Assets/_MythHunter/**` for both push and pull-request events. Latest branch CI [#35](https://github.com/Aizekhan/MythHunter/actions/runs/38073513821), on head `21d8d89938bc4c165aa485ae3abf96e003744cc1`, passed static validation and .NET tests. PR #19 body now records the scope, latest evidence, and remaining gates. User-confirmed Unity full compile/Console and game bootstrap/ECS smoke are OK; refreshed aligned-head GitHub CI is running.
- Latest-head Unity validation: on migration head `21d8d89938bc4c165aa485ae3abf96e003744cc1`, Unity CLI `6000.0.45f1` produced `EPIC-03-6-EditMode-latest.xml` at `2026-10-10 18:12:11Z`: total EditMode 9 passed, 0 failed, 0 skipped; `RPGFramework.ECS.Runtime.Tests.dll` discovered and all 8 `EntityManagerTests` passed, including invalid/destroyed ID rejection. This closes the rerun gate. User subsequently confirmed full Unity Editor/Console compilation and game bootstrap/ECS smoke are OK. Continue watching for legitimate gameplay call paths affected by invalid-ID behavior.

## Current source facts
- Framework assembly: `Assets/_Framework/ECS/Runtime`, namespace `RPGFramework.ECS`.
- Legacy contracts and implementation: `Assets/_MythHunter/Code/Core/ECS`, namespace `MythHunter.Core.ECS`.
- Legacy public contract files have existing Unity GUIDs; preserve metadata identity until the planned removal is explicitly reviewed.
- Legacy `EntityManager.AddComponent` creates backing storage for any supplied ID; Framework implementation throws `ArgumentException` for unknown/destroyed IDs. This behavior difference must be intentionally tested and handled, not hidden.
- `ComponentCache<T>`, registries, factories, archetypes/templates, serializers, entity factories, gameplay systems, installers and `IEcsWorld/EcsWorld` consume legacy ECS types.
- `EcsWorld` depends on MythHunter's `ISystemRegistry` and game lifecycle; it must remain game-layer code, not be moved into Framework.
- Current ECS IDs are integers. `Entity` value-type redesign is out of scope.

## Migration strategy
Do not perform a namespace-only global replacement. Move consumers in cohesive groups against the canonical Framework contracts and implementation, preserving gameplay APIs where possible. Since Unity's predefined Assembly-CSharp may reference an auto-referenced project asmdef, MythHunter/game code may call Framework; Framework must never reference Assembly-CSharp or MythHunter.

Before source edits on the branch:
1. Refresh the usage map against the merged PR head.
2. Inventory every remaining `MythHunter.Core.ECS.IComponent`, `IEntityManager` and concrete legacy `EntityManager` reference, including reflection, editor/generator and serializer code.
3. Create a rollback checkpoint/branch and confirm no unrelated local changes will be discarded.

## Suggested implementation slices
1. **Canonical contract cutover**: update the legacy contract source ownership so there is only one CLR identity for `IComponent` and `IEntityManager`; preserve call-site namespace compatibility only if it does not create wrapper/duplicate interfaces. Compile immediately.
2. **Runtime binding**: bind the Framework `IEntityManager` to Framework `EntityManager` in the game composition root. Do not bind two managers or permit parallel entity stores.
3. **Consumers**: migrate caches, cache registry/extensions, optimizer, component factory, entity/archetype/template factories, serialization contracts, systems and installers in small batches. Keep ComponentCache as a game-side optimization until evidence shows it can be generalized.
4. **World boundary**: keep `IEcsWorld`, `EcsWorld`, `ISystemRegistry` and lifecycle methods in MythHunter/Game Layer; change only their manager contract dependency.
5. **Remove legacy duplicates**: delete legacy `EntityManager.cs` and old contract definitions only after repository-wide references are zero and all consumers resolve the Framework types. Preserve unrelated file/asset GUIDs.
6. **Validation**: .NET tests, static forbidden-reference/GUID checks, Unity import/full compile, Unity EditMode tests, smoke-check game startup and key ECS paths.

## Acceptance criteria
- Exactly one runtime `IComponent`, `IEntityManager` and active entity store is used by MythHunter.
- Framework assembly has no references to MythHunter, UnityEngine/UnityEditor, game-specific phases, logger, DI, or providers.
- All existing consumers compile against the Framework API.
- No unresolved legacy contract/type references remain (including editor tooling, serialization, and reflection-based paths).
- Entity create/destroy, add/remove/get/query and invalid-ID behavior have focused test coverage.
- The intended behavior change for adding to unknown/destroyed IDs is documented and verified not to break legitimate call paths.
- Unity project compiles; EditMode tests pass; game bootstrap initializes and uses the single Framework manager.
- Roadmap STATUS and master plan reflect evidence, not assumptions.

## Risks and rollback
- Main risk: existing systems may rely on legacy manager behavior for arbitrary IDs or on concrete `MythHunter.Core.ECS.EntityManager` construction.
- Rollback: revert only this feature branch/PR; keep PR #18 merged as a standalone runtime. Never reset or discard unrelated local worktree changes.
- Do not start the next Framework module (DI/events/scheduling) until this migration is validated and duplicate ECS runtime is removed.
