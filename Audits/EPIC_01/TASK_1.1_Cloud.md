# Audit — Cloud

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Cloud

## Name

Cloud

## 1. Is it needed by any RPG?

Cloud services are not required by every RPG. Authentication, remote persistence and analytics are optional infrastructure capabilities.

For a reusable RPG Framework, cloud integration should therefore be optional and adapter-based rather than part of the mandatory runtime core.

## 2. Does it depend on a specific game?

The inspected interfaces contain almost no direct MythHunter gameplay dependencies.

However, the code is placed inside the MythHunter namespace/project and the abstractions are currently generic application services rather than RPG-specific domain contracts.

## 3. Can it be reused without changes?

Partially.

ICloudService, IAuthService, IDataService and IAnalyticsService are reusable abstractions in principle.

IDataService is overly generic for a framework persistence architecture, while authentication and analytics contracts are better treated as optional infrastructure modules with provider adapters.

AnalyticsEvent uses a generic dictionary/object payload, which weakens type safety and can create runtime schema coupling.

## 4. Framework or Game Layer?

**Framework infrastructure, optional module.**

Cloud should not be part of the universal gameplay core. It should sit behind interfaces and be replaceable by external providers.

Concrete provider implementations belong in adapters/integration modules, not Framework Core.

## 5. Are there architectural problems?

- Cloud combines unrelated concerns: authentication, remote data storage and analytics.
- ICloudService is a weak common abstraction; its shared lifecycle does not necessarily fit all providers.
- IDataService uses string keys and unconstrained generic payloads, giving weak schema guarantees.
- IAuthService assumes username/password account flows that do not fit every platform/provider.
- IAnalyticsService exposes Dictionary<string, object>, creating runtime-only schema.
- AnalyticsEvent has its own event model instead of clearly integrating with the Framework Event abstraction.
- No concrete provider implementation exists in this top-level structure.
- Authentication, data and analytics have different security, lifecycle and failure semantics and should not be forced into one conceptual service family.

## 6. What does it depend on?

Observed:
- Cysharp.Threading.Tasks
- System.Collections.Generic
- System.DateTime
- MythHunter.Cloud.Core from Analytics

No direct Unity or gameplay/component dependency was observed in the inspected files.

## 7. Who depends on it?

No direct consumer was found in the top-level code search. The interfaces appear intended for application/bootstrap/services that need account, remote data or analytics functionality.

Exact consumers should be verified in later detailed audits.

## 8. Are there unnecessary dependencies?

Potentially:
- Analytics depending on Cloud.Core only to inherit ICloudService.
- The common ICloudService abstraction itself may be unnecessary if it does not represent a useful shared capability.
- AnalyticsEvent may be better independent of cloud delivery and routed through an analytics adapter.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split Cloud into optional infrastructure modules: Auth, Remote Data, Analytics.
- Remove or narrow ICloudService if a meaningful common contract cannot be established.
- Define provider-agnostic result/error/session semantics.
- Replace untyped analytics dictionaries with a controlled telemetry contract.
- Separate analytics event creation from analytics transport.
- Keep provider SDKs behind adapters.
- Define security/session handling independently from generic Cloud initialization.

## Conclusion

- [x] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: Cloud is **optional Framework infrastructure**, not Game Layer. The current code is a useful contract layer but needs separation of concerns and stronger contracts before becoming universal Framework infrastructure.
