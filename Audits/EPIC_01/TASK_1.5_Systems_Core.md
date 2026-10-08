# Audit — Systems/Core Systems

## 1. Чи потрібна будь-якій RPG?
Так. Generic system registry, lifecycle, ordering, scheduling, timer and update abstractions потрібні RPG framework.

## 2. Чи залежить від конкретної гри?
Змішано. Частина інфраструктури універсальна, але current Core Systems напряму знає про GamePhase, PhaseSystem, Lobby/Combat/Phase events та MythHunter logging/DI.

## 3. Чи можна перевикористати без змін?
Ні.
Потрібне відокремлення generic system runtime від game-specific phase semantics та application composition.

## 4. Framework чи Game Layer?
Mixed.

## 5. Чи є архітектурні проблеми?
- SystemRegistry одночасно відповідає за registration, priority ordering, lifecycle initialization, update loops, phase filtering, event subscription, activation state, lookup та DI container exposure.
- ISystemRegistry прямо експонує GetContainer(), протікаючи DI infrastructure в system abstraction.
- SystemRegistry прив'язаний до MythHunter Events.Domain.GamePhase.
- Phase filtering змішане: string phase IDs + legacy GamePhase enum.
- SystemRegistry swallowing exceptions під час Initialize/Update/Dispose може приховувати partial system failure.
- SystemRegistry зберігає власний phase state, хоча phase authority фактично знаходиться в PhaseSystem.
- GetSystem<T>() знає про SystemGroup і має nested search policy.
- SystemRegistryExtensions містить legacy compatibility APIs та створює GamePhaseProvider/EmergencyPhaseProvider самостійно.
- SystemRegistryExtensions використовує reflection для доступу до private _eventBus.
- RegisterGroupWithCategory змішує group construction, DI resolution, registration, logging та error handling.
- ParallelSystemRegistry є дуже великим orchestration object: priority sorting, parallel groups, job scheduler, async execution, phase lookup, error policy.
- ParallelSystemRegistry знову знає про concrete MythHunter IPhaseSystem/GamePhase.
- ParallelSystemRegistry викликає system.Update() перед PrepareJobs(), що може означати duplicate work/side effects залежно від system implementation.
- Dependency metadata збирається, але у показаній реалізації _systemDependencies не використовується для фактичного topological scheduling.
- TimerSystem має generic timer mechanics, але також hardcodes Lobby, Phase and Combat event semantics.
- TimerSystem визначає category за substring у timer name — brittle implicit protocol.
- TimerSystem ExtractPhaseFromName hardcodes Rune/Planning/Active/Freeze.
- TimerInfo exposes Category as string.
- EventThrottlerUpdateSystem є thin integration adapter і має сенс лише як bridge між Event infrastructure та update loop.

## 6. Від чого залежить?
Core.ECS, DI, Events, MythHunter logging; додатково Unity/UniTask/job infrastructure у ParallelSystemRegistry та domain GamePhase у phase-related paths.

## 7. Хто залежить від цього?
Майже всі gameplay/application systems, installers та GameBootstrapper/system composition.

## 8. Чи є зайві залежності?
Так.
- ISystemRegistry -> DI container.
- SystemRegistry -> domain GamePhase/events.
- Extensions -> concrete Phase providers and reflection over private fields.
- TimerSystem -> domain events and phase model.
- ParallelSystemRegistry -> concrete PhaseSystem discovery.

## 9. Що потребує рефакторингу?
- Split registry responsibilities into registration, lifecycle, scheduler/update loop and activation policy.
- Remove DI container exposure from ISystemRegistry.
- Introduce generic phase/provider abstraction rather than GamePhase enum.
- Make system scheduling dependency-aware or remove unused dependency metadata.
- Separate generic TimerService/TimerSystem from Lobby/Phase/Combat event adapters.
- Replace stringly typed timer categories with typed identifiers.
- Remove reflection-based service discovery and private field access.
- Define explicit failure policy for initialize/update/dispose.
- Separate synchronous, asynchronous and job scheduling contracts.
- Keep parallel scheduling infrastructure out of generic SystemRegistry when possible.

## Висновок:
☑ Framework
☑ Game Layer
☑ Technical Debt

Загальна класифікація: Mixed. Core Systems містять справжню Framework infrastructure, але current registry/scheduler/timer layer перевантажені application/game-specific concerns і потребують декомпозиції.