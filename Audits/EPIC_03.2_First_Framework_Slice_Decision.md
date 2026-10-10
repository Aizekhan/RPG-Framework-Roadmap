# EPIC 03.2 — First Framework Slice Decision

## Status
DECISION RECORDED — source code unchanged. This is a boundary decision, not an implementation or compile claim.

## Goal
Choose one small, high-value slice that starts turning MythHunter into a reusable Framework without requiring a broad rewrite.

## Evidence reviewed
Files on branch `dev` at observed source head `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`:
- `Assets/_MythHunter/Code/Core/ECS/IComponent.cs`
- `Assets/_MythHunter/Code/Core/ECS/Entity.cs`
- `Assets/_MythHunter/Code/Core/ECS/IEcsWorld.cs`
- `Assets/_MythHunter/Code/Core/ECS/IEntityManager.cs`
- `Assets/_MythHunter/Code/Core/ECS/ISystem.cs`
- `Assets/_MythHunter/Code/Core/ECS/EcsWorld.cs`
- `Assets/_MythHunter/Code/Core/ECS/EntityManager.cs`

The local inspection reported:
- no `.asmdef` or `.asmref` beneath `Assets/_MythHunter`;
- no test C# files under `Assets/_MythHunter/Tests` or elsewhere under `Assets`;
- six assembly definitions belong to UniTask;
- EditMode ran one passing Addressables package stub test; PlayMode discovered zero tests;
- Unity CLI reported that script recompilation was not required. This is not evidence of a forced clean compile;
- the local worktree has changes from environment/project setup, including the Unity pipeline package and settings. These changes are not discarded or committed by this decision.

## Decision: start with ECS identity and contracts, not extraction
The first Framework-owned slice should be a small, dependency-light ECS contract/identity package. Do not migrate the full `EcsWorld`, `EntityManager`, scheduler, storage, cache or archetype implementation as the first action.

Initial candidate scope:
1. Define what an entity handle/identity means and how validity/lifecycle is represented.
2. Define the minimal component marker and entity-manager contract only after checking every current usage and value/reference-type constraints.
3. Keep `EcsWorld` and the system registry integration outside the first slice: the current `EcsWorld` imports `MythHunter.Systems.Core.ISystemRegistry`, so pulling it into a neutral assembly now would create the wrong dependency direction.
4. Keep the current implementation files in place until the contract has a verified owner and all callers/serialization/editor/code-generation usage are mapped.

## Why this is the smallest useful slice
- Identity and the core contracts are prerequisites for a reusable ECS and for future RPG modules.
- `EcsWorld` is already coupled to MythHunter's systems registry and is therefore not a safe first extraction.
- Existing `Entity` is a wrapper around an `int`, while `IEntityManager` primarily exposes raw integer IDs. Before extraction we must decide whether the wrapper is canonical, whether IDs can be reused, and what happens for invalid or destroyed IDs.
- The current `EntityManager` silently creates storage when `AddComponent` receives an unknown ID, and returns `default` from a missing `GetComponent`. These are existing semantics to review, not to change without an explicit contract decision.
- The manager indexes components by `typeof(TComponent)` and its interface constraint allows component kinds that are not uniformly value types, while `TryGetComponent<T>` specifically requires `struct`. These API constraints must be reconciled before a public Framework contract is frozen.

## First implementation task after this decision
Create a source-backed usage map for the entity ID / `Entity` / `IComponent` / `IEntityManager` contracts and list behavior decisions in a short decision table. Then propose the target neutral API. Do not add an asmdef or move files until that map establishes all callers and the local Unity project has an agreed rollback point.

## Explicit non-goals
- No broad ECS rewrite.
- No switch to archetype storage or a new ECS library.
- No new tests falsely represented as pre-existing baseline tests.
- No source code changed by this document.
- No claim that the Unity project has a clean compile baseline.

## Acceptance criteria for the next coding step
- Every caller of affected types is inventoried.
- Entity validity, destruction, ID reuse, missing component behavior, and component type constraints have explicit decisions.
- Framework-owned contracts contain no MythHunter namespace, game policy, UnityEditor code or system registry dependency.
- The plan identifies a bounded migration with rollback, compile and focused test steps.
