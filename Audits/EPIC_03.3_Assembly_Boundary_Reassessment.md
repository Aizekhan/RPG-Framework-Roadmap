# EPIC 03.3 — Assembly Boundary Reassessment

## Status
ARCHITECTURE DECISION — source PR remains draft and is not approved for merge.

## New evidence
Repository inspection shows:
- No assembly definitions currently exist under `Assets/_MythHunter`.
- MythHunter source files are therefore currently compiled by Unity's predefined assembly layout (except where other package/plugin assembly definitions apply).
- The existing code includes UnityEngine dependencies across UI, debug tools and providers, and UnityEditor references in several files beneath `Assets/_MythHunter/Code` (some guarded by editor symbols, some appear as direct imports).
- The ECS contracts are used broadly by gameplay, systems, archetypes, caches, serializers and installers.
- The draft PR's isolated `Framework.ECS.Contracts` asmdef cannot safely be consumed by the existing predefined assembly. This means moving interfaces alone does not produce a compiling boundary.

## Decision
Do not expand the current PR into a large assembly migration. Do not add one giant `MythHunter.Runtime.asmdef` merely to connect the contracts: editor-only classes, direct UnityEditor imports and runtime package dependencies make that too broad to validate safely in one step.

The right near-term sequence is:

1. **Treat PR #18 as a boundary experiment, not a candidate for merge.**
2. Design a minimal assembly graph with three ownership zones:
   - `Framework.ECS.Contracts`: only `IComponent` and the minimal stable ECS contracts.
   - `MythHunter.Game` (name provisional): gameplay/runtime implementation and consumers; must reference the Framework contracts.
   - `MythHunter.Editor` (name provisional): editor tooling and UnityEditor-dependent code; references game/runtime code, never the reverse.
3. Before creating these assemblies, split the actual source set by dependencies and compilation symbols. Move or isolate editor-only files first where needed; fix direct Editor imports in runtime files before the corresponding assembly can be created.
4. Keep UnityEngine-specific components/UI/presentation out of Framework contracts. A Framework contract assembly may use no Unity references, while a separate Unity adapter assembly may be introduced later if a concrete need appears.
5. Only after dependency grouping is source-backed, implement the smallest coherent migration on a feature branch, add focused tests and validate with Unity.

## Why not merge the current draft
The PR moves contract source files but does not yet establish a consumable reference path for the rest of the project. It also has no focused tests or compilation evidence. Merging it now risks breaking the Unity project. Keep it draft until replaced or amended by a complete, validated assembly slice.

## Plan change
The old immediate next step “add the contracts asmdef and move two interfaces” is too narrow because it fails to account for Unity's predefined-assembly reference direction. Replace it with an assembly-layout/dependency mapping task followed by a smaller enforceable slice.

## Acceptance criteria for the next task
- Inventory all source files that need to belong to each proposed assembly.
- Identify every direct `UnityEditor` reference in runtime directories and every dependency crossing candidate boundaries.
- Confirm the minimum acyclic assembly graph.
- No assembly is created until each of its compile-time dependencies is representable.
- Implementation is done only on the feature branch, with no changes to `dev`.
- Draft PR does not merge without Unity compilation and focused tests.

## Explicitly not doing now
- No all-at-once assembly migration.
- No ECS rewrite or entity ID redesign.
- No user-local reset, cleanup, or branch force update.
