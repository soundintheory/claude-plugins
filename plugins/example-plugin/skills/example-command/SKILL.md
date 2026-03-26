---
name: example-command
description: An example slash command demonstrating the user-invoked skill format
argument-hint: <message> [--option]
allowed-tools: [Read, Glob, Grep]
version: 1.0.0
---

# Example Command (User-Invoked)

This is a user-invoked skill — it runs when someone types `/example-command` in Claude Code.

## Arguments

The user ran this command with: `$ARGUMENTS`

## Instructions

1. Acknowledge the arguments provided above
2. Explain that this is a demo command from the Sound in Theory plugin marketplace
3. Show the user how to find more plugins by running `/plugin`

## How to Adapt This Command

1. **Rename the directory** — `skills/your-command-name/`
2. **Update the frontmatter**:
   - `name`: the slash command will be `/your-command-name`
   - `description`: shown in `/help`
   - `argument-hint`: shown as a hint when the user types `/your-command-name`
   - `allowed-tools`: tools this command is pre-approved to use (reduces permission prompts)
3. **Replace the Instructions section** with the steps Claude should follow when the command runs

## Frontmatter Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Slash command name (e.g. `name: deploy` → `/deploy`) |
| `description` | Yes | Short description shown in `/help` |
| `argument-hint` | No | Shown as a hint in the UI, e.g. `<env> [--dry-run]` |
| `allowed-tools` | No | Pre-approved tools — reduces permission prompts |
| `model` | No | Override model: `haiku`, `sonnet`, or `opus` |
| `version` | No | Semantic version number |