# Project Status

## Source Project

This roadmap controls the architecture audit and refactoring of the source project **Aizekhan/MythHunter** (Unity RPG project, branch `dev`).

All code audits and implementation work are performed against `Aizekhan/MythHunter` unless explicitly stated otherwise.

Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`

Current plan: `RPG_FRAMEWORK_MASTER_PLAN.md`

## Current Position

- Epic: 02 — Architecture Rebuild
- Task: Architecture Rebuild
- Current item: Define forbidden dependencies
- Status: ACTIVE

## Sequential Execution Rule

Only one item may be ACTIVE at a time.

Later items remain LOCKED until the current item is completed.

## Completion Rule

An item is complete only when:

- Its audit is documented using `AUDIT_TEMPLATE.md`.
- A conclusion is recorded.
- Framework / Game Layer classification is recorded.
- Technical debt and refactoring needs are recorded.
- The corresponding checkbox in `RPG_FRAMEWORK_MASTER_PLAN.md` is checked.

## Status Transition Rule

When the current item is completed:

1. Mark the completed item as DONE.
2. Check its checkbox in `RPG_FRAMEWORK_MASTER_PLAN.md`.
3. Move `Current item` to the next item in strict order.
4. Set the new item to ACTIVE.
5. Keep all later items LOCKED.
6. If the current task has no remaining items, mark the task DONE, check the task checkbox in the master plan, and move to the next task.
7. If the current Epic has no remaining tasks, mark the Epic DONE and move to the next Epic.

## Source of Truth

`RPG_FRAMEWORK_MASTER_PLAN.md` is the master order.
`STATUS.md` is the current execution pointer.
Detailed audit documents contain the evidence and decisions.

Never start a later item before the current ACTIVE item is completed and the status is advanced.
