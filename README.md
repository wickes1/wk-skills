# `wk-skills`

Personal Agent Skills library. One source of truth for gotchas, procedures, and patterns — discovered natively by Claude Code (and OpenCode via `.claude/skills/` compat).

## Use from another project

```bash
claude --add-dir /path/to/wk-skills
```

Or install as a Claude Code plugin (two steps):

```
/plugin marketplace add wickes1/wk-skills
/plugin install wk-skills@wk-skills
```

`--add-dir` is the primary path — it tracks the live repo, and OpenCode also reads `.claude/skills/` natively. The plugin install caches a snapshot; pull new skills with `/plugin marketplace update`.

## Add a skill

```
.claude/skills/<domain>-<topic>/SKILL.md
```

Folder name == frontmatter `name`. Description is one line, ~15–20 tokens, trigger-shaped. See `PROJECT.md` for the full spec, `CLAUDE.md` for agent onboarding.

## On secrets

Skills here are **public and generic**. Anything user-specific (tokens, hostnames, IDs, paths) is a placeholder variable declared in a **Required inputs** table. Resolution happens at runtime via a separate private skill, env vars, or the user. See PROJECT.md §4.
