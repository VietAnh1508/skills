# Known Limitations

## `Session: fresh` detection is heuristic

**Issue:** The executor detects an active session by checking whether the app redirects to a sign-in page after loading the App URL. This fails for apps that handle auth without a redirect:

- **SPAs that silently load user data** — no redirect occurs, but the home screen renders differently when authenticated. An existing session will not be detected as BLOCKED.
- **Apps with a public landing page at the root URL** — the landing page is not a sign-in page, so the executor may falsely report BLOCKED when there is no session at all (or, depending on the app, silently pass through with a live session).

**Impact:** If an active session exists and isn't detected, the audit runs from an authenticated state instead of a cold start. Results for sign-in and onboarding flows will be invalid.

**Workaround:** Manually sign out in the browser before running any `Session: fresh` scenario. The executor's BLOCKED check is a safety net, not a guarantee.

**Future fix:** Supplement the redirect check with a JS inspection of `localStorage`, `sessionStorage`, cookies, or an app-specific DOM marker (e.g. `[data-user]`, `#app[data-authenticated]`) to detect session state more reliably across SPA architectures.
