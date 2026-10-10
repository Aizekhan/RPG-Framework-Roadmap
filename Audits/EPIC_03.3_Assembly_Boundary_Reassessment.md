# EPIC 03.3 — Assembly Boundary Reassessment

## Status
COMPLETE — assembly-reference direction verified; no broad MythHunter assembly migration required for the first slice.

## Verified Unity assembly behavior
Unity's assembly-definition documentation states that predefined assemblies reference project assemblies created with Assembly Definition assets by default when their `Auto Referenced` option is enabled. The initial Framework assembly is auto-referenced, so existing predefined-assembly code can reference it without introducing one large MythHunter runtime asmdef. Source: Unity Manual, https://docs.unity.cn/Manual/ScriptCompilationAssemblyDefinitionFiles.html

The reverse direction remains important: a custom asmdef assembly cannot depend on types compiled into Unity's predefined `Assembly-CSharp` assembly. Therefore Framework code must not depend on MythHunter runtime types.

## Source constraints
- No `.asmdef` or `.asmref` currently exists under `Assets/_MythHunter` on `dev`.
- `IComponent` and `IEntityManager` have many consumers in the current MythHunter predefined assembly.
- `EcsWorld` depends on `MythHunter.Systems.Core.ISystemRegistry`; keep that integration on the MythHunter side.
- Some Editor-specific imports exist in runtime-looking files. These require guarded imports/Editor-only method bodies, but do not force a whole-project assembly migration for a pure .NET Framework module.
- Keep the six existing UniTask asmdefs outside this first step.

## Decision
The first reusable slice is now staged as `RPGFramework.ECS.Runtime` under `Assets/_Framework/ECS/Runtime`. It contains the framework-facing contracts and an initial standalone entity manager with no Unity or MythHunter dependencies. A separate EditMode test assembly exercises it.

Legacy `MythHunter.Core.ECS` contracts/runtime remain in place during this first step. This temporary coexistence makes the new module independently testable without a risky game-wide rename. It is explicitly transitional: a later task must migrate consumers to the Framework APIs and remove the duplicate legacy ECS implementation.

## Validation and current PR
- Draft PR: https://github.com/Aizekhan/MythHunter/pull/18
- Pure .NET GitHub Actions test run: https://github.com/Aizekhan/MythHunter/actions/runs/38048892361
- Latest reported result: 7 passed, 0 failed, 0 skipped.
- Unity import/compilation for this branch has not yet been demonstrated.

## Conclusion
Do not create a broad `MythHunter.Runtime.asmdef` for this step. Keep the Framework module pure .NET, test it independently, preserve current MythHunter operation, and do not merge PR #18 until the local Unity import/compile/test gate passes.
