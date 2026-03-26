# example-plugin

A starter template demonstrating the Sound in Theory plugin structure. Copy and adapt this when building a new plugin.

## Install

```
/plugin install example-plugin@soundintheory
```

## Skills

### `example-skill` (model-invoked)

Claude activates this automatically when it detects a relevant context. Edit the `description` in `SKILL.md` to control when it fires.

### `/example-command` (user-invoked)

A slash command you can call directly:

```
/example-command hello world
```

## Structure

```
example-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── skills/
    ├── example-skill/
    │   └── SKILL.md        # Model-invoked — triggers from context
    └── example-command/
        └── SKILL.md        # User-invoked — /example-command slash command
```
