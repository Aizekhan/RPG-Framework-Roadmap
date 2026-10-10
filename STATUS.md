# Project Status

## Source Project
This roadmap controls refactoring of **Aizekhan/MythHunter** (Unity project, branch `dev`).

- Roadmap: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`
- Source baseline: `Audits/EPIC_03.1_Source_Baseline.md`
- First-slice decision: `Audits/EPIC_03.2_First_Framework_Slice_Decision.md`
- ECS usage map: `Audits/EPIC_03.2_ECS_Contract_Usage_Map.md`

## Current Position
- Epic: 03 — First Reusable Framework Slice
- Active task: 3.4 — Compile-time dependency map and minimal assembly cut
- Status: ACTIVE
- Objective: identify the smallest acyclic Unity assembly layout that permits Framework contracts to be consumed without pulling Editor/game dependencies into Framework Core.
- Unity use: not needed for further architecture/source inspection on GitHub. Needed when the new assembly/source integration is validated locally.
- Source experiment is isolated in [draft PR #18](https://github.com/Aizekhan/MythHunter/pull/18). It is intentionally not merge-ready: the predefined assembly cannot directly reference the newly introduced asmdef, and no Unity compile was run against the branch.

## Evidence already recorded
- Source branch/commit reported locally: `dev` / `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`.
- Unity version: `6000.0.45f1`.
- Six assembly definitions under `Assets/Plugins/UniTask`; none found under `Assets/_MythHunter`.
- No test C# files found under `Assets`; existing Unity results were one unrelated Addressables stub test passing and zero PlayMode test cases.
- Unity CLI reported “script recompilation was not required”; this is not a forced clean compile.
- Local worktree contains Unity/CLI setup changes. A patch/test-result/status bundle is saved under `D:\RPG-Framework-Baseline`; it is a partial recovery bundle and must be preserved.

## Current constraints
1. Do not reset, clean, or discard the user's local worktree.
2. Do not add `EcsWorld`, `EntityManager`, caches, archetypes, serializers, concrete components or game systems to the first Framework assembly.
3. Preserve `IComponent` / `IEntityManager` API identity; move the canonical definitions, do not copy them.
4. Keep the existing namespace initially if it avoids a broad rename; namespace cleanup can be a later coordinated migration.
5. Add focused tests as part of the extraction; do not rely on the third-party Addressables test as project coverage.
6. Compile and test the local Unity project only when there's an actual source boundary to validate, not for ongoing design-only steps.

## Next action
Map the files and direct references that would cross proposed Framework/Game/Editor assembly boundaries, then choose the smallest valid assembly cut. Revise or replace draft PR #18 only after this map exists. Do not merge without focused tests and Unity compile evidence.

## Source of truth
- Master Plan: task ordering and checkboxes.
- STATUS.md: exact active task.
- Audits: evidence and recorded decisions.
