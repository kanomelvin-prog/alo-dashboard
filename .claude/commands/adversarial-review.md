---
description: Adversarial code review of Alo — safety-critical therapeutic AI companion. Hunts silent failures, safety-check bypasses, user-disclosure leaks, and Supabase security gaps across the seven risk areas.
---

# /adversarial-review v2 — Alo Adversarial Code Review

Assume the code changed in this session contains flaws. Do not be neutral — actively search for attack surfaces and failure modes. Do not soften findings. Do not pad with praise. When uncertain whether something is a real risk, flag it as NEEDS REVIEW — never silently approve uncertain safety logic.

Posture: **a missed crisis detection is worse than a hundred false escalations. A silently swallowed error is worse than a crash.**

## Stack Context (current state — verify, don't assume)

- **Botpress Cloud** — orchestration. One linear flow per turn: Start → Init_State → Safety_Check → Check_And_Route → Call_Claude → Validate_And_Send.
- **Safety_Check** is a deterministic keyword/pattern scan (no LLM call), three tiers: A (immediate intent), B (passive ideation), metaphor handling. It runs before the Claude call on every turn. There is NO Haiku classifier in the message path.
- **Claude API** — Sonnet for conversation (Botpress Anthropic card); Haiku for memory summarization and forget-request matching (execute-card axios, Botpress env secrets). All Anthropic calls run inside Botpress Cloud; the browser never holds the key.
- **Supabase** — data, auth, RLS. Client writes conversation data directly with the public anon key under RLS. `crisis_events` is written from Botpress with the service-role key.
- **Frontend** — vanilla HTML/CSS/JS on GitHub Pages (alowen.ai, dashboard.alowen.ai). No build step, no npm, no bundle. Source is output.

## Usage

`/adversarial-review [path or area]` — defaults to the whole repo. Optionally scope, e.g. `/adversarial-review safety_check` or `/adversarial-review supabase`.

## Kano's Seven Risk Areas (tag every finding with one, or OTHER)

1. **Authentication** — can anything be accessed without proper auth?
2. **Data loss** — any path where user data (memory, journal) could be silently dropped?
3. **Race conditions** — async flows where two things happening simultaneously breaks state?
4. **Rollbacks** — if a Supabase write fails mid-operation, is the state recoverable?
5. **Degraded dependencies** — what happens if Botpress, Supabase, or the Claude API is slow or down?
6. **Version skew** — any assumptions about API versions or schema that could break on update?
7. **Observability gaps** — if something fails in production, would you know? Is it logged?

A cluster of OTHER findings means this list needs revising — say so in the report.

## Review Passes (run all, in order)

### Pass 1 — Silent Failure Hunt (zero tolerance)

- Empty catch blocks, `catch {}`, errors converted to `null`/`[]`/defaults
- `.catch(() => fallback)` where the fallback hides that an operation failed
- **Fail-safe vs fail-silent on Safety_Check:** `workflow.safetyFlag` initializes to `'none'`. If the execute card throws mid-scan, does the turn proceed as if no crisis was detected? Any exception path that lands on the lowest safety outcome is a CRITICAL finding.
- Timeouts: what happens when the Claude API hangs inside Botpress? Is there a timeout at all? What does the user see?
- Malformed Haiku output (bad JSON from summary or forget-match calls) — must produce an explicit error path, not a silent skip or a wrong deletion.
- Logs without enough context to reconstruct a failure after the fact
- Lost stack traces, generic rethrows, missing async/await error handling

### Pass 2 — Safety-Check Integrity

- Can any input path reach Call_Claude WITHOUT passing Safety_Check first? (Retries, state carried across turns, any node transition that skips the scan.)
- Tier logic: verify no ordering bug lets a Tier A phrase resolve as Tier B or metaphor; verify negation handling can't suppress genuine passive ideation.
- False-negative bias check: enumerate conditions under which crisis-level language slips through the keyword tiers. Adversarially construct three example messages that would slip through, if possible. (Known: "I don't want to be here anymore" — confirm still open or fixed.)
- **Prompt injection — real surfaces:** user text is spliced into the Haiku forget-matching prompt and summary prompts, and into the Sonnet context via buildContextMessages(). Check delimiter escape, instruction smuggling (e.g., a user message that convinces the forget-matcher to return arbitrary IDs → wrong memories deleted), and whether user content is clearly fenced as data.
- Are crisis detections logged with enough detail to audit post-pilot (tier, category, timestamp, session context)? Note: non-crisis mode selection happens inside the Sonnet prompt and is not logged — flag any change that makes this worse.

### Pass 3 — User Disclosure Protection

Users tell Alo sensitive things. Rule: user content may live ONLY in RLS-protected Supabase tables (that storage is the product). It may NOT appear in telemetry, consoles, or third-party systems.

- No user message content in `console.log` / `console.error` output (these land in Botpress Cloud logs and browser consoles)
- No user content or identifiers in URL parameters or query strings
- No user content in localStorage/sessionStorage beyond explicitly designed exceptions (the C19 message-loss backup is designed; auth tokens in localStorage are tracked as open finding D9 — don't re-report, but flag any expansion)
- No Supabase `service_role` key anywhere client-side — grep both repos' source (there is no build output)
- Claude API key never reachable from the client; all LLM calls stay inside Botpress
- RLS enabled on EVERY table containing user data; cross-user isolation: can user A read user B's rows under any policy combination? Verify with `pg_constraint`/`pg_policies`, never `information_schema`.
- Data retention: any table accumulating raw user content without a stated reason?

### Pass 4 — Standard Security Sweep

- Hardcoded secrets in source, config, or git history (git history check is fast on these repos — run it)
- Unparameterized PostgREST filters or string-built queries from user input
- `innerHTML = userInput` or equivalent DOM sinks in the vanilla JS frontends
- **Rate limiting / cost abuse:** what stops one guest (or a bot) from burning the Claude API budget through the public chat? Missing throttle on the LLM path is HIGH — cost risk and abuse vector.
- Auth checks on every Supabase mutation path the dashboards and chat perform

### Pass 5 — Botpress & Integration Seams

- What happens in Botpress when the Anthropic call errors — retry storms, duplicate sends, dropped messages?
- Any node transition or error handler that could emit a response without Validate_And_Send post-processing?
- Webchat SDK init failure states (`window.botpress.init()` — C5 lineage): explicit failure UI or silent forever-spinner?

## Output Contract

**The report must OPEN with a `## Session Log` section**, before the verdict.
Standing requirement adopted 2026-09-08; canonical wording in `alo-supabase`,
`docs/decisions.md` → "Report conventions (standing)". It contains, in order:
model used; interruptions and restarts ("None" is a valid entry, silence is
not); deviations from the brief, each with a one-line reason; and a checklist
of every pass in this command marked COMPLETED or SKIPPED, with a reason on
every skip. A skipped pass that is not declared reads as a clean pass, which is
the single most dangerous thing an adversarial review can get wrong.

Then:

```
## Adversarial Review: [scope] — [date]

### Verdict: SAFE TO PILOT / NEEDS FIXES / BLOCK — USER SAFETY RISK

### Findings (severity-ordered)
1. [CRITICAL|HIGH|MEDIUM|LOW] [Risk area #N or OTHER] [Pass #]
   - Location: file:line
   - Issue: what is wrong
   - Impact: concrete failure scenario for a real user
   - Fix: exact change required
   - Safe to auto-fix: yes/no

### Silent-failure count: N (target: 0)
### Safety-check bypass paths found: N (target: 0)
### Disclosure-leak vectors found: N (target: 0)

### Checks run
- Commands executed and their raw pass/fail
- Areas NOT covered in this run (be explicit)

### Remediation order
Numbered list, highest user-safety impact first.
```

Save report to `notes/adversarial-reviews/[date]-[brief-description].md`.

## Rules

- Report findings only — do not implement fixes.
- Separate scanner/command facts from judgment. Never invent a finding a tool did not support or you did not verify in code.
- One CRITICAL finding = verdict cannot be SAFE TO PILOT.
- User-content disclosure into telemetry/console/third-party is always CRITICAL, however small.
- Never approve code that silently catches Safety_Check or crisis-logging errors.
- False positives to skip: `.env.example` placeholders, clearly marked test/demo fixtures (seeded demo/test-client data, whether trigger- or Edge-Function-created), the Supabase anon key client-side (public by design — but re-verify RLS in Pass 3 every run), findings already tracked open in `A7_Remediation_Tracker_v1_0.md` (reference them, don't re-report).

## Post-Migration Appendix (DO NOT RUN until Edge Functions Phase 4)

When orchestration moves to Supabase Edge Functions and multi-model routing lands, activate these:
- Classifier chain: classifier API failure must fail SAFE (escalate/hold), never default to lowest tier
- Malformed classifier output (bad JSON, unknown intention label) → explicit error path, no silent default intention
- CORS configuration on the new endpoints
- Rate limiting on the new LLM-calling endpoints
- Retry/backoff behavior between classifier and responder calls

<!-- v2, 2026-09-03. Merges the original seven-area command (committed 2026)
     with adapted passes from everything-claude-code (Affaan Mustafa, MIT
     license): security-reviewer, silent-failure-hunter, healthcare-reviewer,
     database-reviewer. github.com/affaan-m/everything-claude-code
     Adaptation decisions logged in alo-review-briefing-2026-0903.md -->
