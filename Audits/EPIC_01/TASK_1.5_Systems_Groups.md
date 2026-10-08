# Audit — Systems/System Groups

## 1. Чи потрібна будь-якій RPG?
Так. Grouping systems into logical execution/lifecycle groups is useful in any modular RPG framework.

## 2. Чи залежить від конкретної гри?
SystemGroup — ні. PhaseSystemGroup — частково так через phase semantics and the current phase contract.

## 3. Чи можна перевикористати без змін?
SystemGroup майже так. PhaseSystemGroup — ні, через MythHunter GamePhase compatibility and IPhaseProvider design.

## 4. Framework чи Game Layer?
Framework candidate, з Game Layer integration у PhaseSystemGroup.

## 5. Чи є архітектурні проблеми?
- SystemGroup owns child systems and directly forwards Initialize/Update/Dispose; this is a reasonable composite pattern but has no explicit failure policy.
- Group allows mutable AddSystem/RemoveSystem at runtime, which can make execution ordering/lifecycle nondeterministic.
- SystemGroup depends on IMythLogger, adding logging infrastructure to a generic structural abstraction.
- PhaseSystemGroup mixes generic grouping with phase activation policy.
- PhaseSystemGroup stores phase IDs as strings, while also implementing legacy GamePhase APIs.
- PhaseSystemGroup contains duplicate phase representations and compatibility logic.
- Empty active phase list means 'always active', which is implicit policy and can be surprising.
- PhaseSystemGroup may duplicate phase filtering already performed by SystemRegistry, causing checks at both group and registry levels.
- Constructor accepts priority but never stores or uses it; priority belongs to registry/scheduling, not group.
- SystemGroup GetSystems exposes the whole internal execution set through IReadOnlyList, which is safe enough but still makes ordering externally observable.

## 6. Від чого залежить?
Core.ECS/ISystem, Core phase provider, logging; PhaseSystemGroup additionally depends on concrete GamePhase domain enum.

## 7. Хто залежить від цього?
SystemRegistry and SystemRegistryExtensions; phase-based gameplay groups can depend on PhaseSystemGroup.

## 8. Чи є зайві залежності?
Так.
- SystemGroup -> IMythLogger for a structural container.
- PhaseSystemGroup -> legacy GamePhase API.
- PhaseSystemGroup can duplicate registry phase filtering.
- Constructor priority parameter in PhaseSystemGroup is unused.

## 9. Що потребує рефакторингу?
- Keep a small generic SystemGroup/CompositeSystem in Framework.
- Move logging out or make it optional infrastructure.
- Define explicit group lifecycle and mutation rules.
- Remove priority from group constructor.
- Create a generic activation predicate/provider abstraction instead of GamePhase/string compatibility.
- Ensure phase filtering has one authoritative owner.
- Remove legacy SetActivePhases/GamePhase methods after migration.
- Decide whether empty phase list means always active; make this explicit.

## Висновок:
☑ Framework
☐ Game Layer
☑ Technical Debt

Загальна класифікація: Framework candidate. SystemGroup is a useful reusable composite; PhaseSystemGroup is an integration layer that should become thinner after phase abstraction is separated.