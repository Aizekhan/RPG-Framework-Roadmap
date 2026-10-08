# Audit — Services

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Services

## Name

Services

## 1. Is it needed by any RPG?

Service abstractions are common in RPG architecture, but a generic Framework should define capabilities by domain rather than provide an undifferentiated Services bucket.

## 2. Does it depend on a specific game?

Yes, strongly.

- GameSettingsService contains MythHunter-specific modes and values.
- IHeroDataService is explicitly about HeroDataModel.
- LocalHeroDataService persists MythHunter hero data.
- PrefabProvider assumes Heroes and Archetype IDs.

## 3. Can it be reused without changes?

Mostly no. The service pattern is reusable, but these contracts and implementations encode MythHunter concepts and conventions.

## 4. Framework or Game Layer?

Primarily **Game Layer**. Possible Framework responsibilities should move into dedicated infrastructure modules such as Persistence, Resource/Asset Infrastructure and Configuration.

## 5. Are there architectural problems?

- Services is an architectural catch-all containing unrelated responsibilities.
- GameSettingsService hardcodes MythHunter game modes, player counts, mana and lobby timer values.
- IHeroDataService couples the service layer directly to Entities.Heroes.
- LocalHeroDataService combines filesystem persistence, caching, serialization, logging and domain-specific hero storage.
- GetUserHeroIdsAsync accepts userId but the implementation ignores it and returns all local heroes.
- LocalHeroDataService uses UnityEngine.JsonUtility, making persistence implementation Unity-specific.
- PrefabProvider hardcodes Prefabs/Heroes and ArchetypeId-to-path conventions.
- IPrefabProvider exposes Unity GameObject, unsuitable for a pure Framework core.
- PrefabProvider mixes runtime resource loading and Unity Editor validation.
- Service contracts expose UniTask directly.

## 6. What does it depend on?

Observed:
- Cysharp.Threading.Tasks
- MythHunter.Core.DI
- MythHunter.Utils.Logging
- MythHunter.Entities.Heroes
- MythHunter.Resources.Core
- UnityEngine
- UnityEditor under conditional compilation
- System.IO and JSON serialization

## 7. Who depends on it?

Likely consumers include Hero systems/factories, gameplay/lobby/UI flows, resource/prefab loading, application configuration and installers. Exact consumers belong to later detailed audits.

## 8. Are there unnecessary dependencies?

Yes, candidates include direct hero-domain coupling, UnityEngine/GameObject exposed by service interfaces, UnityEditor implementation concerns, the Services catch-all structure, and direct UniTask exposure at reusable boundaries.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split Services into dedicated modules with one responsibility each.
- Move game settings into Game/Application configuration.
- Move hero persistence into Persistence/GameData infrastructure.
- Move prefab access into Resource/Asset infrastructure.
- Keep Unity-specific GameObject/AssetDatabase adapters at the infrastructure boundary.
- Make user identity part of persistence scope so userId cannot be silently ignored.
- Separate persistence, cache and serialization responsibilities.
- Consider reducing direct UniTask exposure at reusable abstraction boundaries.

## Conclusion

- [ ] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: Services is primarily **Game Layer** and is currently a mixed responsibility bucket. Some capabilities should later be promoted into dedicated Framework infrastructure modules, but the current concrete services should not be treated as universal Framework code.
