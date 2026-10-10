# EPIC 03.3 — Assembly Boundary Reassessment

## Status
COMPLETE — the initial module boundary and the Unity reference direction are recorded.

## Verified Unity assembly behavior
Unity's predefined assemblies reference project asmdef assemblies by default when `Auto Referenced` is enabled. A custom asmdef must not depend on types from Unity's predefined `Assembly-CSharp`. The initial Framework assembly is therefore isolated and depends on no MythHunter types.
Source: Unity Manual, https://docs.unity.cn/Manual/ScriptCompilationAssemblyDefinitionFiles.html

## Source constraints observed
- No `.asmdef` or `.asmref` exists under `Assets/_MythHunter` on `dev`.
- Current MythHunter ECS contracts have broad consumers in the predefined assembly.
- `EcsWorld` depends on `MythHunter.Systems.Core.ISystemRegistry` and remains game-owned.
- UnityEngine/UI/content/editor tooling remain outside the Framework runtime.

## Decision
Use a standalone pure .NET module at `Assets/_Framework/ECS/Runtime`; it is a new canonical Framework API, not a file move of the old MythHunter contracts. Keep legacy MythHunter interfaces and implementation intact while the first slice is independently developed and tested.

The new API uses namespace `RPGFramework.ECS`, and the new assembly has no project-assembly references and disables Unity engine references. This allows existing predefined-assembly code to reference it when needed, while ensuring the Framework doesn't depend on MythHunter.

## Corrections to the earlier draft
An earlier contract-only draft under `Assets/_MythHunter/Code/Core/ECS/Contracts` was a false start because the coexistence of the new and legacy interfaces was not clearly bounded and the old declarations risked confusing the source of truth. PR #18 now instead stages the standalone `RPGFramework.ECS.Runtime` under `Assets/_Framework/ECS/Runtime`, preserves the legacy files/GUIDs, and explicitly tracks consumer migration as future work.

The PR also contains Editor/runtime portability edits. They need a scoped review before merge; do not silently treat them as necessary for the ECS API.

## Validation evidence
- Pure .NET and NUnit workflow: https://github.com/Aizekhan/MythHunter/actions/runs/38049497156
- Tested commit: `be362f9df6910b3027f05f5b742b65bc3dc65182`
- Static boundary/GUID validation passed.
- 8 tests passed, 0 failed, 0 skipped.
- Unity editor import/compilation and Unity Test Runner have not yet been demonstrated for this branch.

## Conclusion
The pure Framework ECS runtime is the initial reusable slice. Do not add a broad MythHunter runtime asmdef for it. Do not merge PR #18 until Unity validation runs against the feature branch and unrelated Editor/runtime fixes have been reviewed.
