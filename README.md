# Sound in Theory — Claude Code Plugin Marketplace

A shared collection of Claude Code plugins for the Sound in Theory development team.

## Installation

### Add this marketplace to Claude Code

Run the following slash command in Claude Code:

```
/plugin marketplace add soundintheory/claude-plugins
```

This registers the marketplace under the name `soundintheory`. You only need to do this once.

### Install a plugin

Browse available plugins:

```
/plugin
```

Or install directly by name:

```
/plugin install <plugin-name>@soundintheory
```

For example:

```
/plugin install example-plugin@soundintheory
```

### Update plugins

```
/plugin update
```

---

## Contributing a Plugin

1. Create a new directory under `plugins/<your-plugin-name>/`
2. Follow the structure below
3. Open a pull request — plugins are available to the team once merged to `main`

### Plugin Structure

```
plugins/
└── your-plugin-name/
    ├── .claude-plugin/
    │   └── plugin.json          # Required metadata
    ├── README.md                # Plugin documentation
    ├── skills/
    │   ├── your-skill/
    │   │   └── SKILL.md         # Model-invoked skill (context-aware guidance)
    │   └── your-command/
    │       └── SKILL.md         # User-invoked slash command (/your-command)
    └── .mcp.json                # Optional MCP server configuration
```

### plugin.json

```json
{
  "name": "your-plugin-name",
  "description": "What this plugin does",
  "author": {
    "name": "Your Name",
    "email": "you@soundintheory.co.uk"
  }
}
```

### Skill types

**Model-invoked** — Claude activates the skill automatically based on task context. The `description` field controls when it triggers:

```yaml
---
name: your-skill
description: This skill should be used when the user mentions "X", asks about "Y", or is working on Z.
version: 1.0.0
---
```

**User-invoked** — Creates a `/your-command` slash command the user can call explicitly:

```yaml
---
name: your-command
description: Short description shown in /help
argument-hint: <required> [optional]
allowed-tools: [Read, Glob, Grep, Bash]
version: 1.0.0
---
```

See [example-plugin](./plugins/example-plugin/) for a working reference implementation.
