# Audit — Utils

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Utils

## Name

Utils

## 1. Is it needed by any RPG?

Small shared utilities are useful in any RPG, but a generic Framework should avoid a large catch-all Utils module. Reusable utilities should be moved next to the subsystem they support or kept as narrowly defined infrastructure libraries.

## 2. Does it depend on a specific game?

Partially.

Validation and general math/task helpers are broadly reusable, while MythLogger is explicitly MythHunter-oriented and UnityApiUtils is Unity-specific infrastructure.

## 3. Can it be reused without changes?

Partially.

- Ensure / Validator are broadly reusable.
- MathExtensions are reusable only within Unity.Mathematics/Unity types.
- MythTaskExtensions are tied to UniTask.
- UnityApiUtils is tied to Unity.
- IMythLogger/MythLogger are coupled to MythHunter naming and Unity logging/file APIs.

## 4. Framework or Game Layer?

**Mixed, with a strong recommendation to dissolve Utils into dedicated Framework modules.**

Potential Framework candidates:
- validation primitives
- selected math extensions
- async helpers as an optional UniTask adapter
- logging abstraction
- Unity-specific utility adapters

The MythHunter-specific logger implementation is not universal Framework core.

## 5. Are there architectural problems?

- Utils is a catch-all namespace containing unrelated concerns.
- IMythLogger is explicitly named MythLogger/MythHunter, reducing portability.
- MythLogger combines console formatting, categories, context, enrichers, file logging, rotation and Unity/device metadata in one class.
- MythLogger uses UnityEngine directly, so the logging implementation cannot be used outside Unity.
- MythLoggerFactory provides a global static service locator-like access path alongside DI.
- Log category list contains many game-specific categories such as Rune, Phase and Character.
- UnityApiUtils is explicitly Unity infrastructure but sits in generic Utils.
- MathExtensions mixes UnityEngine.Vector3 and Unity.Mathematics.float3 conversion with math operations.
- MythTaskExtensions exposes and depends directly on UniTask.
- ValidationResult is mutable through its public Errors list.

## 6. What does it depend on?

Observed:
- UnityEngine
- Unity.Mathematics
- Cysharp.Threading.Tasks
- System.IO / reflection-like runtime metadata through logging helpers
- Unity application/device APIs

Validation itself has minimal dependencies.

## 7. Who depends on it?

The logger is broadly used throughout current Framework/Game code. Other utility groups are likely used by systems, networking, UI and services. Exact consumer mapping belongs to later module-level audits.

## 8. Are there unnecessary dependencies?

Yes, candidates include:
- Framework code depending on MythLogger naming and implementation details.
- Global MythLoggerFactory beside DI.
- Game-specific log categories inside generic infrastructure.
- UnityEngine references in utility code that could be isolated.
- Direct UniTask dependency in generic async extensions.
- Math conversion helpers placed in a broad Utils namespace.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Remove the catch-all Utils concept over time.
- Split validation, logging, math, async and Unity adapters into dedicated modules.
- Rename/recast the logging abstraction as generic Framework logging.
- Keep Unity console/file implementation behind the logging abstraction.
- Remove or constrain global logger factory usage in favor of DI.
- Move game-specific categories to Game Layer configuration.
- Make ValidationResult immutable or expose read-only errors.
- Keep Unity-specific helpers outside the pure Framework core.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Mixed**. Several utilities are valid Framework candidates, but the current Utils directory is a catch-all and contains Unity- and MythHunter-specific implementations. It should be decomposed rather than preserved as a single Framework module.
