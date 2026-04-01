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
## File Structure
- `index.html` — production dashboard (~3,516 lines)
- `dev.html` — development copy (edit this, promote to index.html when ready)
- `styles.css` — production stylesheet (~3,097 lines), versioned at ?v=29
- `dev.css` — development stylesheet (edit this, promote to styles.css when ready)
- `CLAUDE.md` — this file
## Session Start — Read These First
Before doing any work, read these files from the Obsidian vault to orient yourself:
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
## Code Standards — Non-Negotiable
- Mobile-first: therapists use this on phones and tablets
- WCAG AA minimum: 4.5:1 contrast ratio for normal text, 3:1 for large
- Touch targets minimum 44x44px
- CSS custom properties only — never hardcode colors or spacing values
- Never use !important unless absolutely unavoidable
- Always edit dev.html and dev.css first, never index.html or styles.css directly
- Increment ?v=N on styles.css reference after every CSS change
- Supabase calls use raw fetch — no SDK. Match existing api() and apiMutate() helper pattern
## Critical Warnings
- INVISIBLE CHARACTER BUG: AI-generated code can contain U+200B zero-width spaces
  that silently break JavaScript. Verify page loads after every large code block.
  Check with: grep -rP '[\x{200B}]' . --include="*.html" --include="*.js"
- NEVER expose conversation transcripts in any dashboard view
- NEVER allow therapists to see raw chat content — Zero-Knowledge is non-negotiable
- RLS policies are active — always test with therapist-scoped credentials, not service role
- index.html and dev.html are currently identical — always work in dev files first
## Git Workflow
- Always work on a feature branch, never directly on main
- Commit message format: "dashboard: [what changed]"
- Push to main triggers GitHub Pages deploy (allow 1-2 min to propagate)
- Verify live URL after every deploy
## What Claude Code Must Never Do
- Do not modify Supabase schema — use Supabase Studio
- Do not touch auth configuration without explicit approval and full test plan
- Do not add analytics, tracking, or third-party scripts without approval
- Do not add any feature that shows therapists conversation content
- Do not add engagement hooks of any kind
- Do not switch from raw fetch to Supabase SDK — keep existing pattern
