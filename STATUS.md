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
- Active task: 3.3 — First enforceable boundary
- Status: ACTIVE
- Objective: extract the canonical `IComponent` and `IEntityManager` contracts into a Framework-owned assembly, with no duplicate types or Framework-to-MythHunter dependency.
- Unity use: not needed for further architecture/source inspection on GitHub. Needed when the new assembly/source integration is validated locally.
- No source code has yet been changed through this task.

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
Prepare the exact minimal source diff for the contract assembly, preserving unrelated project changes. Once the source files are changed locally, validate compile/test and record the actual result. If the GitHub path can safely carry the source edits, do so on an isolated branch/PR; do not change `dev` in place without a reviewable recovery point.

## Source of truth
- Master Plan: task ordering and checkboxes.
- STATUS.md: exact active task.
- Audits: evidence and recorded decisions.
