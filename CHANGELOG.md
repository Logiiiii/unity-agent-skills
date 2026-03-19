# Changelog

## 1.0.0 — Initial Release

**Target Jahro version:** 1.0.0-beta6+

### Skills

- **jahro-setup** — Installation, API key configuration, feature overview, project detection
- **jahro-commands** — `[JahroCommand]` attribute authoring, dynamic registration, group organization
- **jahro-watcher** — `[JahroWatch]` attribute authoring, supported types, performance guidance
- **jahro-snapshots** — Snapshot mode selection, QA capture workflow, team setup
- **jahro-production** — Three-tier disable mechanism, CI/CD guidance, production checklist
- **jahro-troubleshooting** — Decision trees for commands not appearing, watcher not updating, console not opening, snapshots failing, launch button missing
- **jahro-migration** — Migration from IMGUI menus, custom loggers, cheat systems, performance HUDs

### Reference Files

- **api-reference.md** — Complete Jahro API: attributes, runtime API, supported types, configuration, lifecycle
- **common-patterns.md** — RegisterObject lifecycle, combined commands+watchers, static vs instance, dynamic registration, anti-patterns

### Evaluation Infrastructure

- 21 eval scenarios (3 per skill) in `evals/scenarios.md`
- Machine-readable eval definitions in `evals/evals.json`
- Baseline gap analysis in `evals/baseline-results.md`

### Distribution

- README with installation instructions for Claude Code, Cursor, and generic AI assistants
