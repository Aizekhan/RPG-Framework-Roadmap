# Project Status

## Source Project
This roadmap controls the architecture audit and refactoring of source project **Aizekhan/MythHunter**, Unity project on branch `dev`.

- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- Roadmap reassessment: `Audits/EPIC_02.9_Post_Architecture_Roadmap_Reassessment.md`

## Current Position
- Epic: 03 — Baseline and Enforceable Boundary
- Task: 3.1 — Source baseline
- Current item: Record MythHunter source branch/commit, working tree, Unity/package/assembly state, compile and tests
- Status: ACTIVE
- Gate: no source-code migration until the baseline and rollback point are documented.

## Why this is active
The architecture target is documented, but the available roadmap evidence does not contain a verified local Unity compile/test baseline. A GitHub source snapshot or a no-op CI workflow is not a substitute for opening the actual project in Unity.

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
Unity compilation and tests have not been verified from the connected GitHub workflow. The local project must be opened in its declared Unity version, and baseline results/working-tree state must be supplied or otherwise directly verified before source migration can proceed.

## Next action
Complete EPIC 03.1 Source baseline. Record exact source commit, working-tree state, Unity version, package and assembly inventory, then compilation and test results. If compilation cannot be run, record the blocker and do not start an extraction.

## Source of Truth
- `RPG_FRAMEWORK_MASTER_PLAN.md`: execution order and checkboxes.
- `STATUS.md`: exact active task.
- `Audits/*`: evidence and architectural conclusions.

Do not mark implementation complete merely because documentation changed or files moved.