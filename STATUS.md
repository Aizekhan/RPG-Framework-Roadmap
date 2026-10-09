# Project Status

## Source Project
This roadmap controls the architecture audit and refactoring of source project **Aizekhan/MythHunter**, Unity project on branch `dev`.

- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`
- Source baseline audit: `Audits/EPIC_03.1_Source_Baseline.md`

## Current Position
- Epic: 03 — Baseline and Enforceable Boundary
- Task: 3.1 — Source baseline
- Current item: Local Unity compile/test baseline and local working-tree/checkpoint verification
- Status: ACTIVE — PARTIAL/BLOCKED pending local evidence
- Gate: no source-code migration until baseline compile/test state and rollback point are documented.

## Evidence already recorded
- Remote source branch observed: `dev`
- Remote source head observed: `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`
- Declared Unity version: `6000.0.45f1`
- Six assembly definitions found under UniTask; no `.asmdef` observed under `Assets/_MythHunter/Code/` in the inspected tree.
- Editor and Runtime test directories exist.
- Current GitHub Actions workflow is a no-op and does not compile or run tests.

See `Audits/EPIC_03.1_Source_Baseline.md` for the complete evidence and limitations.

## Sequential Execution Rule
Exactly one roadmap item may be ACTIVE. Later items remain LOCKED until the current item passes its exit gate and this file and the Master Plan are updated together.

## Completion Rule
A roadmap item is complete only when:
- evidence and findings are documented;
- ownership and relevant dependencies are stated;
- applicable compile/test or validation results are recorded;
- regressions are distinguished from pre-existing failures;
- a rollback point exists for source changes;
- its Master Plan checkbox and this status pointer agree.

## Current Blocker
The GitHub source inventory is complete enough to document metadata, but it does not prove the local project's compile/test state or working-tree status. The available CI workflow does not perform Unity compilation or tests. Local Unity results and a rollback checkpoint are still required before source migration.

## Next action
Open the local project in Unity `6000.0.45f1`. Confirm the local branch/commit and working-tree state, preserve unrelated changes, create or confirm a recoverable checkpoint, wait for script compilation, run available EditMode/PlayMode tests and report the results. If compilation or tests fail before any source change, record those failures as baseline failures.

## Source of Truth
- `RPG_FRAMEWORK_MASTER_PLAN.md`: execution order and checkboxes.
- `STATUS.md`: exact active task.
- `Audits/*`: evidence and architectural conclusions.

Do not mark implementation complete merely because documentation changed or files moved.