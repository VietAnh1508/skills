# Skills Repo

This repo is a collection of custom Claude Code skills installable via `npx skills add VietAnh1508/skills`.

## Structure

Each skill lives in its own directory:

```
socratic-tutor/
├── SKILL.md          — Claude Code skill (loaded by the skill system)
├── PROMPT.md         — standalone copy-paste prompt for non-Claude-Code users
└── references/
    └── examples.md   — annotated session dialogues (for skill authors, not loaded at runtime)

ux-audit/
├── SKILL.md          — Claude Code skill (orchestrator)
└── references/       — executor instructions, scenario template, static review guide
```

## Key file roles

**`*/SKILL.md`** — The Claude Code skill entry point. Loaded into context automatically when the skill triggers. Keep under 500 lines; heavy reference material goes in `references/` and is loaded on demand.

**`socratic-tutor/PROMPT.md`** — A self-contained version of the socratic-tutor skill for people who can't or won't use Claude Code. Designed to be copy-pasted as a system prompt into any AI chat (ChatGPT, Gemini, Claude.ai, etc.). Must stay fully standalone — no `references/` or progressive disclosure available in that context.
