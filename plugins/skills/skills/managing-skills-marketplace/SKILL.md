---
name: managing-skills-marketplace
description: Use when adding, editing, renaming, or removing a personal skill that should sync across machines, or when a skill change isn't showing up on another machine. Applies to the laforcem/claude-skills plugin marketplace.
---

# Managing the skills marketplace

Personal skills live in the public GitHub repo `laforcem/claude-skills` (marketplace `laforcem-skills`, plugin `skills`). Edit the repo, never the installed plugin cache.

## Layout

```
.claude-plugin/marketplace.json            # marketplace catalog
plugins/skills/.claude-plugin/plugin.json  # plugin manifest (no `version`; updates key off the commit SHA)
plugins/skills/skills/<name>/SKILL.md      # one folder per skill
```

## Workflow

1. **Find the clone.** Default `~/claude-skills`. If missing, `gh repo clone laforcem/claude-skills ~/claude-skills`. Run `git pull` first.
2. **Make the change** under `plugins/skills/skills/<name>/`. The folder name must equal `name:` in the frontmatter. To add a skill, write `SKILL.md` with `name` and a `description` that starts "Use when...". For a skill that already exists in `~/.claude/skills/`, copy it in, then delete the local copy so it doesn't load twice.
3. **Check for leaks.** The repo is public. Read the diff for credentials, internal hostnames, employer names, and client data. Work-specific skills do not belong here.
4. **Validate:** `claude plugin validate <repo path>`. Fix errors; the missing-author warning is fine.
5. **Commit and push.** Ask the user before pushing.
6. **Pick up on each machine:** `/plugin marketplace update laforcem-skills`, then `claude plugin update skills@laforcem-skills`, then `/reload-plugins` (or restart).

## Common mistakes

| Mistake | Result |
|---|---|
| Added a `version` field to `plugin.json` | Pins the plugin; other machines stop updating until it's bumped |
| Edited `~/.claude/plugins/cache/...` | Change is overwritten on next update |
| Folder name differs from `name:` | Skill loads under an unexpected name |
| Left a copy in `~/.claude/skills/` | Duplicate skill |
| Pushed a work-specific skill | Leaked to a public repo |
