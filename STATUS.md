# Project Status

> Operational pointer only. Roadmap scope/history live in [RPG_FRAMEWORK_MASTER_PLAN.md](RPG_FRAMEWORK_MASTER_PLAN.md). Do not duplicate milestone histories here.
> Last reconciled against GitHub: 2026-10-11.

## Repositories
- Roadmap: [Aizekhan/RPG-Framework-Roadmap](https://github.com/Aizekhan/RPG-Framework-Roadmap)
- Source: [Aizekhan/MythHunter](https://github.com/Aizekhan/MythHunter)
- Source branch: `dev`
- Current source head: `3bb562ec75eb280f92cc487cc3ab1d7eb13c9efb` — [documentation-only architecture cleanup](https://github.com/Aizekhan/MythHunter/commit/3bb562ec75eb280f92cc487cc3ab1d7eb13c9efb). Runtime source is unchanged from prior code head `e43fed636df0d1b257da5f9708f1a8af685fbd0e`.

## Verified completed sequence
- EPIC 05.4: PR #25 merged as `df83fdbc438071bfe714978ade7416f9cb2a4146`; Unity EditMode 33/33 and integration checks confirmed.
- EPIC 05.5: PR #27 merged as `81e621ca98aa4ef409ead5cae7b6380328689b4c`; Unity EditMode 36/36 and integration checks confirmed.
- EPIC 05.6: PR #28 merged as `4830f7735dce6a12ecbf2eee3d5c2379c9dda340`; Unity EditMode 41/41 and integration checks confirmed.
- EPIC 05.7: PR #29 merged as `3b2783bd8005c58a3454dfad7dc132272fccdbd4`.
- EPIC 05.8: PR #30 merged as `e7ecbd854a7ff2c8b30a8308597b452ec570fd7c`.
- The master plan now records the above history. Do not infer current work from an old chat summary.

## Exactly one active roadmap item
**ACTIVE — EPIC 01 Phase 1 reference-validation closeout; immediate tracked implementation gate is [MythHunter PR #48](https://github.com/Aizekhan/MythHunter/pull/48), ComponentCache consistency.**

Current PR head: `6631edf83362e6aed70ca70bc26b7c96ed15587e` on `feature/epic-01-component-cache-consistency`; draft and not merged.

### Unmet gates for PR #48
- Local Unity EditMode evidence currently reports 44/44 at local HEAD `40b6296`, **not** the current PR head `6631edf`; repeat on exact head.
- Full-project compilation with zero Console errors.
- Player Build.
- Movement, Visibility and Hero consumer smoke checks.
- Review final local/remote diffs and record exact tested SHA. Do not merge until all gates are satisfied.

### Separate duplicate maintenance PR
- [PR #50](https://github.com/Aizekhan/MythHunter/pull/50) is the only retained draft for the `PreloadDebugTool.cs` EditorStyles compile issue; its local Unity/Player validation remains a separate gate. It is not a second active roadmap item.
- PR #49 is **closed as superseded by #50** (closure comment recorded on PR #49). This is PR deduplication only; the defect remains unvalidated until #50's required local checks pass.

## Next action
1. Inspect and preserve the local worktree: `git status --short --branch`; do not reset, clean, discard, or switch branches unsafely.
2. Compare local diffs with PR #48 head `6631edf83362e6aed70ca70bc26b7c96ed15587e`; use a separate worktree if necessary.
3. Validate exact PR head in Unity, then update PR #48 and this status with results. Do not start EPIC 05.9 or broad refactoring before EPIC 01 Phase 1 exit gates are met.


## Dormant GitHub PRs — not active roadmap work
These historical PRs predate the current architecture effort and must not be merged as-is. This inventory is a maintenance queue, not a second roadmap. It does not change the single active roadmap pointer above.
- [PR #1 — CombatSystem V2](https://github.com/Aizekhan/MythHunter/pull/1): the PR's current diff against `dev` is only two folder .meta files plus `ProjectSettings.asset` changing `companyName` to `MasterMind`; its original source feature isn't in the residual diff. Preserve that setting as possibly intentional, but do not restore the stale branch wholesale.
- [PR #3 — RuneSystemV1](https://github.com/Aizekhan/MythHunter/pull/3): 1 commit ahead and 244 behind `dev`; its residual diff only proposes `_MythHunter` and `Core` folder metadata. Validate GUIDs and whether those files are still needed before deciding; do not merge the old branch directly.
- [PR #5 — Feature/phases](https://github.com/Aizekhan/MythHunter/pull/5): 9 commits ahead and 224 behind `dev`, with a 206-file residual diff. Requires fresh source-level triage or closure after verifying whether any unique behavior remains.
- [PR #4 — UnifiedMythHunterWizard](https://github.com/Aizekhan/MythHunter/pull/4) is a long-open dev → main integration PR, not a roadmap implementation task. Review as a release/integration decision separately; do not merge it together with focused source fixes.
- PR #2 was closed as a duplicate of PR #3's folder-metadata proposal; PR #49 was closed as a duplicate of PR #50's PreloadDebugTool fix.

## Conflict rule
If STATUS, master plan, audit files, PR metadata, or current `dev` disagree, stop work and reconcile them before implementation. Test results are valid only for the exact commit tested.
