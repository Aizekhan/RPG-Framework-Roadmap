# Audit — Components

Source project: `Aizekhan/MythHunter`  
Branch: `dev`  
Path: `Assets/_MythHunter/Code/Components`

## Name

Components

## Structure observed

```text
Components
├── Core
├── Character
├── Combat
├── Movement
└── Lobby
```

The directory is ECS component storage for MythHunter gameplay/application state. Examples inspected include `StatsComponent`, `SkillsComponent`, `CombatStatsComponent`, `HealthComponent`, `PathComponent`, `LobbyStateComponent`.

## 1. Is it needed by any RPG?

**Yes, conceptually.**

RPGs commonly need ECS data components for identity-related data, stats, health, combat, movement, abilities/skills, inventory, and similar runtime state.

However, the current component set is not universally applicable as-is because it contains MythHunter-specific concepts such as lobby state and hero selection.

## 2. Does it depend on a specific game?

**Partially.**

The component layer itself mostly depends on `MythHunter.Core.ECS`, but many component definitions encode MythHunter-specific gameplay and application concepts.

Examples:
- `LobbyStateComponent` represents the current game's lobby/hero-selection flow.
- `SkillsComponent` uses the local `Skill` model.
- `StatsComponent` uses the local `StatType` enum.
- `CombatStatsComponent` contains game-specific combat mechanics such as Rage and Concentration.
- `PathComponent` stores Unity `Vector3` values.

So the directory is not a pure universal component library.

## 3. Can it be reused without changes?

**Partially.**

The ECS pattern and some concepts can be reused. Individual components need different treatment:

- Generic concepts such as health, position, movement, stats and team/faction can potentially become framework modules.
- MythHunter-specific concepts such as lobby state, hero selection, and specific combat resources should remain in a game/RPG module unless generalized.

The current serialization approach also needs redesign before treating these components as framework assets.

## 4. Framework or Game Layer?

**Shared abstraction / mixed.**

The component mechanism and generic component concepts belong to the Framework.

The current concrete component collection is mixed:
- Framework candidates: generic runtime data.
- Game Layer candidates: lobby, hero-specific, and MythHunter-specific combat/gameplay data.

## 5. Are there architectural problems?

- The directory mixes universal data concepts with game-specific gameplay/application state.
- Some components depend directly on Unity types such as `Vector3`, coupling simulation data to Unity.
- Components contain their own binary serialization logic, coupling data components to a specific persistence/serialization implementation.
- Some components appear to contain multiple conceptual responsibilities. For example `CombatStatsComponent` combines combat attributes, combat resources, and combat-tracking state.
- Some serialization implementations are incomplete/stubbed (`PathComponent`, `LobbyStateComponent` return empty byte arrays).
- The current structure does not clearly distinguish universal RPG data from MythHunter-specific data.

## 6. What does it depend on?

Observed dependencies include:

- `MythHunter.Core.ECS`
- .NET collections and primitives
- UnityEngine in Unity-specific components
- Local game-domain types such as `StatType` and `Skill`

## 7. Who depends on it?

Components are intended to be consumed by gameplay systems and other ECS infrastructure. The likely consumers include:

- Systems
- Entity factories/templates
- Serialization/persistence infrastructure
- UI/presentation adapters
- Networking/replication code

The exact consumer graph belongs to the later file-level audit.

## 8. Are there unnecessary dependencies?

**Likely yes.**

Main candidates:

- Component → UnityEngine for data that may belong in platform-specific adapters.
- Component → concrete serialization implementation.
- Component → game-specific domain concepts inside structures intended to become Framework components.

These require file-level verification before refactoring.

## 9. What needs refactoring?

No code changes in this audit.

Refactoring candidates:

- Separate generic Framework components from MythHunter/Game Layer components.
- Move serialization responsibility out of individual components into dedicated serialization infrastructure where appropriate.
- Remove or isolate Unity-specific data types from simulation components where platform independence is required.
- Split overly broad components after file-level responsibility analysis.
- Define rules for what qualifies as a Framework component.
- Replace incomplete serialization implementations before relying on them for persistence/networking.

## Conclusion

- [ ] Framework
- [ ] Game Layer
- [x] Technical Debt

**Classification:** Components are a **mixed/shared structure**. The ECS component model belongs to the Framework, while the current concrete component collection contains both reusable RPG data and MythHunter-specific Game Layer data. The structure also contains technical debt around serialization, Unity coupling, and responsibility boundaries.
