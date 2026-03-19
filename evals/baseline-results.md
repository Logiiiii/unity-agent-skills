# Baseline Results — Gap Analysis

Analysis of what Claude knows vs. what it gets wrong about Jahro **without** any skills loaded.

## Methodology

Assessed against the 21 eval scenarios from `scenarios.md`. Gaps categorized as:
- **Critical**: Claude generates incorrect code that won't compile or causes bugs
- **Significant**: Claude misses Jahro-specific patterns, invents APIs, or gives incomplete guidance
- **Minor**: Claude gets the gist right but lacks specificity or best practices

## Overall Finding

Claude understands C# attributes, MonoBehaviour lifecycle, and general Unity patterns well. The primary gaps are **Jahro-specific API surface knowledge** — exact constructor signatures, correct parameter ordering for dynamic registration, supported types, configuration menu paths, and the existence/naming of specific features like snapshot modes.

## Per-Skill Gap Analysis

### jahro-setup

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| Unity Package Manager exists | Doesn't know Jahro's exact Git URL |
| General concept of API keys | Doesn't know console.jahro.io or the settings path |
| Debug consoles exist | Doesn't know ~ key or triple-tap activation |

**Critical gaps**: Menu path `Tools → Jahro Settings`, package ID `io.jahro.console`, activation methods.

### jahro-commands

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| C# attributes with parameters | May invent wrong constructor overloads |
| MonoBehaviour lifecycle (OnEnable/OnDisable) | May not know RegisterObject/UnregisterObject pattern |
| Static vs instance methods | May not know Jahro requires registration for instance commands |
| General naming conventions | May not know kebab-case is the convention |

**Critical gaps**: `[JahroCommand(name, group, description)]` exact constructor; `Jahro.RegisterObject(this)` pattern; dynamic registration parameter order `(name, description, groupName, callback)` — Claude may swap description and group; supported parameter types list.

### jahro-watcher

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| C# field/property attributes | Doesn't know JahroWatch constructor |
| General monitoring concepts | Doesn't know supported type list or display behavior |
| Performance concepts | Doesn't know "only reads when UI visible" optimization |

**Critical gaps**: `[JahroWatch(name, group, description)]` constructor; shared RegisterObject for both commands and watchers (deduplication); supported types table; performance model.

### jahro-snapshots

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| General concept of log capture | Doesn't know Recording vs Streaming modes |
| Team collaboration concepts | Doesn't know specific snapshot workflow |
| Mobile debugging needs | Doesn't know session statuses or ~5s streaming interval |

**Critical gaps**: Three snapshot modes (Recording, Streaming All, Streaming Except Editor); session status lifecycle; 10-session local rotation; QA capture workflow specifics; web console integration.

### jahro-production

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| Preprocessor defines in Unity | Doesn't know JAHRO_DISABLE specifically |
| Development vs Release builds | Doesn't know three-tier evaluation order |
| Debug.isDebugBuild concept | Doesn't know auto-disable setting name/location |

**Critical gaps**: `JAHRO_DISABLE` define name; three-tier priority order (define → auto-disable → manual); `Tools → Jahro Settings → General Settings` path; "Jahro Console: Disabled in this build" validation message; `BeforeSplashScreen` lifecycle.

### jahro-troubleshooting

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| General debugging techniques | Doesn't know Jahro-specific diagnostic paths |
| Missing registration as common issue | May not know assembly scanning configuration |
| Attribute-based systems need registration | Doesn't know specific error messages or settings paths |

**Critical gaps**: Assembly scanning in Jahro Settings; specific decision tree branches; Launch Button enable/disable distinction; configuration conflict patterns (JAHRO_DISABLE + expecting Jahro to work).

### jahro-migration

| What Claude Knows | What Claude Gets Wrong |
|:---|:---|
| IMGUI patterns (OnGUI, GUI.Button) | May not map correctly to Jahro patterns |
| Debug.Log is Unity's logging | Doesn't know Jahro intercepts Debug.Log automatically |
| Custom command systems exist | May invent a Jahro.Log() API that doesn't exist |

**Critical gaps**: Jahro intercepts Debug.Log automatically (no wrapper needed); correct mapping table (IMGUI → commands, labels → watchers, custom parsers → JahroCommand); what to keep vs. remove; incremental migration approach.

## Summary: What Each Skill Must Teach

| Skill | Must Teach |
|:---|:---|
| jahro-setup | Package detection, menu paths, activation methods, API key flow |
| jahro-commands | Exact attribute constructors, RegisterObject pattern, dynamic registration overloads, parameter types |
| jahro-watcher | Attribute constructors, shared registration, supported types table, performance model |
| jahro-snapshots | Three modes, QA workflow, session statuses, streaming behavior, web console light touch |
| jahro-production | JAHRO_DISABLE, three-tier priority, validation message, BeforeSplashScreen |
| jahro-troubleshooting | Decision trees per issue type, assembly scanning, specific fix snippets |
| jahro-migration | Decision guide (what maps to what), auto-intercept for Debug.Log, incremental approach |
