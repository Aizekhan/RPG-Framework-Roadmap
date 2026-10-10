# EPIC 03.4 — Assembly Dependency Inventory

## Status
PARTIAL — contract assembly is plausible, but source changes need targeted hygiene before Unity validation.

## Current compilation shape
- No .asmdef or .asmref under Assets/_MythHunter on dev.
- Existing MythHunter source mostly compiles in Unity's predefined assembly layout.
- The staged Framework.ECS.Contracts.asmdef uses autoReferenced: true; predefined assemblies can reference this assembly by default. A giant MythHunter.Runtime.asmdef is not required merely to reference the contracts.
- The dependency direction remains strict: Framework contracts may not reference predefined MythHunter types.

## Confirmed consumer clusters
- IComponent: concrete component structs, ECS caches/storage/factories, archetype/template code, serialization contracts and implementations, generated code.
- IEntityManager: storage, component factories/cache, archetype/template code, gameplay systems, entity/hero factories, installers and ECS world.
- EcsWorld depends on MythHunter.Systems.Core.ISystemRegistry and remains on the predefined/Game side.
- The six UniTask asmdefs remain outside this task.
- UnityEngine-dependent components/UI/debug/settings/services do not need to move to a named assembly for this first contracts-only boundary.

## Specific source defect to address
Assets/_MythHunter/Code/Events/EventBus.cs contains a top-level using UnityEditor while the file is in runtime source. This is not a dependency of the contract assembly and must not expand this extraction, but it is a real portability/build defect to fix separately or when a runtime asmdef would otherwise expose it.

Other Editor references found by search are either in Editor directories or guarded by UNITY_EDITOR in inspected excerpts. Verify each actual file before moving it into a named runtime assembly.

## Required source review of draft PR #18
The current draft stages IComponent and IEntityManager under Contracts, empties their original source files, and adds an asmdef. The contract-only boundary may be valid because it is auto-referenced, but no compile may be claimed until Unity confirms it.

Check:
- the old script GUIDs are preserved by the new asset meta files;
- only one declaration of each interface exists across the project;
- the Contracts assembly has no non-BCL dependency;
- all current predefined-assembly callers resolve the contracts after Unity imports the branch;
- the PR diff contains no unrelated edits;
- no Entity wrapper or runtime storage migration was added.

## Smallest next action
1. Keep PR #18 draft and dev unchanged.
2. Complete static PR hygiene: GUID preservation, single declarations, asmdef validation and changed-file review.
3. Add focused test source in a test assembly for a testable seam. Do not represent an unrun test as passed.
4. Validate once in local Unity against this exact branch. No more setup diagnostics unless a concrete compile error blocks the task.
5. Update the plan only if validation reveals a design-changing constraint.

## Acceptance criteria
- Canonical declarations exist once.
- Existing Unity consumers compile against them.
- Framework contracts have no MythHunter dependency.
- Focused tests exercise relevant behavior.
- Unity compile outcome is recorded; no merge without it.
- User-local uncommitted changes remain intact.
