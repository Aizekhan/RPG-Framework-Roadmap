# EPIC 03.2 — ECS Contract Usage Map and API Decisions

## Status
SOURCE USAGE MAP RECORDED — implementation shape narrowed; source code unchanged.

## Why Unity is not the current tool
Unity is the host runtime/editor for the existing MythHunter project. It is not needed to inspect C# contracts, map their references on GitHub, decide ownership or draft the first API boundary. Unity becomes necessary when we modify assembly definitions/source integration and need to prove the project compiles and the relevant behavior still works. Do not open Unity for further architecture-only work.

## Source files inspected
- `Assets/_MythHunter/Code/Core/ECS/IComponent.cs`
- `Assets/_MythHunter/Code/Core/ECS/Entity.cs`
- `Assets/_MythHunter/Code/Core/ECS/IEntityManager.cs`
- `Assets/_MythHunter/Code/Core/ECS/EntityManager.cs`
- `Assets/_MythHunter/Code/Core/ECS/IEcsWorld.cs`
- `Assets/_MythHunter/Code/Core/ECS/EcsWorld.cs`
- `Assets/_MythHunter/Code/Core/ECS/ComponentFactory.cs`
- `Assets/_MythHunter/Code/Core/ECS/ComponentCache.cs`
- `Assets/_MythHunter/Code/Entities/EntityFactory.cs`
- `Assets/_MythHunter/Code/Data/Serialization/ComponentSerializerBase.cs`
- `Assets/_MythHunter/Code/Systems/Core/ISystemRegistry.cs`

## Current usage map (observed from branch `dev`)
| Contract/type | Observed consumers | Consequence |
|---|---|---|
| `IComponent` | `IEntityManager`, ComponentCache, ComponentFactory, archetype/template builders and registries, serialization contracts/implementations, component structs, Editor code generator | Widely used across gameplay and infrastructure; preserve this public identity when migrating. |
| `IEntityManager` | `EcsWorld`, `ComponentFactory`, `ComponentCache`, `ComponentCacheRegistry`, `ComponentCacheExtensions`, `EcsOptimizer`, `EntityFactory`, HeroFactory, archetypes/templates, hero/lobby/combat/movement systems and installers | Very broad caller surface. Do not replace its ID types in one step. |
| `Entity` class | Declared as a wrapper around an integer ID; GitHub code search did not establish meaningful production use | Do not assume it is canonical. Before removing or adopting it, complete targeted search for `Entity`, `.Id`, constructors and reflection/serialization/editor uses. |
| Raw entity IDs (`int`) | EntityManager API, factories, archetype APIs, caches, systems, UI/domain-facing methods, code generator | Existing de facto API is integer-based; changing to an Entity value type is a compatibility migration, not the first extraction. |
| `IEcsWorld` / `EcsWorld` | `GameBootstrapper` resolves and initializes the world; CoreInstaller binds it | World startup/lifecycle is integrated with game composition. Keep out of the very first neutral contract slice. |
| `ISystemRegistry` | `EcsWorld`, bootstrap, state machine, flow manager and installers | Registry is MythHunter-owned and includes phase/category policy; current `EcsWorld` has a direct Game dependency. |
| Component storage semantics | `EntityManager` stores `Dictionary<int, Dictionary<Type,IComponent>>`; reverse index `Dictionary<Type,HashSet<int>>` | Storage is implementation detail and should remain out of the minimal contract package. |
| Component value-type assumptions | `IEntityManager` broadly constrains `TComponent : IComponent`, while `TryGetComponent`, cache, serializers, template builders and most components constrain `struct, IComponent` | Decide intentionally whether Framework guarantees struct-only components. Do not silently narrow the public API during extraction. |

## Important behavior discovered
- IDs begin at 1 and increment; current code does not reuse IDs within the manager lifetime.
- `DestroyEntity` silently returns when ID is not registered.
- `AddComponent` creates backing storage even for an ID never returned by `CreateEntity`. This can accidentally resurrect/introduce an entity.
- `GetComponent` returns `default` when the entity/component is missing, which makes “missing” ambiguous for value types.
- `TryGetComponent<T>` requires a struct component, unlike several other interface methods.
- `Entity(int id)` accepts any integer and holds no link to a manager, so its existence does not establish that the entity is alive.
- EntityFactory's clone path uses reflection to get/add components and its helper only lists a subset of component types; it is a caller-specific risk to address separately, not in the first contract extraction.
- No MythHunter tests were found locally; tests for the selected behavior will need to be added as implementation work. This is not a reason to keep repeating baseline diagnostics.

## Recommended decisions for the first slice
1. **Keep `int` entity IDs for the first extraction.** Too many callers and content/editor paths use them; changing ID representation would expand the task dramatically.
2. **Keep `IComponent` as the canonical minimal marker contract**, initially source-compatible and namespace-neutral after extraction. Do not duplicate the interface in a second assembly.
3. **Decide whether to keep `IEntityManager` together with `IComponent` or split a minimal contract.** Recommended first cut: extract the current interface as-is to preserve signatures, while documenting and testing existing semantics; API tightening is a later breaking change.
4. **Do not extract `Entity` yet.** It appears redundant with the integer-ID API and needs a full-use check.
5. **Do not extract `IEcsWorld` / `EcsWorld` in the first slice.** Their lifecycle integration and the concrete `ISystemRegistry` coupling belong to the Game composition boundary for now.
6. **Leave `EntityManager`, caches, archetypes, serializers, factories and all concrete components where they are in the first contract-only move**, then migrate callers coherently once the Framework assembly can reference the canonical contract.

## Narrow first implementation step
Create a single Framework-owned assembly for neutral ECS contracts and move (not copy) the canonical `IComponent` and `IEntityManager` definitions into it, retaining the current namespace initially if namespace compatibility reduces churn. Update source consumers to reference that assembly without introducing any reverse reference from Framework to MythHunter. Do not add `EcsWorld`, `EntityManager` or `Entity` to the assembly in this step.

This exact extraction should only begin when the source branch can be edited locally and the local changes can be preserved. Because current worktree modifications are unrelated setup changes, use a named branch/patch checkpoint; do not reset the tree. Add focused tests for component/entity-manager contract behavior alongside a small testable implementation seam; test sources should not be represented as already existing.

## Acceptance criteria
- No duplicate `IComponent` or `IEntityManager` runtime type exists.
- Framework-owned contracts do not import MythHunter, UnityEngine/UnityEditor, logger, DI, game phases, system registry or providers.
- All existing implementations and consumers compile against the canonical interfaces.
- Focused tests cover create/destroy, invalid IDs, add/remove/has/get and missing component semantics.
- Unity compile outcome and test report are recorded honestly; failures are not hidden as a passing baseline.
- Unrelated working-tree changes remain intact.
