# EPIC 03.1B — Existing Contract Usage and Dependency Audit

## Status
ACTIVE — continuation of EPIC 03.1 Framework contracts

## Purpose
Identify actual dependency coupling around existing candidate interfaces before finalizing or moving Framework contracts. No production source edits are included.

## Source basis
- Source repository: `Aizekhan/MythHunter`
- Branch: `dev`
- Inspected tree SHA: `66f83dbf6a3ab87cc7584c098dd481a38c5279e2`
- Existing candidate interfaces and current implementations were fetched by path from this source branch.
- A global code-search query via the connector returned no matches, so the findings below rely on the actual fetched interface/implementation files and the prior code audit; a comprehensive call-site count is not yet established.

## Findings

### 1. ECS: the current contracts are not independent of Systems

Evidence:
- `Assets/_MythHunter/Code/Core/ECS/EcsWorld.cs` imports `MythHunter.Systems.Core`.
- `EcsWorld` injects both `IEntityManager` and `ISystemRegistry`.
- Its `Initialize`, `Update` and `Dispose` delegate to the system registry.
- `IEntityManager` uses the existing `IComponent` and exposes int IDs and array-returning enumeration.

Implication:
- The current world contract/implementation combines entity/component world ownership with system lifecycle/scheduling.
- Moving `IEcsWorld` by itself into a Framework assembly would not make the world independently compilable while `EcsWorld` retains the concrete `ISystemRegistry` dependency.
- This is an implementation boundary issue; keep the existing public surface stable until the composition/scheduler dependency is redesigned.

Decision:
- Treat entity management/component/query contracts as the smallest first ECS contract candidate.
- Do not move `EcsWorld` runtime implementation until system scheduling ownership is explicitly separated.
- Keep entity ID and query API redesign out of the first contract move unless direct consumer analysis demonstrates a required breaking change.

### 2. DI: the container contract leaks concrete lifecycle helper types

Evidence:
- `IDIContainer` includes registration/resolution/lifetime functions and directly returns or accepts `DIScope` and `LazyDependency<T>`.
- `DIContainer` depends on `MythHunter.Utils.Logging.IMythLogger`.
- `DIContainer` includes reflection and concrete registration/lifetime implementation details.

Implication:
- Moving `IDIContainer` alone across the Framework boundary may not compile independently because related helper types and namespace ownership remain in MythHunter.
- The container is currently coupled to the logging abstraction and implementation concerns; DI should depend on a neutral logger contract rather than a MythHunter-specific interface.

Decision:
- Keep DI API design as a separate focused contract decision; first record usages of scope/lazy members and whether callers genuinely need public access to them.
- Do not move `DIContainer` implementation in this contract task.
- Do not copy `IDIContainer` into a new namespace while leaving the old one active; use one canonical contract and migrate consumers coherently later.

### 3. Events: current public event API couples occurrence, priority, async scheduling and UniTask

Evidence:
- `IEvent` requires `GetEventId()` and `GetPriority()`.
- `IEventBus` uses `Cysharp.Threading.Tasks.UniTask` in its async API, struct-only event constraints and `EventPriority`.
- `EventBus` implementation includes priority queues, event pooling, debug middleware, concrete game-event processor registration, asynchronous polling/delay and reflection-based dispatch.

Implication:
- `IEventBus` is not a minimal neutral dispatch contract as currently defined.
- Moving it unchanged can make the Framework events contract depend on UniTask and preserve current policy coupling.

Decision:
- The first extractable seam is a platform-neutral event occurrence/subscription/publish contract, but its exact members require a call-site and behaviour inventory.
- Keep priority, async dispatch, queueing, pooling, middleware and replay as explicitly selected policies/extensions unless usage evidence demonstrates they are inseparable.
- Do not rewrite the current EventBus until semantic behaviours are captured and tests exist.

### 4. Serialization: current object-owned byte API is too policy-specific for the generic contract by default

Evidence:
- `ISerializable` forces implementers to expose `Serialize(): byte[]` and `Deserialize(byte[])`.
- The source also contains `IComponentSerializer`, `ComponentSerializerRegistry` and `VersionedSerializer` under the Data/Serialization area.

Implication:
- There are multiple serialization concepts to reconcile before creating a new Framework public API.
- A generic serializer contract should not silently merge save-file policies with network wire representation.

Decision:
- Audit `IComponentSerializer` and its registry/versioning contracts alongside the current usages before finalizing Framework.Serialization.
- Do not extract only `ISerializable` as the final universal serialization API.

### 5. Existing assembly boundary

Evidence:
- The repository tree contained project assembly definitions only under `Assets/Plugins/UniTask/`.
- No `.asmdef` was found under `Assets/_MythHunter/Code/`.

Implication:
- No current source-level assembly boundary enforces the intended Framework/Game separation in the audited code tree.
- Creating an assembly definition around one existing interface without checking every dependency can cause broad compilation failures.

Decision:
- Before first code change, establish baseline compile/test results in local Unity 6000.0.45f1 and calculate the minimal set of files/dependencies for the first independent contract assembly.
- The available GitHub connector did not run Unity compile/tests; build status must not be reported as verified.

## Recommended first extraction candidates
Candidate order based on low dependency surface, subject to usage confirmation:
1. `IComponent` as a marker contract, once the namespace/assembly move is coordinated with all consumers.
2. A deliberately minimal entity/component manager contract only if its current semantics and query allocations are accepted as an initial compatibility surface.
3. System lifecycle contract only after separating it from MythHunter phase/scheduler policy.
4. Event contract only after deciding ownership of ID, priority and async behaviour.
5. DI contract after defining the proper ownership of scope/lazy APIs.
6. Serialization only after reconciling the existing serialization interfaces/registry/versioning.

This is an order of investigation, not a guarantee that these interfaces should all be public final APIs.

## Required next verification
Before changing source:
- enumerate call sites and implementations for the chosen first contract using a source checkout or an available full repository code-search/indexing path;
- inspect actual current `.asmdef` files and Unity package dependencies locally;
- establish compile/test baseline;
- identify stable GUID/meta-file handling for any added/moved Unity assets;
- design the smallest possible diff and rollback checkpoint.

## Conclusion
The code confirms that several candidate interfaces already exist, but the first Framework contract extraction must be dependency-driven, not a namespace rename. ECS world/scheduler, DI helper types, EventBus policy, and serialization abstractions remain coupled. Keep this EPIC 03.1 item ACTIVE until one contract is actually extracted, compiled and validated against the source project.
