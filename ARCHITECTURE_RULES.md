# Architecture Rules

## Dependency Direction

Game Layer
  ↓
RPG Modules
  ↓
Simulation / ECS Core
  ↓
Infrastructure abstractions

Framework Core must not depend upward on concrete game modules.

## ECS Rules

- Entity is identity only.
- Component contains data/state only.
- System contains behavior/logic.
- Query selects entities for systems.
- Events represent occurrences, not persistent gameplay state.
- Stats/capabilities are different from runtime statuses.
- Structural ECS changes must have a defined lifecycle.
- System execution order must be explicit.

## Infrastructure Rules

- DI wires dependencies; it is not gameplay state.
- EventBus is communication, not a replacement for ECS state.
- Persistence has explicit schema/versioning.
- Networking replicates authoritative simulation state.
- Presentation/Unity-specific code stays outside simulation core.

## Module Rules

- One responsibility per module.
- Avoid cyclic dependencies.
- Minimize concrete dependencies.
- Prefer interfaces at architectural boundaries.
- Do not introduce game-specific concepts into universal Core.

## Audit Rules

- Audit before refactor.
- Record every architectural decision.
- Do not skip a level in the master plan.
- Complete the current checkbox before starting the next one.
