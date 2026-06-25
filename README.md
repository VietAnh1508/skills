# Custom Skills

A collection of custom Claude Code skills. Each skill is a slash command that extends Claude's behavior in a specific, reusable way.

## Skills

| Skill | Command | Description |
|---|---|---|
| [socratic-tutor](./socratic-tutor/) | `/socratic-tutor` | A Socratic learning companion that only asks questions — never explains. Activate by saying "I just learned about X" or "I want to learn about X". |
| [ux-audit](./ux-audit/) | `/ux-audit` | AI-powered UX audit. Give it a scenario and it navigates the app like a first-time user, captures screenshots at each key state, and writes a structured findings report. |

---

## socratic-tutor

A learning companion that never explains anything. It only asks questions — to help you surface, stress-test, and consolidate what you know, or to help you find your footing on something unfamiliar.

Works for any topic: code, science, history, economics, medicine, philosophy, anything.

### How to start

**If you've just learned something and want to test your understanding:**

> "I just learned about X, let me explain it to you."
> "Today I learned how X works."

The tutor will ask where to start, then follow your explanation with questions that probe for gaps and push you to reason through edge cases.

**If you're curious about something but don't know where to start:**

> "I want to learn about X."
> "I keep hearing about X but I don't really know what it is."

The tutor won't lecture. It'll ask what drew you to the topic, what you already suspect, what you know that might be adjacent — until a natural starting point emerges.

### What to expect

- You'll only get questions back, never explanations or confirmations.
- Questions follow directly from what you said — not generic prompts from a template.
- The tutor won't tell you if you're right or wrong. It'll ask another question instead.
- At the end of the session, you'll recap what you covered in your own words and identify where your explanation felt shaky.

Type `skip` or `?` to move past a question. The tutor will set it aside and continue from a different angle.

### Tips

- **Narrow the topic before you start.** "How LLMs work" is a multi-session topic. Pick one piece — attention, tokenization, RLHF — and go deep on that.
- **Speak out loud (or write like you are).** Short answers get shallow follow-ups.
- **Wrong answers are fine.** You won't be corrected — you'll be questioned. That's how gaps surface.
- **Use it right after learning something.** Works best when the material is fresh and you're still slightly uncertain.

---

## ux-audit

### Quick start

**Single scenario file:**
```
/ux-audit path/to/my-scenario.md
```

**All scenarios in a directory (runs in parallel):**
```
/ux-audit path/to/scenarios/
```

**Inline — describe the scenario in your message:**
```
/ux-audit  ← then describe what to test
```

**Static review — share screenshots directly:**
```
/ux-audit  ← then attach screenshots
```

### How it works

Two layers run under the hood:

1. **Orchestrator** — reads 1 or N scenarios, spawns one executor sub-agent per scenario in parallel, waits for all to finish, then synthesises the final report.
2. **Executor** — each executor handles one scenario: loads browser tools, walks the journey, evaluates each screen, and writes a structured findings file.

For multiple scenarios the orchestrator deduplicates cross-scenario findings and writes one combined report.

### Creating a scenario file

1. Copy `~/.claude/skills/ux-audit/references/scenario-template.md` into your project (e.g. `.claude/ux-audit/my-scenario.md`)
2. Fill in the app URL, credentials, and journey steps
3. Run `/ux-audit .claude/ux-audit/my-scenario.md`

Keep scenario files in your project — they describe app-specific flows.

### Scenario file fields

| Field | Required | Description |
|---|---|---|
| `App URL` | Yes | Base URL of the running app |
| `App Name` | Yes | Short name used in the report title |
| `App Persona` | Yes | One sentence describing the app and its users |
| `Auth` | No | `email: x / password: y` — omit for public pages |
| `Session` | No | `fresh` (default) or `authenticated` |
| `Viewport` | No | `desktop` (default) or `mobile` (390px) |
| `Output` | No | Report path — defaults to `UX_AUDIT.md` |

### Requirements

- App server running at the specified URL
- [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude-in-chrome/aaocglkjkiohbbkgdibmeenknfghiobf) installed with permissions for the app's origin
- Test account that doesn't require a password change on first login
