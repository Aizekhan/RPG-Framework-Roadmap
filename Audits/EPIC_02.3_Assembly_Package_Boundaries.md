# EPIC 02.3 — Assembly / Package Boundaries

## Status
DONE

## Purpose
Define a practical assembly/package grouping for the logical target module map in `Audits/EPIC_02.2_Target_Module_Map.md`. The goal is to enforce dependency boundaries without exploding every logical responsibility into its own Unity assembly.

## Decision principles
1. Begin with a small number of assemblies grouped by dependency and platform boundary.
2. Create an assembly only when it enforces a meaningful dependency rule, enables independent reuse/testing, or isolates platform/editor code.
3. Logical modules do not automatically require separate assemblies.
4. Avoid assembly cycles. Lower-level contracts must not reference implementations or Game Layer.
5. Keep Unity Editor assemblies separate from runtime assemblies.
6. Package distribution boundaries can be decided later from the resulting stable assembly/API layout; do not equate every assembly with a separate UPM package.

## Proposed initial assembly layout

### 1. Framework contracts
**Proposed assembly:** `RPGFramework.Contracts`

Owns the minimum shared, platform-neutral contracts and small primitives used across Framework modules.

Candidate contents:
- shared IDs and minimal common primitives, only when truly cross-cutting;
- generic service/lifecycle contracts that cannot be owned by a more specific contract assembly without creating awkward cycles.

Rule: keep this assembly small. ECS, events, DI, resources and serialization contracts should move to dedicated assemblies only when dependency evidence justifies it. If contracts can remain in a specific owning assembly, prefer that over a generic shared assembly.

### 2. Framework ECS contracts and runtime
**Proposed assemblies:**
- `RPGFramework.ECS` — ECS contracts and runtime together for the first extraction, unless tests or actual consumers prove that contract/runtime separation is necessary.
- Split into `RPGFramework.ECS.Contracts` and `RPGFramework.ECS.Runtime` only if it materially reduces coupling or is needed for a stable public API.

Owns entity/component/world/query contracts, identity/lifecycle, and the ECS implementation. No concrete MythHunter types and no UI references.

### 3. Framework systems / scheduling
**Proposed assembly:** `RPGFramework.Systems`

Owns system interfaces, lifecycle, scheduler and generic system groups. It may depend on ECS/event contracts as required, but it must not own MythHunter phase ordering/rules. If dependency evidence requires a lower-level system contract assembly, split later.

### 4. Framework dependency injection
**Proposed assembly:** `RPGFramework.DI`

Owns DI contracts and runtime implementation initially. Unity-specific lifecycle helpers or MonoBehaviour integration belong in the Unity adapter assembly, not here.

### 5. Framework events
**Proposed assembly:** `RPGFramework.Events`

Owns generic event contracts and local dispatch implementation. Middleware, pooling, replay and network bridges remain separate logical responsibilities even if initially compiled into the same assembly for practical reasons. It must not reference concrete MythHunter event types.

### 6. Framework serialization and persistence
**Proposed assemblies:**
- `RPGFramework.Serialization` — generic serializer/codec and schema registry infrastructure.
- `RPGFramework.Persistence` — save/load orchestration, persistence lifecycle and migration policies.

Persistence may depend on Serialization. Serialization must not depend on persistence, networking, or MythHunter content.

### 7. Optional Framework runtime modules
Create assemblies only for modules actually implemented and intended to be independently selected:
- `RPGFramework.Resources`
- `RPGFramework.Pooling`
- `RPGFramework.Replay`
- `RPGFramework.Networking`
- `RPGFramework.Cloud.Abstractions` (only if cross-game contracts are genuinely required)

These must not become dependencies of Framework Core by default. Replay can depend on event and serialization contracts. Networking must own transport/protocol boundaries and not redefine persistence serialization.

### 8. Platform adapters
**Proposed runtime adapter assembly:** `RPGFramework.Unity`

Owns Unity-specific runtime integrations such as scene loading, Unity resource providers, Unity lifecycle hooks, and other UnityEngine-backed implementations.

It may reference Framework contracts/modules and UnityEngine. Framework contracts/runtime must not reference it.

Provider-specific services and transport implementations should live in separate adapter assemblies when providers are selected (for example, a cloud provider adapter). Do not create empty provider packages speculatively.

### 9. Developer and Editor tooling
**Proposed assemblies:**
- `RPGFramework.Tools` — platform-neutral developer-tool contracts or reusable tooling code, if needed.
- `RPGFramework.Unity.Editor` — Unity Editor windows, code generation, project analysis, inspectors and editor-only utilities.

`RPGFramework.Unity.Editor` must be Editor-only and must not be referenced by runtime assemblies. MythHunter-specific editors remain in `MythHunter.Editor` unless they are genuinely generic.

### 10. MythHunter Game Layer
**Proposed initial grouping:**
- `MythHunter.Game` — domain rules, heroes, lobby, phases, combat, concrete gameplay systems, game-specific events, content definitions and application flow during the initial extraction.
- `MythHunter.UI` — views, presenters, controllers and presentation-specific code, separated if dependency evidence supports a clean boundary.
- `MythHunter.Unity` — game-specific Unity composition/adapters if needed; may initially be combined with MythHunter.Game if the current repository makes a split unsafe.
- `MythHunter.Editor` — MythHunter-specific content authoring tools and Editor extensions.

The target module map remains the logical ownership reference; assembly grouping should not force an unnecessary proliferation of small assemblies. Split MythHunter.Game further only after dependency analysis shows the split can be enforced without cycles.

## Dependency rules at assembly level

Allowed:
- MythHunter.Game → RPGFramework assemblies it uses.
- MythHunter.UI → MythHunter.Game contracts/domain + Framework public APIs as required.
- MythHunter.Unity → Framework APIs + UnityEngine + game APIs as needed for composition.
- RPGFramework.Unity → Framework contracts/runtime APIs + UnityEngine.
- RPGFramework.Unity.Editor → runtime Framework APIs + UnityEditor, but runtime assemblies must never reference the Editor assembly.
- RPGFramework.Persistence → RPGFramework.Serialization and required contracts.
- RPGFramework.Replay → event/serialization contracts and its own replay contracts.
- optional modules → required lower-level contracts, never back into Game Layer.

Forbidden:
- Any `RPGFramework.*` runtime assembly → `MythHunter.*`.
- Framework Core/contracts/runtime → UnityEditor.
- Framework contracts/runtime → UnityEngine, unless a later explicit decision rejects platform-neutrality for a narrowly named Unity-specific module.
- Framework Core → UI/presentation.
- Serialization → Persistence or Networking (reverse ownership).
- Networking → concrete game content or concrete domain events.
- Runtime assembly → any `*.Editor` assembly.
- Any circular assembly references.

## Unity package strategy
Do not create one Unity Package Manager package per assembly at this stage.

Initial recommendation:
- one distributable package or source subtree for the core Framework assemblies;
- optional modules can remain in the same package initially but must have dependency boundaries enforced by assemblies;
- Unity adapter and Editor tooling can be separate assemblies within the same package;
- split into separately distributed UPM packages only after public APIs and demand are validated.

This avoids packaging overhead before we know which modules users actually need independently.

## Migration approach
1. Add assembly definitions after confirming the real project folder structure and Unity assembly-reference constraints.
2. Start with the smallest enforceable boundary: Framework contracts/primitives.
3. Move one dependency group at a time and fix compiler-reported edges explicitly.
4. Do not move game logic merely to match proposed names.
5. Keep a compiling checkpoint after each extraction.
6. Run tests/build validation before checking off each migrated group.
7. Do not create placeholder assemblies for planned modules that have no implementation.
8. Keep source evidence and architecture proposals clearly distinguished.

## Risks and caveats
- These names and groupings are proposals, not verified existing `.asmdef` names.
- A source-level dependency graph and Unity's current assembly definitions must be inspected before implementing this layout.
- Unity package splitting is premature until consumers and public API shape are known.
- If existing folders cannot be safely separated, combine them initially while preserving module ownership in code and documentation.
- This task changes the architecture plan only; it does not change MythHunter source code.

## Conclusion
Start with coarse, enforceable assemblies, not dozens of tiny packages. Separate platform-neutral Framework, Unity runtime adapters, Editor tooling, and MythHunter Game Layer. Split contract/runtime or Game Layer assemblies further only when an actual dependency boundary justifies the cost.
