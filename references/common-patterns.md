# Jahro Common Patterns

Reusable code patterns for Jahro integration. Skills reference this file for detailed examples.

> Last verified: Jahro 1.0.0-beta6

## Table of Contents

- [Object Registration Lifecycle](#object-registration-lifecycle)
- [Combined Commands and Watchers](#combined-commands-and-watchers)
- [Static vs Instance — Decision Guide](#static-vs-instance)
- [Dynamic Command Registration](#dynamic-command-registration)
- [Watcher Group Organization](#watcher-group-organization)
- [Console Integration](#console-integration)
- [Anti-Patterns](#anti-patterns)

---

## Object Registration Lifecycle

The canonical pattern for registering instance commands and watchers. Always use `OnEnable`/`OnDisable` — not `Awake`/`OnDestroy`.

```csharp
using JahroConsole;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [JahroCommand("heal", "Player", "Restore health")]
    public void Heal(float amount) { health += amount; }

    [JahroWatch("Health", "Player", "Current HP")]
    public float health = 100f;

    void OnEnable()  => Jahro.RegisterObject(this);
    void OnDisable() => Jahro.UnregisterObject(this);
}
```

**Why OnEnable/OnDisable:**
- `OnEnable` runs every time the object becomes active (including after instantiation)
- `OnDisable` runs when the object is disabled or destroyed
- Prevents registering on inactive objects
- Matches Unity's recommended lifecycle for paired setup/teardown

---

## Combined Commands and Watchers

When a class has both `[JahroCommand]` and `[JahroWatch]` attributes, a single `RegisterObject` call handles both. Do not call it twice.

```csharp
public class GameManager : MonoBehaviour
{
    [JahroWatch("Player Count", "Game", "Active players")]
    public int playerCount => PlayerManager.ActiveCount;

    [JahroWatch("Game Time", "Game", "Elapsed time")]
    public float gameTime => Time.timeSinceLevelLoad;

    [JahroCommand("reset-game", "Game", "Reset game state")]
    public void ResetGame() { SceneManager.LoadScene(0); }

    [JahroCommand("set-speed", "Game", "Set game speed")]
    public void SetSpeed(float multiplier) { Time.timeScale = multiplier; }

    void OnEnable()  => Jahro.RegisterObject(this);
    void OnDisable() => Jahro.UnregisterObject(this);
}
```

**If adding watchers to a class that already has commands (and RegisterObject):** Just add the `[JahroWatch]` attributes — no registration changes needed.

---

## Static vs Instance

| Use Static When | Use Instance When |
|:----------------|:-----------------|
| The method doesn't need `this` (e.g., load scene, GC collect, toggle global flag) | The method acts on a specific object (e.g., heal player, teleport character) |
| Only one logical instance exists (e.g., game manager singleton) | Multiple instances exist (e.g., per-enemy or per-player) |
| You want commands available without any scene objects | You need access to instance state (fields, components, transform) |

**Static commands** are discovered via assembly scanning — no `RegisterObject` needed:

```csharp
public class DebugCommands
{
    [JahroCommand("restart-level", "Game", "Restart current level")]
    public static void RestartLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    [JahroCommand("gc-collect", "Maintenance", "Force garbage collection")]
    public static void ForceGC() => System.GC.Collect();
}
```

**Instance commands** require the object to be registered:

```csharp
public class EnemyController : MonoBehaviour
{
    [JahroCommand("kill-enemy", "Debug", "Kill this enemy")]
    public void Kill() { health = 0; Die(); }

    void OnEnable()  => Jahro.RegisterObject(this);
    void OnDisable() => Jahro.UnregisterObject(this);
}
```

---

## Dynamic Command Registration

Use when commands need to be created at runtime rather than compiled with attributes.

### Basic patterns

```csharp
// No parameters
Jahro.RegisterCommand("clear-cache", "Maintenance", "Clear local cache",
    () => PlayerPrefs.DeleteAll());

// One typed parameter
Jahro.RegisterCommand<int>("add-gold", "Cheats", "Add gold",
    amount => Player.Gold += amount);

// Two typed parameters
Jahro.RegisterCommand<float, float>("set-bounds", "Debug", "Set X/Y bounds",
    (x, y) => { Bounds.X = x; Bounds.Y = y; });

// Three typed parameters
Jahro.RegisterCommand<int, int, int>("set-rgb", "Visual", "Set color RGB",
    (r, g, b) => Renderer.color = new Color(r/255f, g/255f, b/255f));
```

### Register command on existing object method

```csharp
var game = FindObjectOfType<GameRuntime>();
Jahro.RegisterCommand("restart", "Game", "Restart level",
    game, nameof(GameRuntime.RestartLevel));
```

### Cleanup

```csharp
Jahro.UnregisterCommand("clear-cache");
Jahro.UnregisterCommand("restart", "Game");  // with group for disambiguation
```

### When to use dynamic vs attribute

| Use Attributes When | Use Dynamic When |
|:-------------------|:----------------|
| Commands are known at compile time | Commands created from runtime data (e.g., level list) |
| Method already exists on a class | Wrapping existing API without modifying classes |
| Standard MonoBehaviour pattern | Non-MonoBehaviour systems (singletons, services) |

---

## Watcher Group Organization

### By system (recommended default)

```csharp
[JahroWatch("Health", "Player")]
[JahroWatch("Stamina", "Player")]
[JahroWatch("Position", "Player")]
[JahroWatch("Velocity", "Physics")]
[JahroWatch("Is Grounded", "Physics")]
[JahroWatch("FPS", "Performance")]
[JahroWatch("Memory", "Performance")]
```

### By priority

```csharp
[JahroWatch("Health", "Critical")]        // things you always need
[JahroWatch("Frame Time", "Critical")]
[JahroWatch("Enemy Count", "Gameplay")]    // game-specific state
[JahroWatch("Spawn Timer", "Gameplay")]
[JahroWatch("GC Allocs", "Diagnostics")]   // technical details
[JahroWatch("Draw Calls", "Diagnostics")]
```

### Static watchers for global state

No `RegisterObject` needed — discovered via assembly scanning:

```csharp
public static class GameStats
{
    [JahroWatch("Total Score", "Game")]
    public static int Score;

    [JahroWatch("High Score", "Game")]
    public static int HighScore;

    [JahroWatch("Session Time", "Game")]
    public static float SessionTime => Time.realtimeSinceStartup;
}
```

### Performance monitoring pattern

```csharp
public static class PerfWatcher
{
    private static float _lastUpdate;
    private static float _cachedFps;

    static PerfWatcher()
    {
        Application.onBeforeRender += _ =>
        {
            if (Time.time - _lastUpdate < 0.25f) return;
            _cachedFps = 1f / Time.unscaledDeltaTime;
            _lastUpdate = Time.time;
        };
    }

    [JahroWatch("FPS", "Performance", "Current frames per second")]
    public static string FPS => _cachedFps.ToString("F1");
}
```

---

## Console Integration

### Pause game on console open

```csharp
void Start()
{
    Jahro.OnConsoleShow += () => Time.timeScale = 0f;
    Jahro.OnConsoleHide += () => Time.timeScale = 1f;
}
```

### Show/hide launch button contextually

```csharp
void OnSceneLoaded(Scene scene, LoadSceneMode mode)
{
    if (scene.name == "MainMenu")
        Jahro.HideLaunchButton();
    else
        Jahro.ShowLaunchButton();
}
```

### Open console from game UI

```csharp
public void OnDebugButtonClicked()
{
    Jahro.ShowConsoleView();
}
```

---

## Anti-Patterns

### Registering in Awake (too early)

```csharp
// BAD: Awake runs before the object is fully active
void Awake() => Jahro.RegisterObject(this);  // may miss some components

// GOOD: OnEnable runs when the object is active
void OnEnable() => Jahro.RegisterObject(this);
```

### Forgetting to unregister

```csharp
// BAD: memory leak — registered commands/watchers reference a destroyed object
void OnEnable() => Jahro.RegisterObject(this);
// OnDisable never called with UnregisterObject

// GOOD: always pair register with unregister
void OnEnable()  => Jahro.RegisterObject(this);
void OnDisable() => Jahro.UnregisterObject(this);
```

### Duplicate registration calls

```csharp
// BAD: calling RegisterObject twice (once for commands, once for watchers)
void OnEnable()
{
    Jahro.RegisterObject(this); // for commands
    Jahro.RegisterObject(this); // for watchers — redundant!
}

// GOOD: one call handles both
void OnEnable() => Jahro.RegisterObject(this);
```

### Debug.Log polling in Update

```csharp
// BAD: spams console, hurts performance
void Update()
{
    Debug.Log($"Health: {health}, Pos: {transform.position}");
}

// GOOD: use watchers instead
[JahroWatch("Health", "Player")]
public float health;

[JahroWatch("Position", "Player")]
public Vector3 Position => transform.position;
```

### Complex logic in command methods

```csharp
// BAD: putting business logic in the command handler
[JahroCommand("spawn-wave", "Spawning", "Spawn enemy wave")]
public void SpawnWave(int count)
{
    for (int i = 0; i < count; i++)
    {
        var enemy = Instantiate(enemyPrefab);
        enemy.Initialize(GetRandomPosition(), defaultDifficulty);
        enemyList.Add(enemy);
        UpdateEnemyCounter();
    }
}

// GOOD: delegate to existing game systems
[JahroCommand("spawn-wave", "Spawning", "Spawn enemy wave")]
public void SpawnWave(int count) => EnemySpawner.SpawnWave(count);
```
