# claude-skills

Personal Claude Code plugin marketplace.

    /plugin marketplace add laforcem/claude-skills
    /plugin install skills@laforcem-skills

Add a skill: put it in `plugins/skills/skills/<name>/SKILL.md`, bump `version` in
`plugins/skills/.claude-plugin/plugin.json`, push. Then `/plugin marketplace update` on each machine.
