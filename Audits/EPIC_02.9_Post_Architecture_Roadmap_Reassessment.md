# EPIC 02.9 — Post-Architecture Roadmap Reassessment

## Status
COMPLETE — roadmap restructured

## Purpose
Reassess EPIC 03–14 after the target architecture was consolidated in EPIC 02, and replace the old phase ordering where it obscured prerequisites, repeated work, or mixed core and optional capabilities.

## Decision
The former epic sequence was useful as an inventory, but not sufficiently precise as an execution order. The Master Plan has been rewritten to:
- put source/build baseline and an enforceable boundary before contract extraction;
- stabilize shared contracts before runtime module extraction;
- resolve ECS identity/storage decisions before ECS migration;
- separate serialization foundation from persistence and networking protocols;
- move concrete MythHunter ownership through a single Game Composition Root;
- separate dependency-debt work from extraction while explicitly avoiding duplicate work;
- keep resources, pooling, replay, cloud, networking, and tools optional;
- add exit gates and compile/test requirements to each phase.

The revised plan is a sequencing proposal grounded in the existing audit and EPIC 02 architecture baseline. It does not claim implementation or compile validation has occurred.

## Source-derived scope reconciliation
1. **EPIC 03 and old contract/runtime extraction:** split into baseline/boundary, contracts, and runtime modules. The source's Unity baseline must be known before the first change.
2. **Old EPIC 05 dependency cleanup:** split by ownership. EventBus and ComponentCache problems are handled alongside the modules that own them; remaining cross-cutting cleanup follows Game integration.
3. **Old EPIC 06 ECS completion:** its storage/identity decisions are prerequisites within the new ECS runtime task, not a free-standing implementation phase after extraction.
4. **Old EPIC 07 infrastructure hardening:** DI/events/lifecycle tests now live within their relevant module tasks. Cross-cutting graph and async cleanup remain later debt work.
5. **Old EPIC 08 services:** explicitly optional and divided into platform-neutral abstractions, game-owned configuration, Unity/provider adapters and tools.
6. **Old EPIC 11 persistence vs old EPIC 12 networking:** the plan defines separate epics and forbids networking from owning persistence schema or implementation details.
7. **Game Layer extraction:** placed after a stable selected Framework core and composed through one authoritative Game Composition Root.
8. **Release/tools:** release readiness requires a sample project that consumes Framework public APIs without MythHunter game-specific code.

## Revised execution order
- **EPIC 02.9:** roadmap reassessment (this task); complete.
- **EPIC 03:** source baseline and first enforceable boundary.
- **EPIC 04:** Framework Core contracts and shared primitives.
- **EPIC 05:** Framework runtime modules: logging/validation, DI, event dispatch, systems lifecycle, ECS.
- **EPIC 06:** serialization foundation, explicitly separated from persistence and networking.
- **EPIC 07:** MythHunter Game Layer and authoritative composition.
- **EPIC 08:** residual dependency debt/lifecycle stabilization without duplicating resolved module tasks.
- **EPIC 09:** optional runtime services and adapters.
- **EPIC 10:** RPG foundation domain modules.
- **EPIC 11:** reusable gameplay modules.
- **EPIC 12:** persistence.
- **EPIC 13:** optional networking.
- **EPIC 14:** tools, sample and release.

This order is not permission to skip task exit gates. Actual source dependencies may require a documented adjustment.

## Milestones
- M0 — Baseline: EPIC 03.1.
- M1 — Enforceable boundary: EPIC 03.2–03.3.
- M2 — Reusable Core: EPIC 04–06 for the explicitly selected scope.
- M3 — MythHunter integration: EPIC 07–08 with a working game path.
- M4 — Optional capabilities: EPIC 09, 12 and 13 as selected.
- M5 — RPG modules and adoption: EPIC 10–11 and 14 for intended release scope.

## Validation and limitations
- Master Plan and STATUS pointer must agree at all times.
- Source migration requires a recoverable checkpoint, Unity compilation and relevant tests after each cohesive step.
- Current source notes identify Unity 6000.0.45f1 and sparse assembly definitions, but no verified local compile/test baseline has been recorded.
- GitHub connector operations do not run Unity compilation. Exact source migration boundaries must be validated in the local Unity project.
- No MythHunter source code was changed by this reassessment.

## Conclusion
The roadmap has been re-sequenced to make prerequisites explicit, reduce overlap, and keep Framework Core small. The next active task is EPIC 03.1 — Source baseline. Do not extract `IComponent` or any other contract until this baseline and the first boundary review have passed their gates.
