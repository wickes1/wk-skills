# `wk-skills` — Project Plan

A personal-but-open Agent Skills library: every entry is one `SKILL.md`, discovered natively by Claude Code (and OpenCode via `.claude/skills/` compat).

---

## §1 Goal & non-goals

**Goal:** One source of truth for gotchas / procedures / patterns. AI agents discover them via standard skill loading; humans read them as articles.

**Open-source posture:** Skill content is generic. No secrets, no machine names, no personal identifiers, no homeserver hostnames. Anything user-specific is expressed as a placeholder variable (see §4 *Inputs convention*) and bound at runtime by a separate private skill or the user.

**Non-goals:**
- Not a global skill install — repo-scoped, attached explicitly via `--add-dir` or marketplace.
- No human-facing site here (separate Astro repo may consume these later).
- No hand-maintained `INDEX.md` / `llms.txt`.

---

## §2 Structure

```
wk-skills/
├── README.md                                 # what this is, how to add
├── PROJECT.md                                # this file
├── CLAUDE.md                                 # agent onboarding
├── .claude-plugin/
│   └── marketplace.json                      # remote-add via /plugin marketplace add
└── .claude/skills/                           # canonical store (Claude + OpenCode both read this)
    └── <domain>-<topic-slug>/
        └── SKILL.md
```

**Skill folder naming:** `<domain>-<topic-slug>`, kebab-case. `domain` = system/tool area (`element`, `claude-code`, `mcp`, `tailscale`, ...). Domain prefix gives free grouping via glob.

---

## §3 `SKILL.md` authoring spec

One `SKILL.md` per folder, at depth 1.

### Frontmatter

| Field | Required | Notes |
|---|---|---|
| `name` | yes | Match folder name. Lowercase kebab-case, ≤64 chars. |
| `description` | yes | **One line, ~15–20 tokens, trigger-shaped.** Both tools route on this. |
| extra fields | no | Both Claude Code and OpenCode ignore unknown frontmatter. |

### Body

- Open with one-line **TL;DR**.
- If user-specific values are required: a **Required inputs** table immediately after the TL;DR (see §4).
- Standing guidance ("When X, do Y"), not narration.
- Reads as both a human article and actionable instructions.
- Size budget: ≤~1,500–2,000 words. Body is recurring token cost once triggered.
- Gotcha template: `## Symptom` → `## Cause` → `## Fix` → `## Notes`.

### Description rule (load-bearing)

Skills' `name` + `description` are loaded at session start (~100 tokens each). The description decides whether the skill is found. Keep it one line, trigger-shaped. Body stays unloaded until triggered.

---

## §4 Inputs convention (composition with private skills)

These skills are public. Any value that varies per user — tokens, hostnames, user IDs, file paths — is expressed as a **placeholder shell variable** prefixed with `$`, declared in a **Required inputs** table at the top of the body. The skill itself never names a specific user, machine, or secret location.

Resolution happens at runtime, in one of two ways:

1. **Private companion skill.** The user has a separate (private, not in this repo) skill that lists their values: "for any Matrix skill, `$ACCESS_TOKEN` comes from secrets store X, `$HOMESERVER` is `…`, `$USER_ID` is `…`". Both skills load together; the AI binds and proceeds.
2. **User-supplied at call time.** If no private skill provides the binding, the AI asks the user for each input. Never guess values.

**Variable naming:** use the canonical names from the domain's own documentation when one exists (e.g., Matrix tutorials use `$ACCESS_TOKEN`, `$HOMESERVER`, `$USER_ID` — match those). If you invent a name, make it generic and obvious.

**What never appears in a skill in this repo:**
- Real homeserver hostnames, IP addresses, Tailscale machine names.
- Real Matrix IDs, GitHub handles other than the repo owner's public one, email addresses.
- Real env var names that encode a user's identity (e.g., `MATRIX_WEEK_ACCESS_TOKEN` → use generic `$ACCESS_TOKEN`).
- Paths under `/Users/<name>/`, `~/Documents/<personal-folder>/`, etc. Use `/path/to/<thing>` or `~/<thing>` placeholders.
- References to private notes ("from Joplin X note", "see my dotfiles") — those belong in the private skill.

---

## §5 Conventions

- One concern per skill. No bundling.
- Folder name == `name` frontmatter.
- No tool-specific assumptions in the body unless the skill is explicitly about that tool.
- No hand-maintained index — the tree + frontmatter is the index.

---

## Appendix — one-time decisions

Recorded so future-me doesn't undo them.

**Layout: `.claude/skills/` direct, no canonical `skills/` + symlinks.** Claude Code is the primary consumer; OpenCode also natively reads `.claude/skills/`. Migration to a canonical form later = one `git mv` + one `ln -s`. Trivial. No `scripts/link-views.sh`, no `AGENTS.md → CLAUDE.md` symlink (OpenCode falls back to `CLAUDE.md` when `AGENTS.md` is absent).

**Description style: lean over pushy.** The upstream `skill-creator` recommends *pushy* descriptions ("Make sure to use this skill whenever…") to combat under-triggering in a crowded global skill pool. This repo defaults to **lean** — these skills are personal-first, attached deliberately via `--add-dir`, so startup tax matters more than competing for attention. If a specific skill later needs broader reach (distributed via marketplace and competing in a hundred-skill context), tune that one description with `skill-creator`'s `run_loop.py` description optimizer rather than flipping the repo-wide default.

**Scale tripwire.** Negligible below a few dozen skills. If count hits hundreds and `/context` shows real bloat, demote the cold, rarely-triggered, purely-passive long tail to plain markdown + one navigator skill. Not a day-one concern.
