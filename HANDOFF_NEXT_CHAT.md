# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked audit source. This handoff is not a separate status database.

## Active roadmap item
EPIC 01 Phase 1 reference-validation closeout. The immediate tracked source PR is [MythHunter #48](https://github.com/Aizekhan/MythHunter/pull/48), ComponentCache consistency.

- Current PR head: `6631edf83362e6aed70ca70bc26b7c96ed15587e`.
- Branch: `feature/epic-01-component-cache-consistency`.
- Draft; do not merge yet.
- Existing local test XML reports 44/44 on local HEAD `40b6296`, which is not the exact current PR head. This is not sufficient as final merge evidence.
- Required: exact-head Unity EditMode, full-project compile and clean Console, Player Build, Movement/Visibility/Hero smoke checks, exact SHA recorded.

## Additional open maintenance gate
[PR #50](https://github.com/Aizekhan/MythHunter/pull/50) is the retained draft for the `PreloadDebugTool.cs` EditorStyles issue. Validate separately; it is not the active roadmap item. PR #49 is redundant and should be closed as superseded by #50 without discarding its branch or history.

## First action
1. Run `git status --short --branch` in `D:\\MythHunter-Git` and preserve all local changes.
2. Do not switch branch, stash, reset, clean, or cherry-pick until the current diffs are understood. Use a separate worktree if needed.
3. Compare PR #48's exact head/diff with the local branch and verify which local changes are part of the PR.
4. Run tests and integration checks against the exact head; record SHA and actual results.
5. After validation, update STATUS, the relevant audit, the master plan and this handoff in one documentation cycle. Do not start EPIC 05.9 until EPIC 01 Phase 1 exit gates are complete.

## Guardrails
- One ACTIVE roadmap item only.
- Never write directly to `dev`; use a PR.
- Never reset, clean, or discard local files.
- A CI pass does not prove Unity compilation or Player Build.
- Do not reuse test counts from a different commit.
