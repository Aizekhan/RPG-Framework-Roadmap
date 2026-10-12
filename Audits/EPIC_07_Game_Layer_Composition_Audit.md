# EPIC 07 — Game Layer & Composition Audit

**Date**: 2026-10-12
**Status**: Audit in progress
**Repository**: MythHunter at commit `6637cca`

---

## Executive Summary

This audit examines MythHunter's current composition architecture to establish clear Framework ↔ Game boundaries and enable modular, plug-and-play feature composition (Lobby, Networking, Solo mode, etc.).

**Key findings:**
- ✅ Centralized bootstrap exists (`GameBootstrapper.cs`)
- ✅ DI-based composition via `InstallerRegistry`
- ❌ All installers are hardcoded - no conditional module loading
- ❌ Framework and Game concerns mixed in same installers
- ❌ No abstraction for optional features (Lobby, Networking, Multiplayer)

---

## Current Composition Architecture

### Entry Point: GameBootstrapper.cs

**Location**: `Assets/_MythHunter/Code/Core/Game/GameBootstrapper.cs`

**Responsibilities:**
1. Create DI container
2. Register all installers via `InstallerRegistry.RegisterInstallers()`
3. Initialize ECS World
4. Initialize Game State Machine
5. Initialize systems by category (`OnBoot`)
6. Inject dependencies into scene
7. Start game flow

**Key code:**
```csharp
private void InitializeCore()
{
    var logger = MythLogger.CreateDefaultLogger();
    _container = new DIContainer(logger);

    _container.BindSingleton<IMythLogger>(logger);
    _container.BindSingleton<RPGFramework.Logging.ILogger>(logger);
    _container.RegisterInstance(_eventNetworkMode);  // Offline/Client/Server

    InstallerRegistry.RegisterInstallers(_container);  // ⚠️ All installers always registered

    _logger = _container.Resolve<IMythLogger>();
    _eventBus = _container.Resolve<IEventBus>();
    _dependencyInjector = _container.Resolve<IDependencyInjector>();
}
```

**Network Mode**: Has `EventNetworkMode` enum (Offline/Client/Server) but **only used for EventBus**, not for feature composition.

---

### Installer Registry: InstallerRegistry.cs

**Location**: `Assets/_MythHunter/Code/Core/Installers/InstallerRegistry.cs`

**Current approach:**
```csharp
public static void RegisterInstallers(IDIContainer container)
{
    var installers = new DIInstaller[]
    {
        new LazyDependencyInjectorInstaller(),
        // Core services (OnBoot)
        new CoreInstaller(),
        new NetworkingInstaller(),          // ⚠️ Always registered
        new EventsInstaller(),
        new ResourceInstaller(),
        new SerializationInstaller(),
        new UIInstaller(),
        new PoolInstaller(),
        new EntitiesInstaller(),
        new DebugToolsInstaller(),
        new HeroSystemsInstaller(),

        // System groups (by category)
        new ProfileInstaller(),
        new NetworkingSystemsInstaller(),  // ⚠️ Always registered
        new LobbySystemsInstaller(),       // ⚠️ Always registered (even for solo!)
        new GameplaySystemsInstaller(),
        new DebugSystemsInstaller(),
    };

    foreach (var installer in installers)
    {
        installer.InstallBindings(container);  // ⚠️ No conditional logic
    }
}
```

**Problems:**
1. **All installers always run** - no way to disable Lobby for solo game
2. **No dependency graph** - order is manual, error-prone
3. **Framework/Game mixed** - CoreInstaller has both Framework and Game concerns
4. **Hardcoded list** - can't extend without modifying this file

---

### Core Installer Analysis

**Location**: `Assets/_MythHunter/Code/Core/Installers/CoreServices/CoreInstaller.cs`

**Registered types:**
```csharp
// Framework Core (should be separate)
BindSingleton<IEntityManager, EntityManager>(container);       // ✅ Framework
BindSingleton<ISystemRegistry, SystemRegistry>(container);     // ✅ Framework
BindSingleton<IEcsWorld, EcsWorld>(container);                 // ⚠️ Game adapter?

// Game-specific (should be separate)
BindSingleton<IEventPool, EventPool>(container);               // ❌ Game
BindSingleton<IEventBus, SimpleEventBus>(container);          // ❌ Game
BindSingleton<IGameStateMachine, GameStateMachine>(container); // ❌ Game
BindSingleton<IPhaseProvider, GamePhaseProvider>(container);   // ❌ Game
BindSingleton<IGameFlowManager, GameFlowManager>(container);   // ❌ Game
BindSingleton<IGameSettingsService, GameSettingsService>(container); // ❌ Game
BindSingleton<SceneLoader, SceneLoader>(container);            // ❌ Game
BindSingleton<ISceneDispatcher, SceneDispatcher>(container);   // ❌ Game
BindSingleton<AutoPreloadConfigurator, AutoPreloadConfigurator>(container); // ❌ Game
```

**Analysis:**
- ✅ Framework ECS contracts (IEntityManager, ISystemRegistry)
- ❌ Game flow, state machine, scenes - all MythHunter-specific!
- ❌ No separation between reusable Framework and game-specific logic

---

## Installer Categorization

### 📦 Framework Core (should be universal)
- `EntitiesInstaller` - ECS entities and component cache
- *(Parts of CoreInstaller that deal with IEntityManager, ISystemRegistry)*

### 🎮 Game Layer (MythHunter-specific)
- `CoreInstaller` - Game flow, state machine, scene management
- `EventsInstaller` - Game event bus
- `UIInstaller` - Game UI
- `GameplaySystemsInstaller` - Core gameplay systems
- `HeroSystemsInstaller` - Hero-specific systems
- `DebugSystemsInstaller` - Debug/development tools

### 🔌 Optional Features (should be conditional)
- `NetworkingInstaller` - Networking infrastructure
- `NetworkingSystemsInstaller` - Network synchronization
- `LobbySystemsInstaller` - Lobby UI and flow
- `ProfileInstaller` - Player profiles (multiplayer?)

### 🛠️ Infrastructure (neutral, but game-specific)
- `ResourceInstaller` - Asset loading
- `SerializationInstaller` - Save/load
- `PoolInstaller` - Object pooling
- `DebugToolsInstaller` - Development tools

---

## Architecture Problems

### 1. **No Framework/Game Separation**
**Current:**
```
InstallerRegistry (hardcoded list)
├── CoreInstaller (Framework + Game mixed)
├── NetworkingInstaller (always loaded)
└── LobbySystemsInstaller (always loaded)
```

**Problem**: Framework contracts (IEntityManager) mixed with game flow (IGameStateMachine) in same installer.

### 2. **No Conditional Module Loading**
**Current:** All installers always run, regardless of:
- Solo vs Multiplayer
- Offline vs Online
- Lobby vs Direct-to-game
- Development vs Production

**Example:** Even a solo offline game loads:
- `NetworkingInstaller`
- `NetworkingSystemsInstaller`
- `LobbySystemsInstaller`

### 3. **No Module Dependencies**
**Current:** Installers run in hardcoded order, no way to express:
- "Networking requires EventBus"
- "Lobby requires Networking"
- "Save System optional if Networking enabled"

### 4. **No Extensibility**
**Current:** To add new feature module, must:
1. Create new installer
2. Manually add to `InstallerRegistry` array
3. Manually determine correct position in array
4. Hope dependencies are correct

**Desired:** Plugin-style registration:
```csharp
GameBuilder.Create()
    .UseFramework()          // Always present
    .UseMultiplayer()        // Adds networking + lobby
    .UseSaveSystem()         // Adds persistence
    .Build();
```

---

## Target Architecture

### Layered Composition

```
┌─────────────────────────────────────────┐
│         Game Composition Root           │ ← GameBootstrapper
│  (Conditional feature selection)        │
└─────────────┬───────────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
┌───▼────────┐    ┌────▼──────────┐
│ Framework  │    │ Game Layer    │
│ Core       │◄───┤ (MythHunter)  │
│ (Universal)│    │ (Specific)    │
└────────────┘    └───────┬───────┘
                          │
              ┌───────────┴───────────┐
              │                       │
         ┌────▼────────┐      ┌──────▼─────────┐
         │  Optional   │      │   Optional     │
         │  Features   │      │   Features     │
         │  (Lobby,    │      │  (Save, Replay)│
         │  Networking)│      │                │
         └─────────────┘      └────────────────┘
```

### Module Categories

**1. Framework Core (always present)**
```csharp
IFrameworkCompositionRoot
├── IEntityManager
├── ISystemRegistry
├── ILogger
└── IDIContainer
```

**2. Game Layer (MythHunter-specific, always present)**
```csharp
IGameCompositionRoot
├── IGameStateMachine
├── IGameFlowManager
├── IPhaseProvider
└── ISceneManagement
```

**3. Feature Modules (conditional)**
```csharp
IFeatureModule
├── NetworkingModule (INetworkSystem, ILobbySystem)
├── PersistenceModule (ISaveSystem, ILoadSystem)
├── ReplayModule (IReplayRecorder, IReplayPlayer)
└── DebugModule (IDebugService, IDebugDashboard)
```

---

## Proposed Solution

### 1. Composition Builder Pattern

```csharp
public interface IGameCompositionBuilder
{
    IGameCompositionBuilder UseFrameworkCore();
    IGameCompositionBuilder UseGameLayer();
    IGameCompositionBuilder UseFeature<TFeature>() where TFeature : IFeatureModule;
    IGameCompositionBuilder UseNetworking(EventNetworkMode mode);
    IGameCompositionBuilder UseLobby();
    IGameCompositionBuilder UseSaveSystem();
    IGameComposition Build();
}
```

**Usage in GameBootstrapper:**
```csharp
private void InitializeCore()
{
    var logger = MythLogger.CreateDefaultLogger();
    var container = new DIContainer(logger);

    var composition = new GameCompositionBuilder(container)
        .UseFrameworkCore()           // Always
        .UseGameLayer()                // Always for MythHunter
        .UseNetworking(_eventNetworkMode)  // Conditional
        .UseFeature<LobbyModule>()     // Conditional
        .UseFeature<DebugModule>()     // Dev only
        .Build();

    _container = composition.Container;
}
```

### 2. Feature Module Interface

```csharp
public interface IFeatureModule
{
    string Name { get; }
    IEnumerable<Type> Dependencies { get; }  // Other modules this depends on
    void Configure(IDIContainer container);
    void Initialize();
    void Shutdown();
}
```

**Example - Networking Module:**
```csharp
public class NetworkingModule : IFeatureModule
{
    public string Name => "Networking";

    public IEnumerable<Type> Dependencies => new[]
    {
        typeof(CoreModule),    // Needs EventBus
    };

    public void Configure(IDIContainer container)
    {
        container.BindSingleton<INetworkSystem, NetworkSystem>();
        container.BindSingleton<IConnectionManager, ConnectionManager>();
    }

    public void Initialize()
    {
        var networkSystem = _container.Resolve<INetworkSystem>();
        networkSystem.Connect();
    }

    public void Shutdown()
    {
        var networkSystem = _container.Resolve<INetworkSystem>();
        networkSystem.Disconnect();
    }
}
```

### 3. Installer Migration

**Old approach (hardcoded):**
```csharp
InstallerRegistry.RegisterInstallers(container);  // Always all installers
```

**New approach (modular):**
```csharp
var builder = new GameCompositionBuilder(container);

// Framework (always)
builder.UseFrameworkCore();

// Game Layer (always for MythHunter)
builder.UseGameLayer();

// Optional features
if (_enableMultiplayer)
{
    builder.UseNetworking(_networkMode);
    builder.UseLobby();
}

if (_enableSaveSystem)
{
    builder.UseSaveSystem();
}

#if UNITY_EDITOR || DEVELOPMENT_BUILD
builder.UseDebugTools();
#endif

var composition = builder.Build();
```

---

## Implementation Plan

### Phase 1: Audit and Design (Current)
- [x] Map current bootstrap flow
- [x] Categorize all installers (Framework/Game/Optional)
- [x] Identify Framework/Game boundary violations
- [ ] Design composition API
- [ ] Create test plan

### Phase 2: Framework Core Extraction
- [ ] Extract Framework-only concerns from CoreInstaller
- [ ] Create `FrameworkCompositionModule`
- [ ] Separate IEntityManager, ISystemRegistry registration
- [ ] Validate no game dependencies in Framework module

### Phase 3: Game Layer Separation
- [ ] Create `GameLayerCompositionModule`
- [ ] Move IGameStateMachine, IGameFlowManager, IPhaseProvider
- [ ] Move scene management, state machine
- [ ] Ensure Framework → Game dependency (not reverse!)

### Phase 4: Feature Modules
- [ ] Create `IFeatureModule` interface
- [ ] Implement `NetworkingModule`
- [ ] Implement `LobbyModule` (depends on NetworkingModule)
- [ ] Implement `SaveSystemModule`
- [ ] Implement `DebugToolsModule`

### Phase 5: Composition Builder
- [ ] Implement `IGameCompositionBuilder`
- [ ] Implement `GameCompositionBuilder`
- [ ] Add dependency resolution (topological sort)
- [ ] Add validation (detect missing dependencies)

### Phase 6: Bootstrap Integration
- [ ] Update `GameBootstrapper` to use builder
- [ ] Add configuration (ScriptableObject or JSON)
- [ ] Add runtime feature flags
- [ ] Remove old `InstallerRegistry`

### Phase 7: Validation
- [ ] Test solo offline game (no Networking, no Lobby)
- [ ] Test multiplayer online (with Networking + Lobby)
- [ ] Test with/without SaveSystem
- [ ] Architecture tests: validate Framework doesn't depend on optional modules

---

## Success Criteria

### Must Have
- ✅ Framework Core module has **zero** game dependencies
- ✅ Game Layer explicitly depends on Framework (via interfaces)
- ✅ Optional features can be enabled/disabled without code changes
- ✅ Solo offline game doesn't load Networking/Lobby modules
- ✅ Multiplayer game doesn't break when SaveSystem disabled

### Should Have
- ✅ Builder API is intuitive and discoverable
- ✅ Module dependencies are validated at build time
- ✅ Clear error messages for missing dependencies
- ✅ Architecture tests enforce module boundaries

### Nice to Have
- Configuration-driven composition (JSON/ScriptableObject)
- Runtime feature toggling
- Hot-reload support for modules
- Module load/unload without restart

---

## Risks

### 1. Breaking Existing Behavior
**Risk**: Refactoring bootstrap might break game startup
**Mitigation**:
- Keep old `InstallerRegistry` until new system validated
- Run full smoke tests (Movement, Visibility, Hero)
- Incremental migration (one module at a time)

### 2. Circular Dependencies
**Risk**: Module A depends on B, B depends on A
**Mitigation**:
- Topological sort during build
- Fail fast with clear error message
- Architecture tests to prevent cycles

### 3. Over-Engineering
**Risk**: Too complex for current needs
**Mitigation**:
- Start simple (just separate Framework/Game/Optional)
- Add dependency resolution only if needed
- Defer advanced features (hot-reload, config-driven)

---

## Next Steps

1. **Validate scope** with user
2. **Design composition API** (interfaces, builder)
3. **Create prototype** (minimal example with 2-3 modules)
4. **Implement Phase 2** (Framework Core extraction)
5. **Validate** with Architecture Tests

---

## References

- Current code: commit `6637cca`
- Related EPICs: EPIC 01 (Audit), EPIC 05 (Framework Runtime)
- Architecture principles: `ARCHITECTURE-QUICK-REFERENCE.md`
- Similar patterns: Unity Packages, ASP.NET Core modules, Godot autoloads

---

**Last Updated**: 2026-10-12
**Next Review**: After Phase 1 design approval
