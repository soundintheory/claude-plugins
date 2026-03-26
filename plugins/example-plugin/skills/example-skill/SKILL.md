---
name: example-skill
description: This skill should be used when the user asks to "show an example skill", "demonstrate a skill", "create a plugin skill", or is building a new Sound in Theory plugin. Provides a reference template for the SIT skill format.
version: 1.0.0
---

# Example Skill (Model-Invoked)

This is a model-invoked skill. Claude loads it automatically when the task context matches the `description` in the frontmatter above — no slash command needed.

## Purpose

Use this as a starting point for skills that should activate automatically. Common uses:

- Enforcing project conventions when editing specific file types
- Providing domain context when working in a particular area of the codebase
- Offering guidance when certain patterns or keywords appear in the conversation

## How to Adapt This Skill

1. **Rename the directory** — `skills/your-skill-name/`
2. **Update the frontmatter**:
   - `name`: match the directory name
   - `description`: list the exact phrases, keywords, or situations that should trigger this skill
   - `version`: start at `1.0.0`
3. **Replace this body** with the actual guidance, context, or instructions you want Claude to follow

## Writing a Good Description

The `description` field is how Claude decides when to use this skill. Be specific:

```yaml
description: >
  This skill should be used when the user is editing Umbraco templates, working
  in the Views/ directory, or mentions "Umbraco", "Razor", or "content types".
  Provides SIT conventions for Umbraco CMS development.
```

Include:
- Specific phrases the user might say
- File paths or directories that signal relevance
- Technology names or domain keywords

## Skill Body Guidelines

- State the purpose clearly at the top
- Organise guidance into sections
- Be concrete — give examples, not just principles
- Reference files in `references/` subdirectory for lengthy supporting material
