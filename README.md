# claude-plugins

My Claude Code plugin marketplace: reusable skills I install per project.

| Plugin | Contents | Use in |
|---|---|---|
| `craft` | Skills `tdd`, `design-review`; agent `reviewer` (read-only: Read, Grep, Glob, Bash) | Any backend project |
| `nestjs` | Skill `nestjs-patterns` | NestJS projects |

Skills shape how the main agent works while it builds. Review is an agent instead of a skill so it runs in its own context with a tool list that cannot edit files, independent of the code it judges.

## Install into one project

Run inside the project folder:

```bash
claude plugin marketplace add Abdelwahab313/claude-plugins --scope project
claude plugin install craft@abdelwahab-plugins --scope project
claude plugin install nestjs@abdelwahab-plugins --scope project
```

This writes the marketplace and enabled plugins into the project's `.claude/settings.json`, so only that project gets them. Each teammate still installs the plugins themselves after trusting the folder.

Skills are namespaced by plugin, e.g. `/craft:tdd`, `/craft:design-review`, `/nestjs:nestjs-patterns`. Claude also loads them on its own when the task matches the skill description.

## Add a skill or plugin

- New skill in an existing plugin: `plugins/<plugin>/skills/<skill-name>/SKILL.md` with `name` and `description` frontmatter.
- New plugin: `plugins/<name>/.claude-plugin/plugin.json` plus a `skills/` folder, then add an entry to `.claude-plugin/marketplace.json`.
- Bump `version` in the plugin's `plugin.json` so installed copies pick up the change.

Test locally before pushing:

```bash
claude plugin validate .
claude plugin marketplace add ./ --scope project   # from a test project, using the absolute path to this repo
```
