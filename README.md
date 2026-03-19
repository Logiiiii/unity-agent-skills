# Jahro Agent Skills

AI agent skills that help Unity developers set up, use, troubleshoot, and migrate to the [Jahro Console](https://jahro.io) debugging platform.

These skills turn your AI coding assistant into a Jahro expert — generating correct code patterns, diagnosing issues, and guiding feature adoption.

## Skills Included


| Skill                     | What It Does                                                  |
| ------------------------- | ------------------------------------------------------------- |
| **jahro-setup**           | Installation, API key config, feature overview                |
| **jahro-commands**        | `[JahroCommand]` attribute authoring and command organization |
| **jahro-watcher**         | `[JahroWatch]` attribute authoring and variable monitoring    |
| **jahro-snapshots**       | Snapshot capture modes, QA workflow, team setup               |
| **jahro-production**      | Production safety: JAHRO_DISABLE, auto-disable, lifecycle     |
| **jahro-troubleshooting** | Diagnose common issues with decision trees                    |
| **jahro-migration**       | Migrate from custom debug UIs, loggers, cheat systems         |


## Installation

### Claude Code (native SKILL.md support)

Clone directly into your project's skills directory:

```bash
git clone https://github.com/jahro/agent-skills.git .agents/skills/jahro
```

Or clone to a specific location and symlink:

```bash
git clone https://github.com/jahro/agent-skills.git ~/jahro-agent-skills
ln -s ~/jahro-agent-skills .agents/skills/jahro
```

Claude Code automatically discovers skills in `.agents/skills/` and activates them based on the `description` field in each SKILL.md.

### Cursor

Copy skill files into Cursor's rules directory:

```bash
git clone https://github.com/jahro/agent-skills.git /tmp/jahro-skills

# Copy each skill
for skill in /tmp/jahro-skills/skills/*/SKILL.md; do
  name=$(basename $(dirname "$skill"))
  cp "$skill" ".cursor/rules/${name}.md"
done

# Copy reference files
mkdir -p .cursor/rules/jahro-references
cp /tmp/jahro-skills/references/*.md .cursor/rules/jahro-references/

# Cleanup
rm -rf /tmp/jahro-skills
```

In Cursor Settings, mark `jahro-setup.md` as **Always** so it provides context for all Jahro-related queries. Other skill rules can be left as **Auto**.

### Generic AI Assistants

Place the `skills/` and `references/` directories in your project root or any location your AI assistant scans for markdown context files. Most assistants that support project-level context will pick up the SKILL.md files automatically.

## Requirements

- **Unity**: 2021.3.0f1 or later
- **Jahro**: 1.0.0-beta6+ (skills target current stable)
- **AI Assistant**: Any assistant that supports markdown context files

## How It Works

Each skill is a standalone SKILL.md file with:

- **YAML frontmatter** containing `name` and `description` (triggers skill activation)
- **Markdown body** with concise instructions, code templates, and links to shared reference files

Skills reference two shared files in `references/`:

- `api-reference.md` — complete Jahro API (attributes, methods, events, types)
- `common-patterns.md` — reusable code patterns and anti-patterns

The AI reads only what it needs: the triggered skill (~300-450 lines) plus reference files on demand.

## Updating

When Jahro releases new features:

```bash
cd .agents/skills/jahro  # or wherever you cloned
git pull
```

Skills are versioned alongside Jahro releases. See [CHANGELOG.md](CHANGELOG.md) for what changed.

## License

MIT