# Handoff — Двіжок / RPG Framework

> Read `STATUS.md` first, then `RPG_FRAMEWORK_MASTER_PLAN.md` and the linked audit source. This handoff is not a separate status database.

## Active roadmap item
EPIC 01 Phase 1 reference-validation closeout. The immediate tracked source PR is [MythHunter #48](https://github.com/Aizekhan/MythHunter/pull/48), ComponentCache consistency.

- Current PR head: `6631edf83362e6aed70ca70bc26b7c96ed15587e`.
- Branch: `feature/epic-01-component-cache-consistency`.
- Draft; do not merge yet.
- Existing local test XML reports 44/44 on local HEAD `40b6296`, which is not the exact current PR head. This is not sufficient as final merge evidence.
- Exact-head test attempt was blocked because compilation hits `PreloadDebugTool.cs` `EditorStyles.boldLabel`; fix is tracked in PR #50.
- PR #50 worktree EditMode attempt timed out at 600 seconds and created no NUnit XML; status is inconclusive. Diagnostics show no active project-specific Unity process and no change to Editor.log over 15 seconds. Do not kill the long-lived Unity CLI server or delete caches.
- CLI help confirms `unity recompile --project-path <path>`; prior use of a positional path for `recompile` was wrong. `unity test` accepts its project as a positional argument.
- Required: exact-head Unity EditMode, full-project compile and clean Console, Player Build, Movement/Visibility/Hero smoke checks, exact SHA recorded.
- Next: solve the Unity Editor/CLI launch issue with a controlled invocation; avoid repeated user diagnostics or further long timeouts.

## Additional open maintenance gate
[PR #50](https://github.com/Aizekhan/MythHunter/pull/50) is the retained draft for the `PreloadDebugTool.cs` EditorStyles issue. Validate separately; it is not the active roadmap item. PR #49 was closed as superseded by #50; its branch/history are preserved. This is not validation of the fix.

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


## Dormant PRs noted during deduplication
Do not merge old open PRs #1, #3 or #5 as-is; they are 224–244 commits behind current dev and need file-level triage. PR #1 retains a potentially intentional companyName: MasterMind project setting; PR #3 contains only candidate folder metadata; PR #5 has a large 206-file residual diff. PR #4 is a separate long-open dev → main integration PR requiring release review. PR #2 and PR #49 are closed duplicates; their branches/history remain preserved.
