# Custom Skills

A collection of custom Claude Code skills. Each skill is a slash command that extends Claude's behavior in a specific, reusable way.

## Install

```bash
npx skills add VietAnh1508/skills
# Then pick the skills you want, and which coding agents you want to install them on

# Install specific skill
npx skills add VietAnh1508/skills --skill ux-audit
```

## Skills

| Skill                                      | Command           | Description                                                                                                                                                                  |
| ------------------------------------------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [socratic-tutor](./socratic-tutor/DOCS.md) | `/socratic-tutor` | A Socratic learning companion that only asks questions — never explains. Activate by saying "I just learned about X", "I want to learn about X", or "explain it like I'm 5". |
| [ux-audit](./ux-audit/DOCS.md)             | `/ux-audit`       | AI-powered UX audit. Give it a scenario and it navigates the app like a first-time user, captures screenshots at each key state, and writes a structured findings report.    |
