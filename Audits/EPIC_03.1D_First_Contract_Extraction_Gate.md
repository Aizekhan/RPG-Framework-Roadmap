# EPIC 03.1D — First Contract Extraction Gate

## Status
BLOCKED — awaiting local source checkout and Unity compile/test baseline

## Purpose
Define the exact conditions required before the first production source change that moves `IComponent` into a Framework-owned assembly. This gate prevents a documentation-only plan from being misreported as a completed extraction.

## Why this gate exists
The source audit and code search confirm that `IComponent` is used across:
- ECS storage and component factory/cache helpers;
- concrete components under multiple game domains;
- archetype/template builders and registries;
- component serializer interfaces and registries;
- gameplay systems;
- an Editor code generator that emits `using MythHunter.Core.ECS;`.

The interface itself is a tiny marker, but its CLR type identity is a cross-cutting generic constraint. A second copy under another namespace would create a distinct type and break constraints; an uncoordinated move would break consumers and generated code.

## CI evidence
The current `.github/workflows/ci.yml` on source branch `dev` is named “Minimal CI”. Its only job echoes that CI is delegated to Unity Cloud Build; it does not check out the source, compile Unity, or run tests. Therefore, this workflow is not a validated compile/test baseline.

The source repository declares Unity Editor version `6000.0.45f1` in `ProjectSettings/ProjectVersion.txt`.

## Gate checklist
The first code patch must not begin until the following have been confirmed from an actual local source checkout:

- [ ] Current source branch and commit SHA recorded; working tree is clean or unrelated work is protected.
- [ ] Open the project with the declared Unity version (or a documented compatible environment).
- [ ] Record baseline Unity compilation result.
- [ ] Record available EditMode/PlayMode test results and distinguish pre-existing failures.
- [ ] Enumerate all references to `IComponent`, including fully qualified names, generic constraints, generated-source templates and non-Code source files.
- [ ] Review all dependencies needed to place the interface in a new Framework assembly.
- [ ] Decide canonical namespace and assembly ownership once; do not define a duplicate interface.
- [ ] Define the smallest patch and rollback point.
- [ ] After the patch, compile again and run relevant tests.
- [ ] Update roadmap only with verified outcomes.

## Allowed work before the gate passes
- Continue source evidence gathering and architecture documentation.
- Inspect existing contract/consumer files from GitHub.
- Draft a proposed minimal patch for review.

## Forbidden claims before the gate passes
- Do not claim the Framework contract has been extracted.
- Do not mark `Framework contracts` complete.
- Do not state that the Unity project compiles or tests pass.
- Do not advance `STATUS.md` to ECS/DI/Event extraction.

## Next required action
Use a local clone of `Aizekhan/MythHunter` branch `dev` in Unity 6000.0.45f1 to establish the baseline and complete the checklist. Once verified, implement the smallest canonical `IComponent` move and validate it before proceeding.

## Conclusion
The candidate and consumer map are sufficiently grounded to select `IComponent` as the first extraction target, but remote file inspection alone cannot safely validate an assembly-level source change. EPIC 03.1 remains ACTIVE and is explicitly blocked on the local compile/test gate.

No MythHunter source code was changed by this gate document.
