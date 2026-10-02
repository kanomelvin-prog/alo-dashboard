# S11 dash follow-ups report v1.0

**Repo:** `kanomelvin-prog/alo-dashboard` -- branch `auto/s11-dash-followups` off `main` @ `f9d0184`.
**Date:** 2026-10-02. **Brief:** `claude_Brief_6_Dash_Followups_v1_0.md` (Brief #6-dash v1.0, Oct 2; ledger D41 item 6).
**Files changed:** `dev.html` and this report. `dev.css`, `index.html`, `styles.css`, `manifest.json` and `CLAUDE.md` are byte-identical to `main`. Nothing was committed or pushed to `main`.

**No LIVE CHANGE in this session.** No promotion, no push to `main`, no SQL, no Edge Function, no auth or Supabase configuration change, and no request of any kind to the live Supabase project. GitHub Pages publishes `main`; this branch is not served. Merging it publishes the new `dev.html` at dashboard.alowen.ai/dev.html. Production `index.html` does not change (section 6.1).

---

## 0. Session log

1. **Model: Claude Opus 5.5 (`claude-opus-5-5`, 1M context), effort high**, as the run settings name. Claude Code, bypass permissions, unattended.
2. **Start.** The launch message: read the brief and execute it exactly as written; create `auto/s11-dash-followups` from `main` before touching any file; do not push to `main`; commit the session report to the branch. After `git fetch`, local `main` = `origin/main` = `f9d0184` (the brief's floor). The branch was created before any file was changed.
3. **Read first, in the brief's order:** `CLAUDE.md` (home and repo); `docs/reports/S8_dash_claims_fixes_report_v1_0.md` in full (its section 7.3 items 1 and 2 and its harness, section 6); `dev.html` in full (5,124 lines at `f9d0184`). Also the three vault files `CLAUDE.md` names for session start, and the S7 report's sections 3-6, because the brief asks that the S7 scenarios still pass. Ledger D41 is on disk now (`alo-supabase/docs/decisions.md`, D34-D41 merged there with Brief #6), so item 6 was read from the ledger itself. The launch message named the work, so `CLAUDE.md`'s "ask what we are working on" step did not apply.
4. **Interruptions.** The session was interrupted once by Kano while the harness scenarios were being written, after the code edits. On resuming ("continue exactly where you were interrupted ... do not redo the edits already made"), the edits were kept as they were; they were committed task by task (item 6) and the harness was finished. No context reset, no reverted commit.
5. **Deviations and judgment calls, each with its reason.**
   1. **"Real, active" includes pending invitees (C).** The brief defines the set by what it excludes: "not demo, not the therapist's own test account, not archived". A pending row (invited, not yet signed up) is none of those, and it is the very case the rollout line exists to count ("... haven't yet"). So "active" is read as "not archived", which is also what the dashboard's own copy means by "your active clients" (D-032). If Kano meant `status = active` only, it is a one-line change to the `realClients` filter (`&& c.client_id`), and the rollout line would then never count an unopened invite. Section 3.
   2. **The rollout sentence has four forms (C).** The brief asks for grammar that holds for 0, 1 and many, and gives no wording. The general form keeps the existing sentence and fixes its agreement ("1 of your 3 clients has opened Alo. 2 haven't yet."). Three edge forms replace sentences that read badly: "Your 1 client has opened Alo." / "Your 1 client hasn't opened Alo yet." (was "1 of your 1 clients have opened Alo. 0 haven't yet."), "All 3 of your clients have opened Alo." (was "... 0 haven't yet."), "None of your 3 clients have opened Alo yet." With no real clients the line stays hidden, as before. All four forms are new copy that no ruling covers word for word. Section 3 has the table.
   3. **The headline can now read "0 clients" (C).** A therapist whose only row is the test account (every new account starts that way) used to read "1 client" and now reads "All clear — no open alerts from your active clients. 0 clients." It is true and grammatical. The alternative, dropping the count at 0, is new copy, so it was not done.
   4. **The helper falls back (A).** `parseDateOnly()` handles exactly `YYYY-MM-DD`. Anything else is parsed by `new Date()` as before, so if a live column ever returns a timestamp, behaviour is unchanged rather than broken.
   5. **Copy for EHR carries no date (A).** The brief asks that "Copy P and Copy for EHR carry the right dates". Copy P carries one (the next-session line), and it is fixed. Copy for EHR, in all three formats, contains no date at all: header, note sections, and the medical-necessity placeholder. The harness checks that no date string appears in any of the three. Adding the session date to it would be new content, so it was not done (section 6.2).
   6. **One commit per task.** The edits were made in one pass before the interruption, then split into three commits by rebuilding each stage from `main`. The last stage is byte-identical to the edited file (`cmp`).
   7. **Harness rebuilt, and rebuilt S7/S8 scenarios are equivalents.** The S8 harness did not survive (the old scratchpad held empty folders). It was rebuilt to the S8 report's section 6 design. The S7 and S8 scenarios were rewritten from those reports' descriptions, not recovered, so their check counts differ from the S7 and S8 tables (section 5).
6. **Commits.** Each passed these checks first: `python3 scripts/scan-invisible.py dev.html dev.css` CLEAN; `node --check` on the inline script; `dev.css` braces 652/652; no U+200B/U+200C/U+200D/U+FEFF/U+2060, NBSP or smart quote in the diff's added lines; `index.html`, `styles.css`, `manifest.json`, `CLAUDE.md` identical to `main`.

| Commit | Task |
|---|---|
| `b21fcb6` | A -- date-only fields read as local dates |
| `7bd5644` | B -- note card label |
| `9e180d7` | C -- headline and rollout line count the same clients |
| this commit | this report |

7. **Checklist.** A COMPLETED. B COMPLETED. C COMPLETED. D COMPLETED before every commit. Verification 1-4 COMPLETED, all PASS (section 5). Report COMPLETED.

---

## 1. Summary

| Task | What changed | Commit | Harness |
|---|---|---|---|
| A | `parseDateOnly()` reads `session_date` / `next_session_date` as a local calendar day; 9 display/copy sites routed through it | `b21fcb6` | `s11_dates_tz` 23/23 in Denver and UTC; `s11_static` 7/7 |
| B | "Billing justification included" -> "Medical-necessity line included" (the only location) | `7bd5644` | `s11_note_label` 9/9 |
| C | Headline count and rollout line both count `realClients`; rollout grammar for 0, 1, many | `9e180d7` | `s11_counts_mixed` 9/9, `s11_counts_grammar` 25/25 |
| D | CLEAN before every commit | all | -- |

**Harness totals:** the branch passes **397 of 397 checks in 34 scenarios (5 S11, including the static one; 12 S8; 17 S7) under both `TZ=America/Denver` and `TZ=UTC`**, with 0 uncaught exceptions, 0 unexpected console errors, 0 network/CSP/SRI errors and 0 requests leaving the machine. `main` passes every S7 and S8 scenario in both zones (324/324 each) and fails the S11 ones: 40/73 S11 checks in Denver (364/397 overall) and 49/73 in UTC (373/397), where its dates happen to be right. Section 5.

**Read these first:**
1. **The write side still uses the UTC date** (section 6.2 item 1). The Add Note form's default date and Quick Note's saved date are `new Date().toISOString().split('T')[0]`, which is tomorrow's date in Denver after 6 pm. Display is now right, but a note written in the evening can still be saved with the wrong date. This is outside the brief (it is a write, not a display or a copy). It needs a ruling, and the fix is small.
2. **"Real clients" counts pending invitees** (deviation 1), and **the rollout line has four new sentence forms** (deviation 2). Both are judgment calls on wording and scope.
3. **Production is unchanged until the next promotion.** `index.html` still parses date-only values as UTC, still shows "Billing justification included" (index 2538), and still counts the test account in the headline.

---

## 2. A -- date-only fields

### The helper (dev.html 1759-1772, comment included)

```js
function parseDateOnly(value) {
  const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(value || '');
  if (!m) return new Date(value);
  return new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3]));
}
```

`new Date('2026-09-22')` is UTC midnight, which is 6 pm on the 21st in Denver. `new Date(2026, 8, 22)` is local midnight on the 22nd wherever the page runs. A comment above the helper says which values belong in it and which do not. The calendar has one more small helper, `noteCalendarDate(n)` (3971-3976, comment included), because its note filters fell back from `session_date` to `created_at`. The date-only value goes through `parseDateOnly()`; the timestamp fallback stays `new Date()`.

### Fields routed through it

| Field | Type, evidence | Where it is displayed or copied (branch lines) |
|---|---|---|
| `session_notes.session_date` | date-only: seeded `(NOW() - INTERVAL '21 days')::date` (trigger v3 / seed migration); written from `<input type="date">`; read back into a date input as-is | note card date (3157); Recent Session Notes header (3107); prep pack `LAST SESSION NOTE (...)` (3742); calendar note dots, month grid and 14-day strip, and both day popups (4001, 4096, 4149 via `noteCalendarDate`) |
| `session_notes.next_session_date` | date-only: written from `<input type="date">`, read back into one as-is | note card "Next session: ..." (3177); Copy P "Next session: ..." (3638) |

Six `parseDateOnly(` call sites in all, plus the definition. Static check: every argument is `<x>.session_date` or `<x>.next_session_date`, and no `new Date(` is applied to either field anywhere in the file.

### Fields deliberately not routed through it

| Field | Why not |
|---|---|
| `therapist_clients.next_session_date` | Date-only (the seed says "Type is `date`"), but this file never parses or displays it as text. It only fills `<input type="date">` with the raw string (4476) and writes it back. Nothing to fix |
| `session_metadata.created_at` | Timestamp (moment of a page load). Counts, "Last Alo", Engagement "last:", calendar Alo dots |
| `client_messages.created_at` | Timestamp. Message times, calendar |
| `homework_cards.created_at`, `completed_at`, `viewed_at` | Timestamps. "Assigned", "Completed", "seen by client", calendar |
| `crisis_events.created_at`, `acknowledged_at` | Timestamps. Alert times, "Flagged ...", prep pack "Alert on ...", the 7-day resolved window |
| `journal_entries.created_at` | Timestamp. Shared-entry dates |
| `session_notes.created_at` | Timestamp. Only the calendar's fallback for a note with no `session_date` |
| `therapist_clients.updated_at` | Not selected (not in `THERAPIST_CLIENT_COLUMNS`), so the archived list shows no date. Untouched |
| `therapists.verify_deadline` | Set as `now() + interval '10 days'` (signup spec): a timestamp. Compared with `Date.now()` in the lock check, never displayed |
| `onboarding_completed_at`, `onboarding_skipped_at` | Written with `new Date().toISOString()`, only null-checked when read |

### Before / after (harness, `TZ=America/Denver`)

| Surface | `main` | branch |
|---|---|---|
| Note card, note dated 2026-09-22 | Monday, September 21, 2026 | Tuesday, September 22, 2026 |
| Note card, note dated 2026-09-15 | Monday, September 14, 2026 | Tuesday, September 15, 2026 |
| Recent Session Notes | Monday, September 21 | Tuesday, September 22 |
| Note card next session (2026-09-29) | Next session: Mon, Sep 28 | Next session: Tue, Sep 29 |
| Copy P | Next session: Monday, September 28, 2026 | Next session: Tuesday, September 29, 2026 |
| Prep pack | LAST SESSION NOTE (Sep 21): | LAST SESSION NOTE (Sep 22): |
| Calendar, September 2026 | note dots on the 14th and 21st | note dots on the 15th and 22nd |
| Day popup for the 22nd | (nothing: the note sits on the 21st) | "Tuesday, September 22" ... Session Notes ... Individual: Family conflict; sleep |

Under `TZ=UTC` both builds render the right dates, which is why the bug was invisible to anyone testing at UTC.

**Timestamp controls** (same scenario, both zones). Three timestamps at 03:00Z, which is the evening before in Denver: the Engagement "last:" date (`session_metadata.created_at`), "seen by client" (`homework_cards.viewed_at`) and "Alert on" (`crisis_events.created_at`). The harness computes each expected date for the run's zone in Node. They show the previous day in Denver and the same day in UTC, on both builds. So timestamps still follow the viewer's zone, and the helper changed none of them.

---

## 3. B and C -- before / after

### B -- the note card label

| Where | Before | After |
|---|---|---|
| Note card meta line under a note whose box is ticked (`main` 3141 -> branch 3178); the same card renders in the In Session list and the View All drawer | "Billing justification included" | "Medical-necessity line included" |

That is every location in `dev.html`: `grep -i "billing justification"` finds nothing else (0 after). The stored field `billing_justification` and the checkbox (S8's "Add a medical-necessity line to complete") are unchanged. `index.html` 2538 still has the old label (section 6.1).

### C -- the two counts

**Diagnosis.** Both lines are built in `loadDashboard()` from `allClients`, which is every `therapist_clients` row the load returns. The load asks for `status=in.(active,pending)`, so archived rows (an archived demo client included) never reach either line.

| Line | Before (`main`) | After (branch) |
|---|---|---|
| Headline, "N clients · ..." (`main` 2028 -> 2067) | `allClients.length`: active + pending, **including the therapist's own test account** (`is_demo`) | `realClients.length` |
| Rollout, "... opened Alo" (`main` 2047-2051 -> 2086-2088) | `allClients.filter(c => !c.is_demo)`: active + pending, test account excluded | `realClients`, the same constant, via `rolloutSentence()` |

`realClients = allClients.filter(c => !c.is_demo)` is defined once (2061), above both lines, with a comment saying why. So the two lines cannot disagree again: a real client is a linked person who is not the test account and not archived. Pending invitees count (deviation 1).

The other headline counts (alerts, unread messages, active homework) still include the test account's. That is deliberate, because an alert must never drop out of a count, and the brief scopes C to the two client counts.

**The rollout sentence** (`rolloutSentence(opened, total)`, 1925-1936):

| Case | `main` | branch |
|---|---|---|
| 1 client, opened | 1 of your 1 clients have opened Alo. 0 haven't yet. | Your 1 client has opened Alo. |
| 1 client, not opened | 0 of your 1 clients have opened Alo. 1 haven't yet. | Your 1 client hasn't opened Alo yet. |
| 3, 1 opened | 1 of your 3 clients have opened Alo. 2 haven't yet. | 1 of your 3 clients has opened Alo. 2 haven't yet. |
| 3, 2 opened | 2 of your 3 clients have opened Alo. 1 haven't yet. | 2 of your 3 clients have opened Alo. 1 hasn't yet. |
| 3, all opened | 3 of your 3 clients have opened Alo. 0 haven't yet. | All 3 of your clients have opened Alo. |
| 3, none opened | 0 of your 3 clients have opened Alo. 3 haven't yet. | None of your 3 clients have opened Alo yet. |
| 0 real clients (test account only) | hidden; headline "1 client" | hidden; headline "0 clients" |

**The brief's two seeds** (harness, both builds):

| Seed | `main` | branch |
|---|---|---|
| 1 real active client + test account + archived demo client | "2 clients · ..." / "1 of your 1 clients have opened Alo. 0 haven't yet." | "1 client · ..." / "Your 1 client has opened Alo." |
| 3 real clients (2 opened) + test account + archived demo | "4 clients · ..." / "2 of your 3 clients have opened Alo. 1 haven't yet." | "3 clients · ..." / "2 of your 3 clients have opened Alo. 1 hasn't yet." |

---

## 4. Verification -- the brief's section 3

| # | Item | Result |
|---|---|---|
| 1 | A: the TZ test; Copy P and Copy for EHR carry the right dates; no timestamp routed through the helper | PASS. `s11_dates_tz` 23/23 under both `TZ=America/Denver` and `TZ=UTC`. Chrome runs with `TZ` set and the CDP time-zone override, and the first check asserts the page's own `Intl` zone. The note dated 2026-09-22 renders "Sep 22" (and "Tuesday, September 22, 2026") in both. Copy P carries the right next-session date. Copy for EHR carries no date in any format (deviation 5). Static (`s11_static` 7/7): 6 call sites, all date-only fields; no `new Date(` on either field; no `*_at` argument. Dynamic: the three timestamp controls in section 2. `main` in Denver: 14/23 |
| 2 | B: old label at 0; new label at every location | PASS. Static: 0 and 1. `s11_note_label` 9/9: on the ticked card in the list and in the View All drawer, absent from the unticked card, and the old label nowhere in the rendered page or its inputs. `main`: 6/9 |
| 3 | C: 1 real + test account + archived demo -> both say 1; 3 real -> both say 3, grammar correct | PASS. `s11_counts_mixed` 9/9 (the request asks for `status=in.(active,pending)`; the archived demo is not listed and the test account still is). `s11_counts_grammar` 25/25: seven seeds, from 3 real clients with 2, 1, 3 and 0 opened, down to 1 real, 1 real + 1 pending, and the test account alone. In each, the headline and the rollout line state the same number. `main`: 7/9 and 12/25 |
| 4 | S7 and S8 scenarios still pass; `node --check`; `scan-invisible.py` CLEAN; zero invisible characters in the diff | PASS. All 17 S7 and 12 S8 scenarios pass on the branch in both zones (section 5). There is one inline script (dev.html 809-5166); `node --check` OK. `dev.html: CLEAN`, `dev.css: CLEAN`, exit 0, before every commit. The diff's added lines hold no U+200B/U+200C/U+200D/U+FEFF/U+2060, NBSP or smart quote |

---

## 5. Harness results

**Method** (the S8 design, rebuilt). Headless Google Chrome 154.0.8037.97 is driven over the DevTools protocol by a zero-dependency Node 25.8.0 script (built-in `http`, `child_process`, `WebSocket`). A local static server serves two builds straight from git: the branch (`HEAD` = `9e180d7`) and `main` (`f9d0184`), so only committed content is tested. Every scenario runs against both builds, in its own freshly launched browser, under `TZ=America/Denver` and again under `TZ=UTC`. Nothing in either page is patched, with one exception: `password_reset_fallback` forces the Set New Password modal to throw, which is the only way to reach its catch. Before any page script, two scripts are injected:
- **a Supabase emulation** that answers `window.fetch` for the project's origin. It covers the GoTrue password and refresh grants (a refresh token can be reused, as GoTrue's reuse interval allows), the recovery PUT, and PostgREST over an in-memory database: eq/neq/in/is/gt/lt filters, `select` projection, `order`, `limit`, `Prefer`, and upsert with `on_conflict`. It returns PostgREST's 400 for an unknown column and enforces the `sensitivity_level` check constraint. It has no row-level security, on purpose. Canary values sit in every column the dashboard should never receive: the `session_metadata` nonce, mode and user agent; crisis `action_notes` and `details`; the legacy private columns on `therapist_clients`. Rules can inject zero-row mutations, 5xx answers, expired tokens, rejected or held refreshes, extra (leaked) rows, and a server that ignores `select=`;
- **recorders** for the clipboard, `alert` / `confirm` / `prompt` and toasts, so copied text can be read back without touching the machine's clipboard.

**Hermetic by construction:** every request the page makes is intercepted at the network layer and failed, unless it goes to the harness's own server or is one of the two pinned driver.js URLs. Those two are answered from local copies whose SHA-512 matches the page's SRI pins. `real_cdn_load` lets them go to cdnjs. Requests that tried to leave the machine: 0. Calls to the live Supabase API: 0. Real credentials used: none.

Every scenario also carries five hygiene checks: it ran to the end; no uncaught exception; no console error (a scenario that deliberately makes the page fail may allow the page's own `console.error` lines, and each one names which); no network/CSP/SRI error; no request left the machine.

| Scenario | Group | Branch, Denver | Branch, UTC | `main`, Denver | `main`, UTC |
|---|---|---|---|---|---|
| `s11_static` | S11 | PASS 7/7 | PASS 7/7 | FAIL 1/7 | FAIL 1/7 |
| `s11_dates_tz` | S11 | PASS 23/23 | PASS 23/23 | FAIL 14/23 | PASS 23/23 |
| `s11_note_label` | S11 | PASS 9/9 | PASS 9/9 | FAIL 6/9 | FAIL 6/9 |
| `s11_counts_mixed` | S11 | PASS 9/9 | PASS 9/9 | FAIL 7/9 | FAIL 7/9 |
| `s11_counts_grammar` | S11 | PASS 25/25 | PASS 25/25 | FAIL 12/25 | FAIL 12/25 |
| `a24_session_columns` | S8 | PASS 13/13 | PASS 13/13 | PASS 13/13 | PASS 13/13 |
| `prep_pack_no_themes` | S8 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `ehr_copy_placeholder` | S8 | PASS 19/19 | PASS 19/19 | PASS 19/19 | PASS 19/19 |
| `homework_no_sensitivity` | S8 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `no_default_settings_modal` | S8 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `password_reset_modal` | S8 | PASS 11/11 | PASS 11/11 | PASS 11/11 | PASS 11/11 |
| `password_reset_fallback` | S8 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `copy_login_help` | S8 | PASS 13/13 | PASS 13/13 | PASS 13/13 | PASS 13/13 |
| `copy_home_client` | S8 | PASS 14/14 | PASS 14/14 | PASS 14/14 | PASS 14/14 |
| `copy_zero_clients` | S8 | PASS 8/8 | PASS 8/8 | PASS 8/8 | PASS 8/8 |
| `console_clean` | S8 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `real_cdn_load` | S8 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `home_scope` | S7 | PASS 13/13 | PASS 13/13 | PASS 13/13 | PASS 13/13 |
| `cap_displacement` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `no_crisis_text` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `signout_signin` | S7 | PASS 14/14 | PASS 14/14 | PASS 14/14 | PASS 14/14 |
| `forced_logout` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `late_refresh_race` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `zero_row_mutations` | S7 | PASS 20/20 | PASS 20/20 | PASS 20/20 | PASS 20/20 |
| `client_edit_private` | S7 | PASS 14/14 | PASS 14/14 | PASS 14/14 | PASS 14/14 |
| `private_unknown` | S7 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `invite_db_code` | S7 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `guards` | S7 | PASS 11/11 | PASS 11/11 | PASS 11/11 | PASS 11/11 |
| `load_failure` | S7 | PASS 11/11 | PASS 11/11 | PASS 11/11 | PASS 11/11 |
| `prep_pack` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |
| `copy_static` | S7 | PASS 14/14 | PASS 14/14 | PASS 14/14 | PASS 14/14 |
| `zero_clients` | S7 | PASS 10/10 | PASS 10/10 | PASS 10/10 | PASS 10/10 |
| `tour_signout` | S7 | PASS 8/8 | PASS 8/8 | PASS 8/8 | PASS 8/8 |
| `happy_path` | S7 | PASS 9/9 | PASS 9/9 | PASS 9/9 | PASS 9/9 |

**Totals:** branch under `America/Denver`: 397/397; main under `America/Denver`: 364/397; branch under `UTC`: 397/397; main under `UTC`: 373/397. On `main` the only failing scenarios are the S11 ones; every S7 and S8 scenario passes on both builds in both zones.

**Harness problems found and fixed while building it** (none of them in the app):
1. `password_reset_fallback` read its alert log from the page after the page had reloaded itself ("Password updated!" navigates). Alerts are now also kept in `sessionStorage`.
2. `late_refresh_race`: the first stub deleted a refresh token on first use. That is stricter than GoTrue, so the client page's parallel 401s failed while A was still signed in, and the "Session expired" toast came from A's own session. The stub now allows reuse, and the check reads only the toasts after B signs in.
3. In the first full run, the Denver browser process stopped answering CDP partway through the `main` build (`Target.createBrowserContext` and `Runtime.enable` timed out before the scenarios' first step). It was the browser, not a page. Every CDP call now has a 20-second timeout, every scenario a 90-second one, and each scenario gets a freshly launched browser. The full run above is the run made after these changes.

**Phone width** (`CLAUDE.md`: mobile-first). At 375x812 (Denver), two surfaces were screenshotted and looked at. Home: headline "All clear — no open alerts from your active clients. 3 clients · 2 active homework." over "1 of your 3 clients has opened Alo. 2 haven't yet."; no horizontal scroll. The note card on In Session: "Next session: Tue, Sep 29" then "Medical-necessity line included", one line each. That client page's document is 397 px wide at a 375 px viewport, on `main` as well as on the branch (measured on both), so it predates this branch (section 6.2). No CSS change was needed.

**Where it is:** the session scratchpad, not committed (the S7 and S8 precedent). The files are `harness/stub.js`, `lib.mjs`, `scenarios.mjs`, `run.mjs`, `shots.mjs` and `inline-check.py`, plus `precommit.sh`; about 1,370 lines. It needs only Node and Chrome. It can go on this branch under `scripts/harness/` if wanted.

---

## 6. Sequencing, needs a ruling, found along the way

### 6.1 Sequencing

1. **Production.** `index.html` was out of scope and is unchanged. It still parses `session_date` and `next_session_date` as UTC, still shows "Billing justification included" (2538), and still counts the test account in the headline. These fixes reach therapists when `dev.html` is next promoted. That promotion already carries S7 and S8 (and v1.6 makes production's whole-row `session_metadata` reads fail, per D41 item 4a).

### 6.2 Needs a ruling / found along the way (pre-existing, not changed)

1. **The UTC date on the write side.** `showNoteForm()` pre-fills Session Date with `new Date().toISOString().split('T')[0]` (3241), and `saveQuickNote()` saves `session_date` the same way (3598). That is the UTC calendar date: from 6 pm MDT (5 pm MST) until midnight, a Denver therapist is offered tomorrow's date, and a Quick Note is saved as tomorrow's note. The Add Note form shows the date, so a therapist can catch it. Quick Note does not show it. Display (this brief) is now right, so the stored date is the remaining error, and it is EHR-bound once copied. The fix is a local `YYYY-MM-DD` formatter used in those two places. It was not made because the brief's A is about display and copy, and section 2 rules out "anything else". Recommended for the next dashboard brief, or as an add-on to this one if Kano rules it in.
2. **Copy for EHR has no date.** All three formats carry no session date (deviation 5). That is fine if the EHR's own note has the date. If clinicians expect the copied text to carry it, that is a content change for a ruling.
3. **The rollout line is only as good as `lastActivity`.** The home load reads the newest 100 `session_metadata` rows across all clients (`limit=100`). D41 item 6 notes that every page load writes a row, so a busy client can fill that window, and a quieter client whose activity is older than those 100 rows is counted as "hasn't opened Alo yet". This predates the branch; the line now counts the right people, but "opened" can undercount. It belongs with the "sessions this week counts page loads" item in D41 item 6.
4. **Client Pulse lists the test account** by its raw display name ("Test Account (You)", where the sidebar says "Your Test Client"). It is a list, not a count, so the two counts still agree. It is noted because it sits right under a headline that no longer counts it.
5. **The client page is 22 px too wide at 375 px** (In Session tab; `main` too). Not investigated further: no CSS change was in scope.
6. **The greeting uses the first word of the display name**, so "Dr. Ada Park" reads "Good morning, Dr." (`showApp()`). Cosmetic, pre-existing.

### 6.3 Declined, as the brief says

The alerts query for archived clients (Gate B). Anything in the client app. `index.html`. `dev.css`, since no fix needed it.

---

## 7. Not verifiable here

1. **The live types of `session_notes.session_date` and `next_session_date`.** No migration in any repo creates `session_notes`. The evidence is the seed's `::date` cast for `session_date`, and the fact that both values fill `<input type="date">` as returned (which only works for `YYYY-MM-DD`). If either were a timestamp column, the helper's fallback keeps the old behaviour.
2. **Real devices and real time zones.** Headless Chrome with a time-zone override is not an iPad in Denver. Section 8 has the check.

---

## 8. Kano's five-minute check on dashboard.alowen.ai/dev.html after merging (test account)

1. **Home.** The headline's client count leaves out the test account, and the line under it gives the same number. With only the test account, you see "0 clients" and no rollout line.
2. **Open a client with a session note** (the test client's seeded notes will do). The note card's date matches the date in Edit (the date input). Before this branch it was a day earlier in the evening hours and anywhere in the Americas.
3. **Copy P** on a note with a next-session date: the date matches the one in Edit.
4. **Copy Prep Pack:** `LAST SESSION NOTE (...)` shows the note's own date.
5. **A note with the medical-necessity box ticked** reads "Medical-necessity line included" under it.
6. **Calendar (either tab):** the grey note dot sits on the note's own date.

---

*S11 dash follow-ups v1.0 -- 2026-10-02. Branch `auto/s11-dash-followups`, four commits with this one. `dev.html` and this report. Nothing on `main`, nothing promoted, no live call.*
