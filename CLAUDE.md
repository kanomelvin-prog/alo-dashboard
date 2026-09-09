# CLAUDE.md — Alowen Clinical Dashboard
## What This Repo Is
Therapist-facing clinical dashboard for Alowen (alowen.ai).
Deployed at dashboard.alowen.ai via GitHub Pages.
Vanilla HTML + CSS + JS. No build step. No framework. No bundler.
## Project Philosophy — Read This First
- Zero-Knowledge architecture: therapists NEVER see transcripts. Clinical signals only.
- Bridge not container: Alo builds client independence, not dependency.
- No engagement hooks: no streaks, no "we miss you," no gamification. Ever.
- Craft integrity: nothing ships until it's done right. No duct tape.
## Critical Rules — Non-Negotiable
- NEVER expose conversation transcripts in any dashboard view.
- NEVER allow therapists to see raw chat content — Zero-Knowledge is non-negotiable.
- Do not modify Supabase schema — use Supabase Studio.
- Do not touch auth configuration without explicit approval and full test plan.
- Do not add analytics, tracking, or third-party scripts without approval.
- Do not add engagement hooks of any kind.
- Do not switch from raw fetch to Supabase SDK — keep existing pattern.
- API keys and secrets never go in commits. The Supabase anon key in `dev.html`/`index.html` is the only exception — it's the public anon key designed to be exposed client-side.
- Flag any credential-looking strings or public vs. private access issues before committing.
## Session Start — Read These First
Before doing any work, read these files from the Obsidian vault:
1. /Users/k-home/Library/Mobile Documents/com~apple~CloudDocs/Elevate-Humanity/Alowen/docs/Alo-Philosophy.md
2. /Users/k-home/Library/Mobile Documents/com~apple~CloudDocs/Elevate-Humanity/Alowen/docs/Alo-Roadmap.md
3. /Users/k-home/Library/Mobile Documents/com~apple~CloudDocs/Elevate-Humanity/Alowen/docs/Dupre-Meeting-Notes.md
After reading, confirm current version, highest priority, and what we must never build. Then ask what we are working on today.
## Stack
- Frontend: Vanilla HTML/CSS/JS (single file, no build step)
- Hosting: GitHub Pages (auto-deploys on push to main)
- Backend: Supabase via raw REST API (no SDK — raw fetch calls only)
- Auth: Supabase GoTrue (email/password, JWT tokens)
- Supabase URL: https://lelsdezstbnzyxvsvbyx.supabase.co
- CSS design system: Warm Desk + Linen Studio (v3.2)
- Design tokens: --cream, --warm-white, --sage, --sage-light, --sage-hover, --charcoal, --stone
## Database Tables — Actual Production Schema
- `profiles` — all user profiles, contains role field (therapist/individual/guest)
- `therapists` — therapist-specific records
- `therapist_clients` — client records per therapist (invite codes, settings, emergency contacts)
- `homework_cards` — therapist-assigned homework assignments
- `crisis_events` — crisis detection audit trail (read-only in dashboard)
- `session_notes` — therapist session notes
- `session_metadata` — per-session data and summaries
- `client_messages` — client messages to therapist
## File Structure
- `index.html` — production dashboard (~3,516 lines)
- `dev.html` — development copy (edit this, promote to index.html when ready)
- `styles.css` — production stylesheet (~3,097 lines), versioned at ?v=29
- `dev.css` — development stylesheet (edit this, promote to styles.css when ready)
- `CLAUDE.md` — this file
## Working Rules
### Git workflow
**Branch workflow ADOPTED 2026-09-08. This supersedes the earlier "work directly on main" rule.**
- Claude Code never commits to `main` and never pushes to `main`. All work goes to a branch named `auto/<short-topic>`, pushed, and merged to `main` by Kano via PR.
- Rationale for the change: pushing straight to `main` here *is* a production deploy, because GitHub Pages publishes `main`. A branch puts a review step between a code change and dashboard.alowen.ai. This matches how `alo-supabase` has worked since 2026-09-02, and R1 (`auto/r1-signup-rewire`, PR #1) was the first dashboard change to use it.
- Kano merges. Kano promotes. Claude Code does neither.
- Push to `main` triggers a GitHub Pages deploy (allow 1-2 min to propagate).
- Verify every push actually landed: after `git push`, run `git status` and `git log --oneline -3`, and confirm against the remote with `git ls-remote origin <branch>`. A push that only exists locally is not a push.
- Delete an `auto/*` branch only after confirming it is fully merged: `git merge-base --is-ancestor <branch-tip> origin/main`. If that check fails, keep the branch and say why.

### File discipline
- **Never touch production files unless explicitly told to.** Every prompt should name the files to modify.
- Edit `dev.html` / `dev.css`. `index.html` and `styles.css` change only during an explicit promotion.
- Always increment the cache buster version (`?v=N`) when CSS changes.

### Promotion parity checklist
`dev.html` references `dev.css?v=N`; `index.html` must reference `styles.css?v=N`. Copying dev over production without rewriting that line points production at a file that is not deployed, and the dashboard loads unstyled. Run all of these after every promotion, before pushing:

```bash
grep -c 'styles\.css?v=29' index.html   # must be 1 (bump 29 to the new N)
grep -c 'dev\.css' index.html           # must be 0
grep -c 'dev\.css?v=' dev.html          # must be 1
```

Then verify on GitHub, not just locally — the deployed file is the one that matters:

```bash
git ls-remote origin main                                    # commit is on the remote
curl -sI https://dashboard.alowen.ai/styles.css | head -1     # 200, after ~2 min
```

A promotion is not done until the parity greps pass **and** the remote has been checked.

### LIVE CHANGE banner convention
Anything that touches production — a promotion to `index.html` / `styles.css`, a push to `main`, a merged PR, an Edge Function redeploy, applied SQL, or an auth or Supabase configuration change — must be called out with a banner in the summary and in the commit body:

```
LIVE CHANGE — <what changed> — affects <who/what> — verify with <command or URL>
```

State it before doing the thing where possible, and always after. The point is that a change reaching real therapists is never buried in the middle of a paragraph of routine work. Routine dev-file edits do not get a banner; if everything is banner-worthy, nothing is.
### Commit messages
- No required format. Plain descriptive messages are fine.
- Examples: `Phase 1A: Share journal entry toggle`, `Fix: timeline badge refreshes immediately`, `Dashboard: shared journal entries view`
### Code standards
- Mobile-first: therapists use this on phones and tablets
- WCAG AA minimum: 4.5:1 contrast ratio for normal text, 3:1 for large
- Touch targets minimum 44x44px
- CSS custom properties only — never hardcode colors or spacing values
- Never use !important unless absolutely unavoidable
- Supabase calls use raw fetch — match existing api() and apiMutate() helper pattern
### Code safety
- Scan for zero-width characters (U+200B, U+200C, U+200D, U+FEFF, U+2060) before every commit. One invisible character in inline JavaScript silently breaks the page.
- Scan command (ruling D3, 2026-09-09, versioned per D20 into `scripts/scan-invisible.py` instead of a duplicated heredoc — not a `grep -P`/`python3 -c` one-liner, whose character class was mangled by shell quoting on 2026-09-08 and produced 2312 false positives; the same mangling can produce a false negative, and this check exists to catch a bug that silently breaks the page. Covers NBSP and smart quotes too, which the old five-codepoint form missed). Exit 0 and `CLEAN` on every line is a pass:
```bash
  python3 scripts/scan-invisible.py dev.html dev.css
  ```
- RLS policies are active — always test with therapist-scoped credentials, not service role.
### Schema changes
- Supabase schema changes (ALTER TABLE, CREATE POLICY, etc.) run in Supabase SQL Editor by the developer, not by Claude Code.
- If a change file includes schema changes, flag them as a prerequisite step and do not attempt to run them.
