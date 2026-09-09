# Phase 2B Report — onboarding surfaces + D22 lockout fix

## Session Log

1. **Model.** Sonnet (Claude Sonnet 5), per the session's explicit instruction.

2. **Interruptions and restarts.** One. The connection dropped mid-turn while
   the session was still gathering context for the workqueue reconciliation
   (before any branch, edit, or commit had been made in any repo). On resume,
   ground truth was re-verified directly (`git status`, `git branch`,
   `WORKQUEUE.local.md` contents in all three repos) rather than trusting
   anything from the dropped turn, confirmed nothing had actually happened
   yet, and the session proceeded from there. No work was lost or redone.

3. **Deviations from the brief.**
   - **Workqueue intake.** The task spec was pasted as command arguments,
     not staged in each repo's `WORKQUEUE.local.md` (the `/work-session`
     skill's actual intake file). `alo-dashboard` and
     `alo-client-chat-full_v1` had empty queues; `alo-supabase` had a
     different, already-completed task set (SB-03..SB-06) still marked
     pending. Per Kano's direction (asked and answered before any edits):
     the pasted spec was written into `WORKQUEUE.local.md` in all three
     repos as the active queue; SB-03..SB-06 were verified complete against
     `main` (all deliverables present: `scripts/teardown-therapist.ts`,
     `supabase/functions/open-as-client/`,
     `supabase/functions/exchange-client-token/`,
     `docs/kano-command-sheet.md`, the applied
     `20260908150000_reduce_trigger_v3.sql`) and marked DONE in place with a
     one-line completion note each, not re-run.
   - **Branch names.** Used the exact branch names the brief specified
     (`auto/phase2b-surfaces` in both app repos, `auto/phase2b-docs` in
     alo-supabase) rather than the skill's default
     `auto/<date>-<topic>` pattern, since the brief was explicit and
     consistent with the existing `auto/phase2-onboarding` /
     `auto/phase2-docs` naming from the prior session.
   - **P2B-08 + P2B-11 combined into one commit** (chat repo). P2B-11's two
     new client-menu items are the only consumer of P2B-08's four-screen
     component and directly depend on it; splitting them would have left an
     intermediate commit either shipping a dead function with no caller or a
     menu item calling a function that doesn't exist yet.
   - **Notification preflight (skill step 0) skipped**, per the brief
     ("skip notification test (proven)").
   - **No SQL applied, no deploy, no push to main** — per the brief and the
     skill's hard limits. Two schema changes are proposed, not applied; see
     DATA FOR ARCHITECT below.

4. **Checklist.** See per-task breakdown below. All 17 numbered items in the
   brief map to a COMPLETED task except two partial-completion notes (P2B-04's
   "Your test client" restore path — fully wired; P2B-05's "Open as client"
   verification — nothing existed to verify, see task detail) and one item
   (task 0, preflight) which required no branch action beyond what's
   described above since all three repos were clean at session start.

---

## Per-task breakdown

### Dashboard (`alo-dashboard`, branch `auto/phase2b-surfaces`)

| Task | Status | Commit |
|---|---|---|
| P2B-01 — D22 lockout fix | COMPLETED | `7e9326a` |
| P2B-02 — tour copy swap + install banner | COMPLETED | `d233fa9` |
| P2B-03 — therapist first-screen overlay | COMPLETED | `30e8c4d` |
| P2B-04 — help menu | COMPLETED (with one flagged gap, see below) | `9367c45` |
| P2B-05 — test client label + line | COMPLETED (with one flagged gap, see below) | `c91b0a7` |
| P2B-06 — rollout surfaces | COMPLETED (with one caveat, see below) | `70137d1` |
| P2B-07 — invite modal single-link | COMPLETED | `5d60aff` |

**P2B-01 (D22).** `isVerificationLocked()` rewritten to fail closed. Full
final source is in DATA FOR ARCHITECT below. Not deployed — the one live
account is `onboarded = true` (set 2026-09-09 15:09 UTC) so it can't be
locked out by this change when it ships with the next promotion.

**P2B-04 gap.** "Your test client" navigates to the `is_demo` client and, if
archived, restores it via the existing `restoreClient()` path (reused
directly, not reimplemented) before opening it. A real restore path exists,
so nothing was STOPped here.

**P2B-05 gap.** The task's acceptance check asked to "verify the existing
Open as client buttons remain in both locations." There is no such button
anywhere in `alo-dashboard/dev.html` — confirmed by grep, zero matches.
`alo-supabase`'s SB-04 built only the server-side token-exchange functions
(`open-as-client`, `exchange-client-token`); no dashboard-side consumer/UI
was ever wired in either app repo. This is a pre-existing gap, not something
this session broke — nothing was there to verify remains. The marker used to
identify the test client is `therapist_clients.is_demo` (boolean), already
fetched and already used elsewhere in the file.

**P2B-06 caveat.** `session_metadata` is fetched with a single
`order=created_at.desc&limit=100` across *all* of a therapist's clients (an
existing query, not something added this session). In principle a client's
most recent session could fall outside that 100-row window if other clients
generated 100+ more-recent rows in between. Not fixed — the brief forbids
adding a new fetch — noted here as a known limit on the "last active N days
ago" line's accuracy at high message volume.

### Chat (`alo-client-chat-full_v1`, branch `auto/phase2b-surfaces`)

| Task | Status | Commit |
|---|---|---|
| P2B-08 — four screens + consent | COMPLETED | `7617ffc` |
| P2B-09 — `?invite=` param prefill | COMPLETED | `e32d62b` |
| P2B-10 — contextual tips | COMPLETED | `9889275` |
| P2B-11 — client menu | COMPLETED | `7617ffc` (combined with P2B-08) |
| P2B-12 — D27 privacy modal cleanup | COMPLETED | `0260bbe` |
| P2B-13 — install banner copy | COMPLETED | `5b33d78` |

**P2B-08.** Fires only after a brand-new invite-code signup (a flag set only
in the `signUp()` success branch of the unified login/signup form, never on
a `signInWithPassword()` success), never on a returning sign-in. S1-S3
skippable (Skip jumps straight to S4); S4 is a non-skippable consent gate.
S2 includes a non-interactive mock journal entry reusing the real
journal-entry/share-toggle markup and CSS classes (no click handlers
attached, so it's inert regardless). On confirm, the overlay closes
*immediately* — the `consent_acknowledged_at` PATCH is fire-and-forget, so a
slow or failing write never strands the client; on failure it
`console.warn`s and sets a `localStorage` fallback flag
(`alo_consent_acknowledged_fallback`). The target column does not exist in
the schema yet — see DATA FOR ARCHITECT.

**P2B-11.** "How Alo works" reuses P2B-08's same component in a `'readonly'`
mode (no consent gate, no PATCH, plain Back/Done navigation, a visible close
button). "Your privacy" reuses the existing wired `#privacyLink` modal via a
`.click()` proxy rather than duplicating its copy, so the two can't drift.

### Docs (`alo-supabase`, branch `auto/phase2b-docs`)

| Task | Status | Commit |
|---|---|---|
| P2B-14 — decisions.md rulings | COMPLETED | `31b08d8` |
| P2B-15 — PRE_PILOT_TEST_PASS.md addendum | COMPLETED | `5fa879f` |

D22/D27/D28 use the numbers already assigned in the source planning
material; the ledger's high-water mark (D21) was read first and checked for
collisions (none) before writing. SB-03..SB-06 were marked DONE in
`WORKQUEUE.local.md` (local-only, not committed) rather than run, per the
verification described above.

---

## Manual test checklist

Full step-by-step manual test procedures for every surface in this phase are
in `alo-supabase/PRE_PILOT_TEST_PASS.md`, new section **"11. Onboarding
surfaces: Phase 2B copy pack + D22 lockout (addendum)"**, subsections
11.1–11.12, following the same format as the existing section 10 addendum.
That section is the authoritative checklist — it is not duplicated here to
avoid the two drifting apart.

Short version of what needs a live pass before merge:

- [ ] Dashboard: first-screen overlay (both button paths), help menu (all
      four items), test client label/line, rollout surfaces, invite modal's
      three send paths, D22 lockout screen (local dev.html only)
- [ ] Chat: four screens + consent (skip path, gate blocking, write
      confirmation), `?invite=` prefill, all four contextual tips, client
      menu, D27 privacy modal content
- [ ] `docs/decisions.md` gets one summary paragraph per section 9/10/11's
      standing requirement, written after the manual pass, not before

---

## Data for architect

### Final `isVerificationLocked()` source (alo-dashboard/dev.html)

```javascript
async function isVerificationLocked(uid) {
  // D22: fail CLOSED. Locked unless we can positively confirm the
  // therapist is either onboarded or still within their verify window.
  try {
    const rows = await api(`/rest/v1/therapists?auth_user_id=eq.${uid}&select=onboarded,verify_deadline`);
    const row = rows?.[0];
    if (!row) return true;
    if (row.onboarded === true) return false;
    if (!row.verify_deadline) return false;
    return new Date(row.verify_deadline).getTime() < Date.now();
  } catch (e) {
    console.log('Verification check failed, defaulting to LOCKED (fail closed):', e.message);
    return true;
  }
}
```

Both call sites already pass the auth user id (`therapistId = session.user.id`
/ `refreshed.user.id`) — no caller changes were needed, only the query filter
and the default-on-failure/no-row behavior.

### KANO-ONLY SQL — required before P2B-08's consent write can land

`alo-client-chat-full_v1`'s new consent gate PATCHes
`consent_acknowledged_at` on `therapist_clients`; neither that column nor
`first_conversation_at` (named in the original brief for future use) exists
yet. Additive, nullable, no default — safe to run any time, including before
this branch merges:

```sql
alter table public.therapist_clients
  add column if not exists consent_acknowledged_at timestamptz,
  add column if not exists first_conversation_at timestamptz;
```

**PRE_PILOT verification step** (also in `PRE_PILOT_TEST_PASS.md` §11.7.6):

```sql
select consent_acknowledged_at
from public.therapist_clients
where client_id = (select id from auth.users where email = '<the new signup email>');
```

Expected: a recent timestamp after completing the consent gate as a new
invite-code signup. Until the ALTER is applied, the client-side code fails
soft — `console.warn` plus a `localStorage` fallback — and never blocks the
client, so testing the UI flow doesn't require the column to exist first;
only step 11.7.6's write-verification does.

### Items STOPped or flagged as gaps (not built)

- **"Open as client" button** — does not exist anywhere in
  `alo-dashboard/dev.html`. P2B-05's acceptance check assumed it existed;
  it doesn't. Not built this session (out of scope for a copy/label task;
  building a token-exchange consumer UI is a materially larger, security-
  sensitive feature that needs its own task).
- **Session-activity accuracy limit** — P2B-06's "last active N days ago"
  line is only as accurate as the existing 100-row, all-clients
  `session_metadata` fetch; see the P2B-06 caveat above.

---

## Kano sitting list

1. **Run the KANO-ONLY ALTER TABLE** above (`consent_acknowledged_at` +
   `first_conversation_at` on `therapist_clients`) before relying on the
   consent write in production — the UI works either way, but the audit
   trail doesn't exist until this runs.
2. **Run the manual test pass** — `PRE_PILOT_TEST_PASS.md` §11 — against
   both `auto/phase2b-surfaces` branches locally, then write the one-paragraph
   summary into `docs/decisions.md` per §11.12's checklist item.
3. **Merge `auto/phase2b-surfaces`** in both `alo-dashboard` and
   `alo-client-chat-full_v1`, and `auto/phase2b-docs` in `alo-supabase`, via
   PR — Claude Code does not merge or promote.
4. **Promote D22** (`alo-dashboard`'s `isVerificationLocked()` fix) to
   `index.html` at the next dashboard promotion, under a LIVE CHANGE banner
   — it is not live yet, only in `dev.html` on this branch.
5. **"Open as client" button** — if this is wanted for the pilot, it needs
   its own task; the server-side token-exchange functions already exist
   (SB-04, `alo-supabase`), but no consumer UI does in either app.
6. **Clean up the superseded send-invite/12b planning material** in the
   claude.ai project (D28 supersedes it) — that lives outside this repo, so
   it's not something Claude Code can act on directly.
7. **Decide whether the "Open as client" gap and the session-activity
   100-row limit are worth a follow-up task** before the pilot, or are
   acceptable as-is for launch.

---

## Verification run this session

- `python3 scripts/scan-invisible.py` — CLEAN on every file changed in every
  repo (`dev.html`, `dev.css` in both app repos; `docs/decisions.md`,
  `PRE_PILOT_TEST_PASS.md` in alo-supabase).
- `node --check` on every inline `<script>` block in both apps' `dev.html`
  — clean.
- Onclick/onchange handler resolution — every custom function referenced
  from markup has a matching definition; no dangling references (checked
  specifically for the deleted `showPrivacyModal()`).
- CSS brace balance — dashboard 647/647, chat 533/533.
- Cache-buster bumps — dashboard `dev.css?v=38` (was 33), chat
  `dev.css?v=15` (was 13), matching every CSS change made this session.
- `npx deno@2 test supabase/functions/_shared/` — 18 passed, 0 failed
  (alo-supabase's existing suite, unaffected by this session's docs-only
  changes — run as a sanity check, not because this session touched
  function code).

## Push / remote verification

All three branches pushed and confirmed present on the remote via
`git ls-remote`:

- `alo-dashboard` — `auto/phase2b-surfaces` @ `5d60aff`
- `alo-client-chat-full_v1` — `auto/phase2b-surfaces` @ `7617ffc`
- `alo-supabase` — `auto/phase2b-docs` @ `5fa879f`

Nothing was pushed to `main` in any repo. No SQL was applied. No promotion,
deploy, or GitHub Pages-affecting action occurred this session.
