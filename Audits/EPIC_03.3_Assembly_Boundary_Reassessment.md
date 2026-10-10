# EPIC 03.3 — Assembly Boundary Reassessment

## Status
CORRECTED — the original concern about predefined assembly references was based on an incorrect assumption.

## Verified Unity assembly behavior
Unity's assembly-definition documentation states that predefined assemblies reference project assemblies created with Assembly Definition assets by default when their `Auto Referenced` option is enabled. The current draft's `Framework.ECS.Contracts.asmdef` has `autoReferenced: true`, so this alone does not require moving all MythHunter scripts into a named assembly. See Unity Manual: https://docs.unity.cn/Manual/ScriptCompilationAssemblyDefinitionFiles.html

The reverse direction still matters: an asmdef assembly cannot use types from Unity's predefined assemblies. Therefore the contracts assembly must stay dependency-free and must not pull in MythHunter runtime implementations.

## Source constraints that remain valid
- No `.asmdef` or `.asmref` currently exists under `Assets/_MythHunter`.
- `IComponent` and `IEntityManager` have many users in the existing predefined assembly.
- `EcsWorld` depends on `MythHunter.Systems.Core.ISystemRegistry` and must not be moved into the contracts assembly.
- Several files have editor-specific references. Those should be cleaned up separately where they are actual compile problems, but they do not by themselves invalidate an auto-referenced contract assembly.

## Decision
Keep the current draft PR #18 as a small, potentially valid first boundary; do not replace it with a broad runtime/editor assembly migration. The remaining tasks are to inspect the PR diff for API compatibility, ensure asset GUIDs are preserved, add focused tests, and validate Unity compilation. The PR remains draft until those validation steps pass.

## Corrected conclusion
A contracts-only asmdef can be referenced from the existing predefined assembly when `Auto Referenced` is true. Do not add a `MythHunter.Runtime.asmdef` solely to connect the contracts. Only broaden the assembly cut if a concrete dependency requires it.
