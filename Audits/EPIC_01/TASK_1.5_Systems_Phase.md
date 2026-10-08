# Audit — Systems/Phase Systems

## 1. Чи потрібна будь-якій RPG?
Фазова state/round progression може бути потрібна turn-based, tactical та session-based RPG. Але конкретний набір фаз і transition rules є Game Layer.

## 2. Чи залежить від конкретної гри?
Так.
- PhaseSystem hardcodes GamePhase.Rune/Planning/Active/Freeze.
- Default durations and next-phase order are MythHunter rules.
- It depends on concrete domain events.
- GamePhaseProvider maps the domain enum to string IDs.

## 3. Чи можна перевикористати без змін?
Ні. Generic phase lifecycle idea is reusable, but current implementation is MythHunter-specific.

## 4. Framework чи Game Layer?
Game Layer, with a Framework candidate for generic phase/state progression.

## 5. Чи є архітектурні проблеми?
- PhaseSystem combines state machine, timer, transition policy, event publishing, pause/resume and game lifecycle handling.
- The phase transition graph is hardcoded in GetNextPhase.
- Phase durations are hardcoded in constructor initialization.
- PhaseSystem duplicates PhaseChangedEvent publication in StartPhase and OnPhaseChangeRequest.
- OnPhaseChangeRequest calls EndPhase/StartPhase and then publishes another PhaseChangedEvent, creating duplicate notifications.
- PhaseSystem is not derived from the generic SystemBase despite implementing ISystem directly, creating a parallel lifecycle model.
- IPhaseSystem exposes concrete MythHunter GamePhase, preventing reuse.
- IPhaseProvider lives under Core.ECS but is semantically a phase/application service, indicating boundary leakage.
- GamePhaseProvider translates between strings and GamePhase, creating a dual identity model for phases.
- EmergencyPhaseProvider is a fallback that hardcodes a different list containing Movement and Combat, while GamePhaseProvider has Rune/Planning/Active/Freeze. This can create inconsistent phase semantics.
- EmergencyPhaseProvider has callbacks but does not invoke them because it never changes phase.
- PhaseSystem uses IEventThrottler for a core state update stream, adding infrastructure coupling.
- GameEnded handler is async but does not perform asynchronous work; UniTask adds unnecessary complexity.
- PhaseSystem uses implicit default duration 10 seconds for unknown phases, which can hide configuration errors.

## 6. Від чого залежить?
Core.ECS, DI, Events, EventThrottler, Logging, UniTask and concrete GamePhase domain model.

## 7. Хто залежить від цього?
SystemRegistry, phase-filtered systems/groups, game flow, gameplay systems and UI/presentation that consume phase events.

## 8. Чи є зайві залежності?
Так.
- concrete GamePhase inside the reusable-looking contracts;
- string ID compatibility layer;
- throttler inside phase core;
- async handler without real async work;
- EmergencyPhaseProvider creation through registry extensions.

## 9. Що потребує рефакторингу?
- Define a generic PhaseId/phase definition contract independent of MythHunter GamePhase.
- Separate phase state machine, duration/timing service and event adapter.
- Make phase transitions data/config driven instead of hardcoded switch/durations.
- Emit one authoritative PhaseChanged event per transition.
- Unify IPhaseProvider/GamePhaseProvider/Emergency provider into one clear abstraction and remove legacy string bridge.
- Remove fallback Emergency provider from normal production flow.
- Use the same system lifecycle abstraction consistently.
- Make invalid/missing phase duration an explicit configuration error rather than defaulting silently.
- Reassess whether throttling belongs in event infrastructure instead of PhaseSystem.

## Висновок:
☐ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Game Layer, with a strong candidate for extracting a generic Phase/State progression infrastructure. Current implementation is tightly bound to MythHunter phases and contains duplicated event/compatibility logic.