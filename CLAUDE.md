# Agent onboarding — `wk-skills`

This repo is a personal Skills library. Each `.claude/skills/<name>/SKILL.md` is one self-contained Agent Skill.

## When acting in this repo

- Read `PROJECT.md` for the authoring spec.
- Skills are discovered natively from `.claude/skills/` — don't add manual indexes.
- Adding a skill:
  1. New folder under `.claude/skills/<domain>-<topic>/` + `SKILL.md`.
  2. Folder name must match the `name:` frontmatter field.
  3. **Append `"./.claude/skills/<domain>-<topic>"` to `.claude-plugin/marketplace.json` → `plugins[0].skills`.** Required for plugin-install users; `--add-dir` and OpenCode don't need this step but the marketplace path does.

## Description rule (do not violate)

`description:` is one line, ~15–20 tokens, **trigger-shaped**. It's the field both Claude Code and OpenCode route on. Examples:

- Good: `Fix Element notification sound not applying; for odd Matrix client notification behaviour`
- Bad: `This skill explains how to change the notification sound in Element Desktop, a Matrix client.`

The description states *when to load*, not what the skill is about.

## Body structure (gotcha template)

```
## Symptom    — what you observe
## Cause      — the underlying reason
## Fix        — standing procedure, valid as a human how-to
## Notes      — version this was true for; when it stops being true
```

Patterns / procedures that aren't gotchas can use different headings. The gotcha template is just the default.

## Writing style

- **Imperative.** "Run X", not "you should run X".
- **Explain the why.** A capable agent makes better edge-call decisions when it knows the rationale. Walls of `MUST` / `NEVER` are a yellow flag — reframe with the reason behind the rule, unless it's genuinely non-negotiable (secrets, destructive ops).
- **Theory of mind, not rote scripts.** Skills get reused in contexts you didn't anticipate. General principles + one worked example beats a narrow click-by-click walkthrough.

## When a skill outgrows one SKILL.md

The body is loaded in full once triggered. If a skill covers multiple variants (per-cloud, per-OS) or grows past the size budget, split into the standard subdirs and reference them from SKILL.md so they load on demand:

```
<skill-name>/
├── SKILL.md          # entrypoint + selection logic
├── scripts/          # executable helpers — invoked, not read into context
├── references/       # supplementary docs, loaded on-demand from SKILL.md
└── assets/           # output templates, fonts, icons, etc.
```

SKILL.md should point at the right reference for the situation ("for AWS, read `references/aws.md`"). The whole point of the split is to avoid loading everything upfront — keep SKILL.md to the routing logic plus what's universally needed.

## Public repo — no secrets, no personal info

This repo is open source. Skill content must be generic enough that any AI working for any user can apply it. Anything user-specific (tokens, hostnames, Matrix IDs, GitHub handles other than the repo owner's, paths under `/Users/<name>/`, env var names that encode an identity like `MATRIX_WEEK_ACCESS_TOKEN`) is a **placeholder** — declared in a `## Required inputs` table at the top of the body, prefixed with `$`. Bound at runtime by a private companion skill or asked from the user. See PROJECT.md §4.

## Size budget

≤~1,500–2,000 words per skill body. Once triggered, the whole body lands in context.
