# VesselSense Claude Plugins

Claude Code plugin marketplace for Signal K marine development, published at `github.com/VesselSense/vesselsense-claude-plugins`.

## What This Repo Is

A Claude Code **plugin marketplace** — a collection of reusable skills, commands, and agents that anyone can install into their Claude Code environment for Signal K development. Users add the marketplace once, then install individual plugins.

```
/plugin marketplace add VesselSense/vesselsense-claude-plugins
/plugin install signalk-kip@vesselsense-claude-plugins
```

## Repo Structure

```
.claude-plugin/
  marketplace.json              # Registry manifest — lists all plugins
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json               # Plugin manifest (name, version, description, author)
    skills/
      <skill-name>/
        SKILL.md                # Skill definition (frontmatter + content)
        *.md                    # Supporting reference files
    commands/                   # Optional: slash commands
      <command-name>.md
    agents/                     # Optional: custom subagents
      <agent-name>.md
```

## Current Plugins

| Plugin | Type | Contents |
|--------|------|----------|
| **signalk-kip** | Dashboard automation | 1 skill (4 files), 1 command, 1 agent |
| **signalk-sensesp** | Framework reference | 1 skill (1 file) |
| **wilhelmsk-dashboard** | Layout generation | 1 skill (3 files) |

## Adding a New Plugin

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "<plugin-name>",
     "description": "One-line description",
     "version": "1.0.0",
     "author": { "name": "VesselSense", "url": "https://github.com/VesselSense" }
   }
   ```

2. Add skills under `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` with frontmatter:
   ```yaml
   ---
   name: <skill-name>
   description: Keywords and trigger phrases for model invocation.
   ---
   ```

3. Optionally add commands (`commands/*.md`) and agents (`agents/*.md`).

4. Register the plugin in `.claude-plugin/marketplace.json` by adding an entry to the `plugins` array:
   ```json
   {
     "name": "<plugin-name>",
     "source": "./plugins/<plugin-name>",
     "description": "Same one-line description"
   }
   ```

5. Update `README.md` with the new plugin's install command and description.

## Modifying an Existing Plugin

- Edit files in place under `plugins/<plugin-name>/`.
- Bump the `version` in `plugin.json` when making user-facing changes.
- Keep the description in `plugin.json` and `marketplace.json` in sync.

## Quality Rules

These are mandatory for all plugin content:

### No Personal or Vessel-Specific Content

This is a public, community-facing repo. Never include:
- Personal file paths (`/Users/...`, `/home/...`)
- Specific vessel names, IPs, hostnames, or credentials
- References to specific local project directories

### No Hardcoded Server URLs

Signal K servers run at different addresses. Never hardcode `localhost:3000` or any specific hostname in actionable code/instructions. Instead:
- Use `{SIGNALK_URL}` as a placeholder
- Include a setup step that asks the user for their server URL (via AskUserQuestion or checking conversation context)
- The only acceptable `localhost` references are: example prompts showing the user what to type, and documenting default config values

### Standalone Content

Skills must work when installed into **any** project, not just a specific repo. Do not reference:
- Source file paths from other repos (e.g., `src/interfaces/plugins.ts`)
- Other skills that aren't in the same plugin (no cross-plugin `Related Skills` sections)
- Local tool configurations that won't exist in the user's environment

Supporting files within the same skill directory can reference each other by relative name (e.g., "see `widgets.md`").

### Skill Frontmatter

Every `SKILL.md` must have frontmatter with at minimum `name` and `description`. The description is used for model-invocation matching — make it keyword-rich.

Optional frontmatter:
- `triggers` — list of phrases that trigger model invocation
- `disable-model-invocation: true` — user-only invocation
- `allowed-tools` — restrict which tools the skill can use
- `context: fork` — run in isolated subagent

### Agent Definitions

Agents in `agents/*.md` should:
- Scope their MCP servers (don't pollute the main conversation)
- List explicit tool access in `tools:`
- Reference skills by name in `skills:` (skills in the same plugin)
- Include a Step 0 that resolves any required context (server URL, credentials, etc.)

## Git Conventions

- One logical change per commit
- Commit message format: imperative mood, concise subject line
- Bump `plugin.json` version in the same commit as the change
- Do not commit `.DS_Store` or other OS artifacts

## Naming Conventions

- Plugin names: `signalk-*` for Signal K ecosystem tools, `wilhelmsk-*` for WilhelmSK tools
- Skill names match their parent plugin name (e.g., plugin `signalk-kip` has skill `signalk-kip`)
- Command names are short verbs or nouns (e.g., `kip`)
- Agent names are descriptive with a dash (e.g., `kip-dashboard`)
- All names use kebab-case

## Testing a Plugin Locally

After making changes, test by installing from the local checkout:

```
/plugin marketplace add /path/to/vesselsense-claude-plugins
/plugin install <plugin-name>@vesselsense-claude-plugins
```

Verify:
- Skills appear in the skill list and trigger correctly
- Commands are available via `/<command-name>`
- Agents can be delegated to by the model
- No broken references to files or external resources
