---
name: ux-audit
description: >
  Produces a structured, ranked UX audit report for a single user scenario,
  covering: user journey friction, visual design (typography, colour, spacing,
  consistency), accessibility (contrast, focus indicators, keyboard navigation,
  ARIA labels), copy quality (labels, error messages, microcopy), interaction
  feedback, and mobile readiness.

  Always invoke this skill when the user asks to audit the UX, review a user
  journey, check usability, assess look and feel, test accessibility, or
  evaluate the design of any screen or flow — even if they don't use the word
  "audit". Invoke for any request involving usability review, design feedback,
  first-time-user experience, or accessibility checks on a running app or
  shared screenshots.
---

# UX Audit Skill — Orchestrator

This skill orchestrates UX audits. It reads 1 or N scenarios, spawns an executor sub-agent per scenario (running in parallel), collects their findings files, and synthesises the final report.

For static reviews (screenshots only), no sub-agents are used — see [Static review execution](#static-review-execution).

---

## Pre-flight checks

### 1. Verify Chrome extension

Load the Chrome tab tool via ToolSearch:

```
select:mcp__claude-in-chrome__tabs_context_mcp
```

Call `tabs_context_mcp`. If the call errors or times out, the extension is unavailable — tell the user to install the Claude in Chrome extension and grant it permissions for the app's origin, then retry. If they'd rather not, offer a [static review](#static-review-execution) instead. Do not proceed with a live audit until the extension check passes.

### 2. Confirm with the user

Once the extension check passes, ask the user to confirm before proceeding:

- The app server is running at the URL(s) in the scenario(s)
- The test account exists and does **not** require a password change on login

---

## Input: three modes

### Mode A — Scenario file

```
/ux-audit path/to/my-scenario.md
```

Read the file and extract the scenario config. Scenario file format — see `./references/scenario-template.md` (relative to this skill's directory).

### Mode A (multi) — Directory of scenario files

```
/ux-audit path/to/scenarios/
```

Use `find` or `ls` to list all `*.md` files in the directory, excluding any `*-audit.md` files (those are outputs, not inputs). Each file is one scenario — run them all.

### Mode B — Inline description

User describes the scenario in conversation. Extract app URL, credentials, and journey steps. If anything is missing (especially app URL or auth), ask before proceeding. Example:

> "Audit the sign-up flow on http://localhost:3000. It's a task management app for small teams. Use email: test@example.com / password: testpass123. Start from the landing page as a brand-new visitor."

### Mode C — Static review (screenshots only)

Used when the user directly shares screenshots without a running app. Skip the pre-flight checklist and orchestrator steps entirely — go straight to [Static review execution](#static-review-execution).

---

## Orchestrator steps

### 0. Resolve skill directory

Note the absolute path of the directory containing this SKILL.md file — call it `<skill-dir>`. Use it wherever these instructions reference `<skill-dir>`.

### 1. Collect scenarios

Read every scenario file (Mode A / A-multi) or parse the inline description (Mode B). For each scenario, compute a slug for its findings file:

- Scenario name → lowercase, spaces to hyphens, e.g. `core-loop`
- Findings path: `/tmp/ux-audit-<slug>-findings.md`

### 2. Spawn executor agents

For each scenario, spawn one Agent with this prompt:

```
Skill directory: <skill-dir>
Read `<skill-dir>/references/executor.md` for your full instructions.

Scenario config:
  App URL: <value>
  App Name: <value>
  App Persona: <value>
  Auth: <value>
  Session: <value>
  Viewport: <value>

Scenario steps:
<paste the scenario steps verbatim>

Write your findings to: /tmp/ux-audit-<slug>-findings.md
```

**Spawn all agents in the same message** so they run in parallel. Do not wait for one to finish before spawning the next.

### 3. Wait and collect

Once all agents have completed, read each findings file. Check the `Status:` field in each:

- `OK` — proceed
- `ERROR` — Chrome extension was not available for that scenario. Tell the user which scenario failed and offer a static review for it.
- `BLOCKED` — an active session was detected. Tell the user to sign out and re-run that scenario.

### 4. Synthesise and write the report

Read the report template from `<skill-dir>/assets/report-template.md`. Fill in `{date}` with today's date (YYYY-MM-DD format).

**Single scenario:** Format the findings file into the single-scenario template. Output path:
- Use the `Output:` field from the scenario file if present
- Otherwise write to `<scenario-directory>/<scenario-basename>-audit.md`
- Otherwise write to `UX_AUDIT.md` in the current working directory

**Multiple scenarios:** Use the multi-scenario template section. Identify cross-scenario findings — issues where the same element and dimension appear in 2 or more scenarios' findings. List those first in a "Cross-scenario findings" section, then include each scenario's remaining findings underneath. Output path defaults to `UX_AUDIT.md`; honour an `Output:` field on the first scenario file if present.

---

## Static review execution

Use this path for Mode C — evaluating screenshots instead of a live browser session.

### 1. Gather screenshots

If the user has already shared images, proceed. If not, ask:

> "Please share screenshots of each key screen in the flow, labeled by state (e.g. 'Sign-in page', 'Home after login', 'Error state'). If you have before/after shots for any interactions, include both."

### 2. Evaluate each screenshot

Apply these dimensions — all work from screenshots alone:

| Dimension              | Evaluable? |
| ---------------------- | ---------- |
| Visual hierarchy       | ✅ Yes |
| Visual design          | ✅ Yes |
| CTA clarity            | ✅ Yes |
| Empty / loading states | ✅ Yes (if screenshot shows it) |
| Feedback after actions | ✅ If before/after shots provided — otherwise note the gap |
| Information density    | ✅ Yes |
| Copy quality           | ✅ Yes |
| Mobile readiness       | ✅ If mobile screenshots provided |
| Friction               | ✅ Infer from number of steps visible |
| Confusion              | ✅ Yes |

### 3. Accessibility — partial coverage only

Only colour contrast and tap target size can be evaluated from screenshots. Note this once at the top of any accessibility findings:

> _DOM-level accessibility checks (unlabelled elements, alt text, heading structure, keyboard focus) require a live session and could not be run in this static review._

### 4. Write the report

Use the single-scenario report template. Append `(Static review)` to the scenario name in the title. Add one sentence in the Executive Summary noting that DOM-level accessibility checks were not performed.

---

## Rules (for synthesis step)

- Every finding must name an **exact UI element** with a **concrete suggestion**.
- Skip dimensions that are fine — do not pad with neutral observations.
- Cross-scenario deduplication: if the same issue appears in multiple scenarios, list it once in the cross-scenario section and reference it by name in each scenario's section.
- Prioritise issues on the **critical path** (sign in → core action → confirmation).
- Do not include screenshots in the report file — text only.
