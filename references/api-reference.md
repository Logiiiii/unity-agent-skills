# Jahro Unity API Reference

> Last verified: Jahro 1.0.0-beta6 | Unity 2021.3+

## Table of Contents

- [Namespace and Imports](#namespace-and-imports)
- [Attributes](#attributes)
  - [JahroCommand](#jahrocommand)
  - [JahroWatch](#jahrowatch)
- [Runtime API — Jahro Static Class](#runtime-api)
  - [State and Lifecycle](#state-and-lifecycle)
  - [View Control](#view-control)
  - [Launch Button](#launch-button)
  - [Object Registration](#object-registration)
  - [Dynamic Command Registration](#dynamic-command-registration)
  - [Command Unregistration](#command-unregistration)
  - [Events](#events)
- [Supported Types](#supported-types)
  - [Command Parameter Types](#command-parameter-types)
  - [Watcher Display Types](#watcher-display-types)
- [Configuration Reference](#configuration-reference)
  - [Jahro Settings](#jahro-settings)
  - [Preprocessor Defines](#preprocessor-defines)
- [Lifecycle and Initialization](#lifecycle-and-initialization)

---

## Namespace and Imports

```csharp
using JahroConsole;
```

All Jahro attributes and the `Jahro` static class live in the `JahroConsole` namespace.

---

## Attributes

### JahroCommand

Marks a method as a console command executable from the Jahro interface.

**Constructors:**

```csharp
[JahroCommand]
[JahroCommand(string name)]
[JahroCommand(string name, string group)]
[JahroCommand(string name, string group, string description)]
```

**Parameters:**

| Parameter | Type | Default | Notes |
|:----------|:-----|:--------|:------|
| `name` | string | Method name | Unique command identifier. Spaces are auto-removed. Convention: kebab-case. |
| `group` | string | `"Default"` | Visual grouping category in both Visual and Text mode. |
| `description` | string | `""` | Shown in Visual Mode command cards. |

**Target:** Static or instance methods. Instance methods require the owning object to be registered via `Jahro.RegisterObject()`.

**Supported method signatures:**
- `void Method()` — no parameters
- `void Method(T1)` — one parameter
- `void Method(T1, T2)` — two parameters
- `void Method(T1, T2, T3)` — three parameters
- `string Method(...)` — return value displayed in console log

**Overloads:** Multiple methods with the same command name but different parameter signatures are supported. Text Mode resolves using intelligent parameter mapping.

**Examples:**

```csharp
[JahroCommand("spawn-enemy", "Spawning", "Spawn enemy at position")]
public void SpawnEnemy(Vector3 position) { /* ... */ }

[JahroCommand("spawn-enemy", "Spawning", "Spawn N enemies")]
public void SpawnEnemy(int count) { /* ... */ }

[JahroCommand]  // name inferred as "Ping"
public static void Ping() { Debug.Log("pong"); }

[JahroCommand("get-pos")]
public static string GetPosition() => $"X:{Player.X} Y:{Player.Y}";
```

### JahroWatch

Marks a field or property for real-time monitoring in the Watcher interface.

**Constructors:**

```csharp
[JahroWatch]
[JahroWatch(string name)]
[JahroWatch(string name, string group)]
[JahroWatch(string name, string group, string description)]
```

**Parameters:**

| Parameter | Type | Default | Notes |
|:----------|:-----|:--------|:------|
| `name` | string | Member name (leading `_` stripped) | Display name in the Watcher UI. |
| `group` | string | `"Default"` | Visual grouping category. |
| `description` | string | `""` | Shown in the detail modal when tapped. |

**Target:** Fields or properties (static or instance). Instance members require the owning object to be registered via `Jahro.RegisterObject()`.

**Examples:**

```csharp
[JahroWatch("Health", "Player", "Current health points")]
public float health = 100f;

[JahroWatch("Position", "Player")]
public Vector3 Position => transform.position;

[JahroWatch]  // name inferred as "playerHealth" (leading _ stripped)
private float _playerHealth;

[JahroWatch("FPS", "Performance", "Frames per second")]
public static float fps => 1f / Time.unscaledDeltaTime;
```

---

## Runtime API

All methods are on the static class `JahroConsole.Jahro`.

### State and Lifecycle

```csharp
public static bool Enabled { get; }
public static bool IsOpen { get; }
public static bool IsReleased { get; }
public static bool IsLaunchButtonEnabled { get; }
```

| Property | Description |
|:---------|:------------|
| `Enabled` | `true` if Jahro is active in this build (reflects settings + preprocessor + auto-disable evaluation) |
| `IsOpen` | `true` if the console window is currently visible |
| `IsReleased` | `true` after `Release()` has been called |
| `IsLaunchButtonEnabled` | `true` if the launch button feature is enabled |

```csharp
public static void InitializeIfNeeded();  // Auto-invoked via [RuntimeInitializeOnLoadMethod]
public static void Release();             // Flush state, dispose logger, destroy UI, end session
```

### View Control

```csharp
public static void ShowConsoleView();   // Open the console window
public static void CloseConsoleView();  // Close the console window
```

Aliases: `Jahro.Show()` maps to `ShowConsoleView()`, `Jahro.Close()` maps to `CloseConsoleView()`.

### Launch Button

The floating launch button is draggable and persists position across sessions.

```csharp
public static void EnableLaunchButton();   // Enable launch button feature
public static void DisableLaunchButton();  // Disable launch button feature
public static void ShowLaunchButton();     // Show if enabled
public static void HideLaunchButton();     // Hide if enabled
```

### Object Registration

Scans an object instance for `[JahroCommand]` and `[JahroWatch]` attributes and registers them.

```csharp
public static void RegisterObject(object obj);
public static void UnregisterObject(object obj);
```

**Canonical usage pattern:**

```csharp
void OnEnable()  => Jahro.RegisterObject(this);
void OnDisable() => Jahro.UnregisterObject(this);
```

**Rules:**
- Passing `null` logs an error and does nothing.
- Always unregister to prevent memory leaks.
- One `RegisterObject` call handles both commands and watchers on the same instance.
- Static members with `[JahroCommand]` or `[JahroWatch]` are discovered via assembly scanning and do not need `RegisterObject`.

### Dynamic Command Registration

Register commands at runtime using delegates instead of attributes.

**Register command to instance method by name:**

```csharp
public static void RegisterCommand(string name, object obj, string methodName);
public static void RegisterCommand(string name, string description, object obj, string methodName);
public static void RegisterCommand(string name, string description, string groupName, object obj, string methodName);
```

**Register delegates — no parameters:**

```csharp
public static void RegisterCommand(string name, Action callback);
public static void RegisterCommand(string name, string description, Action callback);
public static void RegisterCommand(string name, string description, string groupName, Action callback);
```

**Register delegates — one parameter:**

```csharp
public static void RegisterCommand<T>(string name, Action<T> callback);
public static void RegisterCommand<T>(string name, string description, Action<T> callback);
public static void RegisterCommand<T>(string name, string description, string groupName, Action<T> callback);
```

**Register delegates — two parameters:**

```csharp
public static void RegisterCommand<T1, T2>(string name, Action<T1, T2> callback);
public static void RegisterCommand<T1, T2>(string name, string description, Action<T1, T2> callback);
public static void RegisterCommand<T1, T2>(string name, string description, string groupName, Action<T1, T2> callback);
```

**Register delegates — three parameters:**

```csharp
public static void RegisterCommand<T1, T2, T3>(string name, Action<T1, T2, T3> callback);
public static void RegisterCommand<T1, T2, T3>(string name, string description, Action<T1, T2, T3> callback);
public static void RegisterCommand<T1, T2, T3>(string name, string description, string groupName, Action<T1, T2, T3> callback);
```

**Parameter order for the full overload:** `(name, description, groupName, callback)`. This differs from the attribute constructor order `(name, group, description)`.

**Examples:**

```csharp
Jahro.RegisterCommand("gc-collect", "Maintenance", "Force GC", () => System.GC.Collect());
Jahro.RegisterCommand<int>("add-lives", "Cheats", "Add lives", amount => Player.Lives += amount);
Jahro.RegisterCommand<int, float>("set-mult", "Tuning", "Set damage and cooldown",
    (damageMult, cooldown) => { Tuning.Damage = damageMult; Tuning.Cooldown = cooldown; });
```

**Register command on object method:**

```csharp
var ctrl = FindObjectOfType<GameRuntime>();
Jahro.RegisterCommand("restart-level", "Game", "Restart", ctrl, nameof(GameRuntime.RestartLevel));
```

### Command Unregistration

```csharp
public static void UnregisterCommand(string name);
public static void UnregisterCommand(string name, string groupName);
```

### Events

```csharp
public static UnityEngine.Events.UnityAction OnConsoleShow;
public static UnityEngine.Events.UnityAction OnConsoleHide;
```

Fired when the console window opens/closes. Common use: pause/resume game.

```csharp
Jahro.OnConsoleShow += () => Time.timeScale = 0f;
Jahro.OnConsoleHide += () => Time.timeScale = 1f;
```

---

## Supported Types

### Command Parameter Types

These types are supported as method parameters for `[JahroCommand]` methods and `RegisterCommand<T>()` delegates:

| Type | Text Mode Input | Visual Mode Input |
|:-----|:---------------|:-----------------|
| `int` | `42` | Number field |
| `float` | `3.14` | Decimal field |
| `bool` | `true` / `false` | Toggle switch |
| `string` | `hello world` | Text field |
| `Vector2` | `1.5 2.0` | X/Y fields |
| `Vector3` | `10 2.5 -7` | X/Y/Z fields |
| `enum` (any) | `Hard` (name) | Dropdown selector |

**Overload resolution:** When multiple signatures exist for the same command name, Text Mode resolves by parameter count and type conversion success.

### Watcher Display Types

| Type | List View | Detail Modal |
|:-----|:----------|:-------------|
| `int`, `float`, `double`, `bool` | Value as-is | Same |
| `string` | Truncated | Full text |
| `Vector2` | Compact coords | Coords + magnitude |
| `Vector3` | Compact coords | Coords + magnitude |
| `Quaternion` | Raw values | Raw + Euler angles |
| `Transform` | Position summary | Position, rotation, scale, child count |
| `Rigidbody` | Summary | Mass, kinematic, gravity, velocity, angular velocity |
| `Collider` | Summary | Trigger status, material, bounds |
| `AudioSource` | Summary | Clip, volume, loop, pitch, mute |
| `Camera` | Summary | FOV, clip planes, aspect ratio |
| Arrays (any) | `TypeName[length]` | Full contents |

**Performance:** Values are only read when the Watcher UI tab is visible. No continuous polling when the console is closed.

**Error handling:** Null values, exceptions, and invalid access attempts display error messages without crashing.

---

## Configuration Reference

### Jahro Settings

Access via **Tools → Jahro Settings** in Unity Editor.

**Account Tab:**

| Setting | Description |
|:--------|:------------|
| API Key | Project authentication key from console.jahro.io |
| Project | Current project identifier (read-only after key validation) |
| Team | Associated team (read-only) |

**General Settings Tab:**

| Setting | Default | Description |
|:--------|:--------|:------------|
| Enable Jahro | ON | Master switch for all Jahro functionality |
| Auto-disable in Release Builds | OFF | Disables Jahro when `Debug.isDebugBuild == false` |
| Launch Key | `~` (Tilde) | Keyboard shortcut to open console in Play Mode |
| Enable Keyboard Shortcuts | ON | Alt+1 (Text), Alt+2 (Visual), Alt+3 (Watcher), Alt+4 (Snapshots) |
| Mobile Tap Activation | ON | Triple-tap at top of screen to open console |
| Assemblies Selection | Auto | Which assemblies to scan for Commands and Watchers (max 31) |

**Snapshots Settings Tab:**

| Mode | Behavior |
|:-----|:---------|
| Recording | Local-only until manual upload |
| Streaming — Except Editor | Stream on devices, record locally in Editor |
| Streaming — All | Stream everywhere including Editor |

**Settings file:** `Assets/Jahro/Resources/jahro-settings.asset` — include in version control for team consistency.

### Preprocessor Defines

| Define | Effect |
|:-------|:-------|
| `JAHRO_DISABLE` | Force-disable Jahro regardless of all other settings. Add in **Player Settings → Scripting Define Symbols**. |

---

## Lifecycle and Initialization

### Startup Evaluation Order

```
1. JAHRO_DISABLE defined?           → Disabled (highest priority)
2. Auto-disable ON + Release build? → Disabled
3. Enable Jahro switch?             → Uses switch value
```

### Initialization Phases

**Phase 1 — Before Splash Screen** (`RuntimeInitializeLoadType.BeforeSplashScreen`):
- Load project settings from `jahro-settings.asset`
- Evaluate enable/disable conditions
- If disabled: log "Jahro Console: Disabled in this build" and exit early
- If enabled: prepare logging infrastructure

**Phase 2 — Runtime Boot** (second `RuntimeInitializeOnLoadMethod`):
- Initialize console storage and UI view
- Verify API key, start session
- Inject UI manager (window, tabs, launch button)
- Bind hotkeys (`~`, Alt+1/2/3/4) and mobile triple-tap

### Shutdown

Triggered on app quit, assembly reload, or explicit `Jahro.Release()`:
- Save console state (window size/position, favorites)
- Dispose logger
- Destroy UI view
- End Jahro session

### Session Management (Snapshots)

- 10 most recent snapshots kept locally with automatic rotation
- Session statuses: Recording → Recorded → Uploading → Uploaded (or Streaming → Streamed)
- Editable titles (up to 100 characters)
- Streaming sends data in ~5-second interval chunks with duplicate log collapsing
