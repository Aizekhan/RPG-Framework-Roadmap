# EPIC 02.9 — Post-Architecture Roadmap Reassessment

## Status
ACTIVE — planning-only review

## Purpose
Reassess the remaining Master Plan after EPIC 02 established the target architecture. This task must happen before further source extraction so that the implementation sequence follows actual dependency and validation constraints rather than treating the existing EPIC 03–14 ordering as immutable.

## Decision
The current plan remains a useful inventory of outcomes, but EPIC 03–14 must not be executed blindly in their current order. Their scopes overlap and some foundational decisions must be validated before later epics can be meaningfully sequenced.

No source code is changed by this reassessment.

## Required review
- [ ] Map every remaining epic/task to the target architecture zones and intended owner.
- [ ] Identify duplicates and overlaps between Framework Extraction, Dependency Cleanup, ECS Completion, and Infrastructure Hardening.
- [ ] Separate prerequisite decisions from implementation tasks.
- [ ] Introduce validation gates that require Unity compile/tests and dependency checks after each cohesive migration step.
- [ ] Decide the minimal first compiling slice using actual source references and Unity assembly constraints.
- [ ] Re-sequence Game Layer extraction and composition-root work around stabilized Framework contracts.
- [ ] Keep optional modules (Resources, Pooling, Replay, Cloud, Networking, Tools) from blocking the minimal reusable Framework Core unless proven necessary for the first game path.
- [ ] Keep Persistence and Networking serialization concerns separate.
- [ ] Update RPG_FRAMEWORK_MASTER_PLAN.md and STATUS.md only after the revised sequence is reviewed and internally consistent.

## Initial findings from existing roadmap
1. EPIC 03 (Framework Extraction) says to extract contracts and runtime modules.
2. EPIC 05 (Dependency Cleanup) contains work that may be prerequisite to, or inseparable from, safe extraction (cycles, duplicated EventBus, serialization coupling, reflection, async lifecycle).
3. EPIC 06 (ECS Completion) includes architectural decisions and validation which EPIC 03 says must be made before ECS extraction (storage and identity), so the ownership/order needs to be reconciled.
4. EPIC 07 (Infrastructure Hardening) repeats work in DI, events, logging, validation, lifecycle and tests already listed in EPIC 03/05.
5. EPIC 08 has both reusable abstractions and optional modules/adapters, which should not automatically be part of Core extraction.
6. EPIC 11 (Persistence) and EPIC 12 (Networking) need independent schemas/protocol responsibilities and must not be prematurely coupled to the initial Framework extraction.
7. The migration strategy already prescribes small, compiling checkpoints and explicitly allows re-baselining when source constraints differ from the proposal.

These are roadmap-level observations from current planning documents, not proof that any implementation is already complete.

## Proposed reassessment approach
### Step A — Build a traceable task matrix
For every remaining task, record: goal, target owner/zone, dependencies, source evidence needed, compile/test gate, and whether it belongs in the first usable Framework milestone or a later optional milestone.

### Step B — Resolve order conflicts
Reconcile EPIC 03/05/06/07 into one dependency-ordered sequence instead of treating them as fully independent large phases. Preserve distinct outcomes, but avoid duplicate tasks.

### Step C — Define milestones
- **M0 — Baseline:** known Unity compile/test state, source commit, working-tree state, assembly and package inventory.
- **M1 — Enforce boundary:** first minimal Framework-owned assembly boundary with no Framework → MythHunter reference.
- **M2 — Minimal reusable core:** extract only contracts/runtime proven to be necessary and independently buildable.
- **M3 — Game integration:** one explicit Game composition path that consumes the Framework without changing gameplay behavior.
- **M4 — Optional capabilities:** adapters and optional runtime modules, each only when selected and independently validated.
- **M5 — RPG modules/tools/release:** universal gameplay modules, authoring tools, sample project and release validation.

These milestones are a proposal pending the task matrix and source/build evidence.

## Gate for completion
This reassessment can be marked DONE only when:
- every remaining Master Plan task is mapped and sequenced;
- overlapping scopes are merged or explicitly differentiated;
- prerequisites and exit criteria are visible;
- the updated Master Plan and STATUS.md agree;
- no active execution task is skipped or marked complete without evidence.

## Current blocker
The roadmap documents state that Unity compilation/test baseline has not yet been established. That evidence must be obtained before choosing the exact first source-code migration slice. The reassessment can clarify sequence and gates now, but must not pretend to verify compilation or source dependencies it has not tested.

## Conclusion
Reassess the remaining roadmap now, before continuing extraction. Keep the architecture baseline as a target, validate it against the real Unity project, then execute a smaller dependency-ordered plan with enforced build/test gates.