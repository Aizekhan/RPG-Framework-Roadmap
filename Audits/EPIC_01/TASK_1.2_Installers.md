# Audit — Core/Installers

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Core/Installers

## Name

Core/Installers

## 1. Is it needed by any RPG?

A composition/installers layer is useful for a modular RPG Framework. It should provide a clean composition root mechanism for registering Framework modules and application modules.

## 2. Does it depend on a specific game?

Yes, strongly in the current implementation.

The installer directory registers concrete MythHunter gameplay, UI, networking, heroes, lobby, phases, resources and debug systems.

## 3. Can it be reused without changes?

The base installer pattern can be reused. The current InstallerRegistry and most installers cannot, because they hardcode MythHunter module composition.

## 4. Framework or Game Layer?

**Mixed, but predominantly Game/Application composition.**

DIInstaller and IDIInstaller are Framework candidates. InstallerRegistry and concrete system installers are Game/Application composition.

## 5. Are there architectural problems?

- InstallerRegistry is a static global composition root with a hardcoded ordered list of concrete game installers.
- It catches exceptions and continues boot, which can leave the container partially configured.
- CoreInstaller registers Framework, Application and Game Layer services together.
- CoreInstaller registers GameFlowManager, GameSettingsService and GamePhaseProvider inside the supposed core installer.
- EventsInstaller replaces an already registered IEventBus implementation based on whether networking is registered; this makes registration order part of runtime semantics.
- NetworkingInstaller, ResourceInstaller and EntitiesInstaller register concrete subsystems directly from Core/Installers, creating strong cross-module coupling.
- UIInstaller registers concrete MythHunter models and presenters as part of the generic Core installer area.
- HeroSystemsInstaller mixes hero domain, resources, UI, persistence and system grouping.
- GameplaySystemsInstaller directly defines MythHunter phases and combat/movement groups.
- LobbySystemsInstaller registers presenters inside a systems installer, mixing presentation and gameplay composition.
- Some installers register systems and immediately resolve them, making registration itself create objects and trigger constructor side effects.
- DIExtensionsInstaller creates Unity GameObjects during installation, so composition is no longer purely container configuration.
- LazyDependencyInjectorInstaller also creates a persistent Unity GameObject during registration.
- ParallelSystemsInstaller overrides the ISystemRegistry registration, which is another order-sensitive replacement.
- Naming/location is inconsistent: files are shown under CoreServices/SystemsGroups but several comments and paths reference older locations.

## 6. What does it depend on?

The installer layer depends on a very large portion of the codebase:
- Core.DI
- ECS
- Events
- Systems and system groups
- Networking
- Resources
- Serialization
- Entities / Archetypes / Heroes
- UI
- Game settings / Game flow
- Scene management
- Debug tools
- Phase system
- UnityEngine

## 7. Who depends on it?

GameBootstrapper depends on InstallerRegistry as the main composition root. The resulting registrations provide dependencies to most runtime modules.

## 8. Are there unnecessary dependencies?

Yes, extensively.

The most important architectural issue is that Framework composition and Game composition are not separated. Core/Installers currently knows about almost every major subsystem.

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Split FrameworkInstaller composition from GameInstaller composition.
- Keep the generic installer contract in Framework.
- Replace static InstallerRegistry with explicit composition-root configuration.
- Make installer ordering explicit through dependency declarations rather than a fragile hardcoded sequence.
- Fail fast on critical installer failures rather than continuing with a partially configured container.
- Avoid object creation during registration where possible.
- Move UI, hero, lobby and gameplay installers out of Core.
- Make optional modules such as Networking explicitly opt-in.
- Separate pure DI registration from Unity GameObject creation.
- Remove duplicate/replacement registrations such as SimpleEventBus → EventBus/NetworkEventBus from unrelated installers; select the implementation at composition time.
- Add installer tests validating the final registration graph and required dependencies.

## Conclusion

- [x] Framework
- [x] Game Layer
- [x] Technical Debt

Classification: **Mixed**. The installer abstraction is Framework infrastructure, but the current Core/Installers folder is effectively the MythHunter application composition root and violates the Framework/Game boundary by registering concrete game modules throughout Core.
