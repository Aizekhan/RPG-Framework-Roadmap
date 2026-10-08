# EPIC 01.5 — Audit Completeness Check

## Scope
Verify the additional top-level/runtime/editor modules that were not covered with dedicated detailed audits in EPIC 01.

## Replay
- `Replay/IReplaySystem.cs` exists and is coupled to `IEvent` plus serialization/replay lifecycle.
- Replay is potentially reusable Framework infrastructure, but the current contract is incomplete and implementation is not present in the audited source area.
- Classification: optional Framework module / Technical Debt.

## Resources / Preload / Pool
- `IResourceManager`, providers, `PreloadManager` and `IPoolManager` form a substantial runtime resource subsystem.
- Resource loading is reusable infrastructure, but current implementation depends on Unity objects, phases, events, logging and pooling.
- `PreloadManager` hardcodes resource keys and phase/scene names, so the generic abstraction and MythHunter configuration must be separated.
- Pooling is optional infrastructure, not a mandatory RPG core dependency.
- Classification: Framework infrastructure candidates + Technical Debt.

## Cloud / External Services
- Cloud exposes `ICloudService`, `IAuthService`, `IDataService` and `IAnalyticsService`.
- These are integration boundaries, not RPG-domain foundation.
- Authentication, cloud persistence and analytics should be optional adapters/extensions and must not become Framework Core dependencies.
- Classification: optional integration layer / Game or Framework adapter depending on target provider.

## Debug / Developer Tools
- Debug services, preload/pool/event tools and profiling tools exist.
- These are developer tooling and must remain outside runtime Framework Core.
- Useful future Framework Tools module, but optional.
- Classification: Tools / Technical Debt where runtime coupling exists.

## Authoring / Editor Tooling
- Unity Editor tooling includes dependency analysis, code generation, hero archetype editors/wizards, prefab tools and pool/debug windows.
- Editor tools should be separated into an Editor-only assembly/module and must not leak into runtime Framework assemblies.
- Classification: Framework tooling or Game-specific authoring depending on tool ownership.

## Utils / Logging / Validation
- Generic validator interfaces and `Ensure` utilities are potential Framework support modules.
- `IMythLogger` is a Framework candidate as a logging abstraction; `MythLogger` implementation should be replaceable.
- Unity-specific utility helpers belong in adapters.
- Existing DI validation mixes runtime and editor concerns and should be separated.
- Classification: Framework support infrastructure + Technical Debt.

## Important completeness discovery
- The original EPIC 01 top-level list contained these folders, but not all received dedicated detailed audits.
- This does not invalidate EPIC 01; it identifies missing depth that must be incorporated before architecture rebuild.
- `Authoring` exists at the top-level code tree but contains no demonstrated runtime module in the sampled source and should remain an authoring boundary.

## Final completeness verdict
- No missing top-level area remains unacknowledged at roadmap level.
- The newly identified modules expand the Framework boundary: Resources/Pooling, Replay, Logging/Validation, Debug/Tools and optional Cloud integrations.
- These modules should not all be placed in Framework Core. They belong in separate optional infrastructure/tooling/adapter layers.
- No gameplay source code was modified.

### Classification
- Framework support: logging, validation, resource abstractions, pooling abstractions, replay contracts.
- Optional Framework modules: replay, resources, pooling, developer tools.
- Adapters/integrations: cloud/auth/analytics, Unity-specific resource/editor integrations.
- Game Layer: concrete resource configurations, content, hero authoring and MythHunter-specific tooling.