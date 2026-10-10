# Project Status

## Source Project
This roadmap controls the architecture audit and refactoring of **Aizekhan/MythHunter**, Unity project on branch `dev`.

- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`
- Source baseline: `Audits/EPIC_03.1_Source_Baseline.md`
- First slice decision: `Audits/EPIC_03.2_First_Framework_Slice_Decision.md`

## Current Position
- Epic: 03 — First Reusable Framework Slice
- Active task: 3.2 — First Framework slice decision and usage map
- Status: ACTIVE
- Next deliverable: source-backed usage map for entity identity and ECS contracts, followed by target API decision.
- Source migration gate: do not move/add assemblies or change source until the affected callers are mapped and a recoverable local checkpoint is established.

## Confirmed local evidence
- Source branch/commit reported locally: `dev` / `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`.
- Unity version: `6000.0.45f1`.
- Six assembly definitions under `Assets/Plugins/UniTask`; none observed under `Assets/_MythHunter`.
- No test C# files found under `Assets`; `Assets/_MythHunter/Tests.meta` exists but is only folder metadata.
- EditMode: 1 passed test from `AddressableAssets.DocExampleCode.Editor.Tests`; it is not a MythHunter test.
- PlayMode: 0 test cases discovered.
- Unity CLI returned “script recompilation was not required”; no forced clean compile was demonstrated.
- Local worktree contains Unity/CLI setup changes and generated files. User preserved a patch, XML test reports, status/diff summaries and VS Code files under `D:\RPG-Framework-Baseline`; this is a partial recovery bundle, not a full repository backup.
- No MythHunter source code has been changed by this work.

## Execution rule
Exactly one roadmap item may be ACTIVE. Update this file and the Master Plan together when the active task changes. Do not ask for more diagnostics unless the result changes a concrete implementation decision.

## Current risks/constraints
1. A clean compilation is not yet independently demonstrated; compile validation is required after the first bounded source change.
2. No existing MythHunter tests were discovered. Relevant tests need to be authored as part of implementation rather than treating third-party package tests as project coverage.
3. Do not discard the local Unity/CLI setup changes or reset the working tree. Preserve them while implementing the framework.
4. Avoid adding speculative empty assemblies or rewriting the entire ECS. Start with the smallest reusable, source-backed ECS contract slice.

## Next action
Map all usages of `Entity`, entity IDs, `IComponent`, `IEntityManager`, `IEcsWorld` and `ISystemRegistry` to identify a safe Framework-owned ECS contract boundary. The observed `EcsWorld` currently depends on `MythHunter.Systems.Core.ISystemRegistry`, so it is excluded from the first slice until that dependency is inverted or kept in the Game Layer.

## Source of truth
- `RPG_FRAMEWORK_MASTER_PLAN.md`: execution order and checklist.
- `STATUS.md`: exact active task.
- `Audits/*`: evidence and recorded decisions.

Do not mark implementation complete based only on document edits or file moves.
