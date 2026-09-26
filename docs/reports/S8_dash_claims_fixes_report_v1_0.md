# S8 dash claims fixes report v1.0

**Repo:** `kanomelvin-prog/alo-dashboard` -- branch `auto/s8-dash-claims` off `main` @ `a782b14`.
**Date:** 2026-09-26. **Brief:** `claude_Brief_4_Dash_Claims_Fixes_v1_0.md` (Brief #4-dash v1.0, Sep 21; ruling D40). **Findings source:** Claims Inventory v1.0 (Sep 21), sections 2 and 4. The inventory lives outside every repo and stays there; it is quoted here only where a before/after needs its verbatim text.
**Files changed:** `dev.html` and this report. `dev.css`, `index.html`, `styles.css`, `manifest.json` and `CLAUDE.md` are byte-identical to `main`. Nothing was committed or pushed to `main`.

**No LIVE CHANGE in this session.** No promotion, no push to `main`, no SQL, no Edge Function, no auth or Supabase configuration change, and no request of any kind to the live Supabase project. GitHub Pages publishes `main`; this branch is not served. Merging it publishes the new `dev.html` at dashboard.alowen.ai/dev.html; production `index.html` does not change (section 7.1).

---

## 0. Session log

1. **Model: Claude Opus 5.5 (`claude-opus-5-5`, 1M context) -- not the Fable 5.1 the brief's run settings name.** The session was launched on Opus 5.5; the brief's `/model` step happens at launch and is not something a running session can do for itself. Recorded here rather than absorbed. **Effort: max.** Claude Code, bypass permissions, unattended after launch.
2. **Start.** The launch message: read the brief and execute it exactly as written; create `auto/s8-dash-claims` from `main` before touching any file; do not push to `main`; commit the session report to the branch. After `git fetch`, local `main` = `origin/main` = `a782b14` (the brief's floor). The branch was created before any file was changed.
3. **Read first, in the brief's order:** `CLAUDE.md` (home and repo); `docs/reports/S7_dash_fixes_session_report_v1_0.md` in full; the Claims Inventory's sections 0-4 in full, plus the section 5 row of every D-entry the brief names; `dev.html` in full (5,174 lines at `a782b14`, every line). Also the three vault files `CLAUDE.md` names for session start. From them: the roadmap (v1.3, March 3) lists dashboard v3.6 and "Pilot prep -> Dupre pilot"; it is a March snapshot, and the priority in flight is the brief's own (Gate A claims-freeze, spine item 8.3, with A-24). Never build: any therapist view of transcripts or raw chat, engagement hooks of any kind, sharing without a client action. The launch message named the work, so `CLAUDE.md`'s "ask what we are working on" step did not apply.
4. **Interruptions and restarts.** None: no context reset, no reverted commit. Two slips were caught and fixed before their commits: a stray blank line in B3's write, and a B4 comment that quoted the removed text (verification 3's grep would have caught it).
5. **Deviations and judgment calls, each with its reason.**
   1. **Model** -- item 1.
   2. **One column list for both reads (B1).** Derived from the code (section 2): the home load reads `client_id` and `created_at`; the client-page load reads `created_at` only. Both reads use the one constant `client_id,created_at`, the way `CRISIS_EVENT_COLUMNS` serves both crisis reads -- the brief points at that rule, and its expected list is this one. On the client-page read, `client_id` is the value the request already filters on, so it adds nothing the browser does not already hold.
   3. **Seven S7 comments reworded (B1).** Verification 1 asks for "no `select=*` anywhere in the file". At `a782b14` the only occurrences were seven S7 comments that forbid it; they now say "a wildcard select". Same meaning; the grep count is 0.
   4. **Em dashes, verbatim.** Three ruled strings put an em dash where the text around them uses `--`: D-065 (the "What clients see" constant writes dashes as `--`), D-048 (Copy for EHR; its other lines are ASCII) and D-008 (a form hint). Section 1 of the brief says to use the ruled wording verbatim, so they are verbatim. For D-065 this is also what the chat app will show: that constant mirrors the chat app's onboarding text, the chat app uses em dashes, and the client brief's C-044 carries the same sentence with the same dash. A comment on the constant records the exception. Changing any of the three to `--` is a two-character edit.
   5. **S1 placement.** The brief names the blocks, not a position in them. The sentence is appended to the end of the login disclaimer, and to the end of both "What you'll see" paragraphs (How Alo works, first-screen overlay), right after "A safety alert if Alo ever detects one." Nothing else in those sentences changed.
   6. **D-032 at 2085.** The ruled sentence replaces the headline's "All clear. " and the counts that followed it stay: "All clear — no open alerts from your active clients. 4 clients · 2 active homework." An S7 comment that quoted the old headline was updated to match (verification 3's grep).
   7. **B3.** (a) "The write path's default" is read as the value an untouched form always sent: `submitHomework()` sends `sensitivity_level: 'low'`. (b) The More options button said "(type, sensitivity, rationale)". With the control gone, that label would have become a false claim, so it reads "(type, rationale)" in both places it is set (the markup and `toggleHwForm()`).
   8. **B5.** The modal's Escape wiring, `wireEscToStaticModal('defaultSettingsModal')`, went with the modal.
   9. **B6 wording.** Each surface keeps its own sentence form, with 6 changed to 12. The placeholder is exactly "Min. 12 characters" (sign-up's own placeholder). The validation reads "Password must be at least 12 characters" (sign-up's own message). The prompt asks for "(min 12 characters)". The prompt() fallback used to drop a short entry silently; verification 8 asks for a refusal "with the 12-character message", so it now shows that message in an alert. That alert is the one behaviour B6 adds.
   10. **Harness rebuilt, not committed.** The S7 harness scripts did not survive; that session's scratchpad held only empty folders. The harness was rebuilt from the S7 report's section 6 description and lives in this session's scratchpad, as S7's did (section 6).
6. **Commits.** Each passed these checks first: `python3 scripts/scan-invisible.py dev.html dev.css` CLEAN; `node --check` on the inline script; `dev.css` braces 652/652; a zero-invisible-character scan of the diff's added lines; and a byte-identity check of `index.html`, `styles.css`, `manifest.json` and `CLAUDE.md` against `main`.

| Commit | Task |
|---|---|
| `289ffed` | B4 (D-052) -- no themes line in the prep pack |
| `b14ea9b` | B1 (A-24) -- explicit `session_metadata` columns |
| `07fd985` | B2 (D-049) -- medical-necessity placeholder |
| `afa0400` | B3 (D-007) -- Sensitivity control removed |
| `47277e5` | B5 (D-019, D-020) -- unreachable Default Client Settings modal removed |
| `398b1da` | B6 (D-025, R-B-22) -- one 12-character password policy |
| `dd111fd` | Section 3 -- ruled copy |
| this commit | this report |

   B4 went first: B1's column list is only right once `primary_mode` is no longer read, so every commit is coherent on its own.

7. **Checklist.** B1 COMPLETED. B2 COMPLETED. B3 COMPLETED. B4 COMPLETED. B5 COMPLETED. B6 COMPLETED. B7 COMPLETED before every commit. Section 3: every "new text" row applied at every listed location, S1 added in three places, and the four "keep" rows (D-024, D-015 at 724, D-029, D-068) unchanged. Verification 1-10 COMPLETED, all PASS (section 5). Report COMPLETED.

---

## 1. Summary

| Task | Finding(s) | What changed | Commit | Harness |
|---|---|---|---|---|
| B1 | A-24 | Both `session_metadata` reads send `select=client_id,created_at` | `b14ea9b` | `a24_session_columns` 24/24 |
| B2 | D-049 | "Medical necessity: [add your justification]" replaces the canned sentence in Copy for EHR (SOAP, DAP, narrative) and Copy P; checkbox relabelled in both note forms | `07fd985` | `ehr_copy_placeholder` 33/33 |
| B3 | D-007 | Sensitivity control and tooltip removed; the write sends `'low'` | `afa0400` | `homework_no_sensitivity` 18/18 |
| B4 | D-052 | Themes line and its `primary_mode` read removed from `generateAddendum()` | `289ffed` | `prep_pack_no_themes` 15/15 |
| B5 | D-019, D-020 (R-B-34) | Modal markup, both functions, the localStorage key and the Escape wiring removed | `47277e5` | `no_default_settings_modal` 12/12 |
| B6 | D-025, R-B-22 | The reset path says and enforces 12 in all three places | `398b1da` | `password_reset_modal` 12/12, `password_reset_fallback` 10/10 |
| Copy | D-001, D-002, S1, D-021, D-058, D-065, D-017, D-032, D-046, D-010, D-030, D-048, D-008 | Ruled wording, verbatim, at every listed location | `dd111fd` | `copy_login_help` 15/15, `copy_home_client` 15/15, `copy_zero_clients` 7/7 |
| B7 | hygiene | CLEAN before every commit | all | -- |

**Harness totals:** the branch passes **175 of 175 checks in 12 scenarios**, with 0 uncaught exceptions, 0 console errors, 0 network/CSP/SRI errors and 0 requests leaving the machine. `main` passes 114 of 175: it fails all 10 scenarios that encode a brief item and passes the 2 controls (`console_clean`, `real_cdn_load`). Section 6.

**Read these first** (section 7):
1. **Production is unchanged until the next promotion.** `index.html` still reads `session_metadata` with no `select=` (index 1562, 2225), and still carries the canned medical-necessity sentence, the themes line and the 6-character reset. A-24's app half reaches therapists only when this is promoted; until then, the database-side guard is what protects production.
2. **"Billing justification included"** (the note card, 3141) now labels a line that is a placeholder, not a justification. It needs a ruling.
3. **"What clients see" now leads the chat app.** Its screen 3 shows the new D-065 sentence; the chat app shows the old one until the client brief (#4-client) lands. The local branch `auto/s8-client-claims` exists in the chat repo at `c0771b4`, with no commits yet.

---

## 2. A-24 (B1): explicit columns

**Before** (`main` 2025, 2871): no `select=`.
```
/rest/v1/session_metadata?client_id=in.(<own client ids>)&order=created_at.desc&limit=100
/rest/v1/session_metadata?client_id=eq.<client>&order=created_at.desc
```

**After** (1978, 2825), with `const SESSION_METADATA_COLUMNS = 'client_id,created_at';` (1091):
```
/rest/v1/session_metadata?client_id=in.(<own client ids>)&order=created_at.desc&limit=100&select=client_id,created_at
/rest/v1/session_metadata?client_id=eq.<client>&order=created_at.desc&select=client_id,created_at
```

**Exact column list: `client_id,created_at`.** Why each one is needed, taken from every reader of these rows in the file after B4:

| Column | Read by (branch lines) |
|---|---|
| `client_id` | The home load groups its rows by client: `sessions.filter(s => s.client_id === link.client_id)` (1995). The client-page load does not read it. |
| `created_at` | Home: the 7-day `sessionCount` (1996), which drives the client header's "N sessions this week" and the dormant status; and `lastActivity` (2012), which drives Client Pulse "Last Alo: ...", the sidebar's "last active N days ago" and the rollout line. Client page: This Week's count and "Last used Alo" (2993, 3030), the prep pack's Engagement count, last date and date range (3632-3648), and the calendar dots and day popups (3938, 4053, 4106). The popup itself only counts rows (4125). |

Nothing else reads these rows. The only other field ever read was `primary_mode`, in the prep pack, which B4 removed. The column the brief's section 2 excludes appears nowhere in the file (0 occurrences), and no `select=*` appears anywhere in it. Both names are known to exist live: before this change the same requests already filtered on `client_id` and ordered on `created_at`, and both are in the seed's `insert into public.session_metadata`.

The constant carries a comment in the pattern of `CRISIS_EVENT_COLUMNS` (what the columns are for; never widen it). Following the brief, neither the code nor this report goes beyond "explicit columns". The database-side guard follows in Migration v1.6; this is the app half.

---

## 3. Copy: before / after, every string, every location

Line numbers are `main` (`a782b14`, as the brief cites them) -> branch. Em dashes are reproduced as they appear in the file.

| ID | Where | Before | After |
|---|---|---|---|
| D-001 + D-002 | Login trust line, 37 -> 37 | "Client data is encrypted at rest and in transit. Conversation history is saved for your clients, and they can delete it. You never see it." | "Client data travels over encrypted connections, and our database encrypts it at rest. Conversation history is saved to each client's Alo account, and they can delete it from there. You never see it." |
| S1 (new) | Login disclaimer, 38 -> 38 | "Alowen supports between-session reflection. It is not emergency care or a crisis service. In a crisis, follow your clinical protocol." | the same, then: "Alerts appear on this dashboard when you open it — Alowen doesn't page, text or email you." |
| D-021 + S1 | How Alo works, "What you'll see", 818 -> 758 | "When a client chooses to share something — a journal entry, a message, a summary — it shows up here. Homework you assign, and whether it got done. A safety alert if Alo ever detects one." | "When a client chooses to share something — a journal entry or a message — it shows up here. Homework you assign, and whether it got done. A safety alert if Alo ever detects one. Alerts appear on this dashboard when you open it — Alowen doesn't page, text or email you." |
| D-021 + S1 | First-screen overlay, "What you'll see", 855 -> 795 | as 818 | as 758 (the overlay carries the same text, so the S1 mirror applies) |
| D-024 | 819 -> 759, 856 -> 796 | "Your clients' conversations with Alo. Not summaries, not themes, not "insights."" | **kept**; B4 makes it true |
| D-058 | Invite message (Text invite and Email invite share `_aloInviteMessage()`), 4449 -> 4408 | "I mentioned Alo — it's for the time between our sessions. Here's your link: https://alowen.ai/?invite=${code}. Nothing you say there comes to me unless you choose to share it." | "I mentioned Alo — it's for the time between our sessions. Here's your link: https://alowen.ai/?invite=${code}. What you say there stays between you and Alo. I only see what you choose to share, when you've used it, and a safety note if Alo is ever worried about you." |
| D-065 | What clients see, screen 3, 4978 -> 4928 | "If Alo notices you might be in trouble, it will tell you where to get help -- and it will let your therapist know something happened. Not what you said. Just that you might need them." | "If Alo notices you might be in trouble, it will tell you where to get help — and it will leave a note for your therapist that something happened, which they see the next time they open their dashboard. Not what you said. Just that you might need them." |
| D-015 | Client Settings, history hint, 724 -> 714; 792 | "Client can view their own past sessions" | **kept** at 724; the 792 copy went with the dead modal (B5) |
| D-017 | Client Settings footnote, 742 -> 732 | "These settings control what features the client sees in their chat interface. Changes apply on client's next session." | "These settings control what features the client sees in Alo. Changes apply the next time your client opens Alo." |
| D-032 | Alerts empty state, 2126 -> 2079 | "No crisis alerts -- all clear" | "No open alerts from your active clients" |
| D-032 | Needs Attention empty state, 225 -> 225 | "All clear — no items needing attention." | "All clear — no open alerts from your active clients." |
| D-032 | Morning summary, 2085 -> 2038 | "All clear. " + counts | "All clear — no open alerts from your active clients. " + counts (deviation 6) |
| D-046 | What Changed, pending client, 3127 -> 3081 | "Client hasn't signed up yet. You can still write session notes and assign homework." | "Client hasn't signed up yet. Session notes and homework open up once they connect." |
| D-010 | Next Session Date hint, 662 -> 652 | "Used for session prep and Alo context" | "Saved with this client's record." |
| D-030 | Zero-client welcome, step 3, 1995 -> 1947 | "They connect via Alowen, and you see their activity here" | "They connect via Alowen, and what they choose to share — plus homework progress and any safety alert — shows up here" |
| D-048 | Copy for EHR header (clipboard), 3496 -> 3450 | "Based on Alowen between-session data -- review and edit before adding to client record." | "Session note from the Alowen dashboard — review and edit before adding to the client record." |
| D-008 | Clinical Rationale hint, 533 -> 523 | "For your records only -- client never sees this" | "For your records only — not shown in the client's app" |
| D-029 | Pending verification screen, 1726 -> 1678 | -- | **kept** |
| D-068 | Tour step 5, 5054 -> 5004 | -- | **kept**, pending the database check |

**Strings changed by the code tasks** (section 4 has the code):

| Task | Where | Before | After |
|---|---|---|---|
| B2 | Copy for EHR, all three formats (3519 -> 3475) | "Medical Necessity: Continued therapeutic support medically necessary for symptom management and functional improvement." | "Medical necessity: [add your justification]" |
| B2 | Copy P (3649 -> 3606) | "Continued therapeutic support medically necessary for symptom management and functional improvement." | "Medical necessity: [add your justification]" |
| B2 | Checkbox, Add Session Note and Edit Session Note (3283 -> 3237, 3396 -> 3350) | "Include billing justification" | "Add a medical-necessity line to complete" |
| B3 | Sensitivity label, tooltip and select (514-524, removed) | "Sensitivity" + "Low: Alo can reference this homework naturally. Medium: Alo mentions it only if the client brings it up first. High: Alo never names it directly -- works on the underlying skill only." | -- (removed) |
| B3 | More options button (537 -> 527, 2934 -> 2888) | "+ More options (type, sensitivity, rationale)" | "+ More options (type, rationale)" |
| B4 | Copy Prep Pack text (3693-3694, removed) | "Themes: Client-reported focus areas: <mode labels>." | -- (removed) |
| B5 | Default Client Settings modal (760-808, removed) | "These defaults apply to all new clients. Override per-client in Edit." / "Changes here apply to future clients only. Existing clients keep their current settings unless edited individually." / three setting hints / toast "Default settings saved" | -- (removed) |
| B6 | Reset prompt fallback (882 -> 823) | "Enter your new password (min 6 characters):" | "Enter your new password (min 12 characters):" |
| B6 | Reset prompt fallback, on refusal (new, 838) | (silent) | "Password must be at least 12 characters" |
| B6 | Set New Password placeholder (915 -> 858) | "Min. 6 characters" | "Min. 12 characters" |
| B6 | Set New Password validation (944 -> 889) | "Password must be at least 6 characters" | "Password must be at least 12 characters" |

**Static check (verification 3):** a table-driven check of every row above. Each new string is found exactly as many times as it has locations: S1 3, D-021 2, B2 placeholder 2, B2 label 2, B6 placeholder 2 and B6 message 3 (sign-up's own included), and 1 for each of the rest. Each kept string is still present: D-024 2, D-015 1, D-029 1, D-068 1. Every old verbatim string from the inventory is found 0 times: 30 strings, including "All clear. " and each B-task string. The same check against `main` fails 50 of its 53 rows. The 3 rows that pass there are kept strings that are identical on both.

---

## 4. Code tasks: before / after

### B2 -- no canned medical-necessity sentence (D-049)

`copyForEHR()` (3472-3476) and `copySoapSection()` for P (3604-3607). The ticked box used to append a fixed billing assurance that no clinician had written for that client. It now appends a line for the clinician to complete:

```
before:  text += '\n\nMedical Necessity: Continued therapeutic support medically necessary ...';
after:   text += '\n\nMedical necessity: [add your justification]';
```

The stored field (`billing_justification`, a boolean) is unchanged, so notes saved before still carry their tick and now copy with the placeholder. The harness recorded the three formats and Copy P on both builds. The branch's SOAP copy of a ticked note, for example:

```
Session note from the Alowen dashboard — review and edit before adding to the client record.

S (Subjective): Family conflict; sleep
...
P (Plan): Have the conversation this week



Medical necessity: [add your justification]
```

The extra blank lines before the last line are pre-existing: each SOAP/DAP section already ends in a blank line, and `main` shows the same gap (section 7.3).

### B3 -- Sensitivity hidden until the bot honours it (D-007)

The `<div class="form-group">` holding the Sensitivity label, its tooltip and `<select id="hwSensitivity">` became one line (514):
`<!-- S8 B3 (D-007): Sensitivity control removed; returns when the bot reads it (Therapist Homework spec, Gate B). -->`

The write (3803):
```
before:  sensitivity_level: document.getElementById('hwSensitivity').value,
after:   sensitivity_level: 'low', // S8 B3 (D-007): what the removed control sent by default
```
The column stays. A new card gets exactly the value an untouched form always sent; `'low'` is a value the seed itself uses. At desktop width, Type now sits alone in the first row of More options (the form grid has two columns); the screenshot was checked and nothing is broken. On phones this form grid is hidden anyway (a pre-existing rule).

### B4 -- the prep pack invents no themes (D-052)

Two lines of `generateAddendum()` were removed:
```
const themes = recentSessions.map(s => s.primary_mode || 'emotional processing')...join(', ') || 'general emotional processing';
text += 'Themes: Client-reported focus areas: ' + themes + '.\n\n';
```
A four-line comment (3650-3653) says why the line must not come back. The pack now reads: header, BETWEEN-SESSION DATA, Source, Engagement, Homework, Client Messages, Safety, then LAST SESSION NOTE.

| | Between Engagement and Homework |
|---|---|
| `main` | `Themes: Client-reported focus areas: <Alo mode label>, <Alo mode label>, ...` |
| branch | (nothing: Engagement is followed directly by Homework) |

No section is left dangling and no line has a label with nothing after it. The "Themes:" that remains in the pack belongs to the LAST SESSION NOTE section: it is the therapist's own note field ("Themes (Subjective)"), not anything derived from Alo, so it stays. Copy P and Copy for EHR never used this line; they read cleanly after B2 (above).

### B5 -- the unreachable Default Client Settings modal (D-019, D-020, R-B-34)

Removed: the modal markup (`main` 760-808, including its intro, footnote and the hints at 772/782/792), `showDefaultSettingsModal()` (`main` 4717), `saveDefaultSettings()` (`main` 4722) together with its `alo_default_settings` key, and `wireEscToStaticModal('defaultSettingsModal')` (`main` 1902). S7's snapshot of the static modals (`_pristineStaticModalHtml`) now holds four modals: invite, client settings, How Alo works, What clients see. The sign-out reset round-trips cleanly (harness).

### B6 -- one password policy (D-025, R-B-22)

| Surface | Before | After |
|---|---|---|
| prompt() fallback (821-839) | asks "(min 6 characters)"; sends at 6+; a shorter entry is dropped silently | asks "(min 12 characters)"; sends at 12+; a shorter entry gets the alert "Password must be at least 12 characters" |
| placeholder (858) | "Min. 6 characters" | "Min. 12 characters" |
| `submitPasswordReset()` (886-891) | refuses under 6: "Password must be at least 6 characters" | refuses under 12: "Password must be at least 12 characters" |

Sign-up (73-74, 1569-1570) is unchanged and now matches word for word. This is client-side only: the recovery PUT goes to GoTrue, which applies the hosted Auth minimum (section 8).

---

## 5. Verification -- the brief's ten items

| # | Item | Result |
|---|---|---|
| 1 | `grep -n "session_metadata" dev.html`: both request URLs carry `&select=` with an explicit list; the excluded column in neither; no `select=*` anywhere | PASS. Grep: 1086 (comment), 1978 (home read, `...&limit=100&select=${SESSION_METADATA_COLUMNS}`), 2043 and 2310 (comments), 2825 (client read, `...&order=created_at.desc&select=${SESSION_METADATA_COLUMNS}`). The constant (1091) is `'client_id,created_at'`, and the harness saw exactly `select=client_id,created_at` on both requests. The excluded column: 0 occurrences in the file. `select=*`: 0 |
| 2 | Home and client page still render "Last Alo" / check-in counts / calendar sessions from the narrowed rows | PASS (`a24_session_columns`). The stub returns only the selected columns. Home: "Last Alo: 1d ago" / "2d ago" / "No Alo activity yet", "last active 2 days ago", "1 of your 3 clients have opened Alo. 2 haven't yet." Client page: "3 sessions this week", "Used Alo 3 times this week", Alo dots on exactly the seeded days, the popup's "1 Alo check-in", and the prep pack's "Engagement: Client engaged in 3 Alo sessions (last: Sep 24)" with its date range. The same checks pass on `main`, so the narrowing changed no rendering |
| 3 | Every ruled string at every listed location; no old string remains; S1 in both places | PASS. Static: section 3's table check. Rendered: `copy_login_help`, `copy_home_client`, `copy_zero_clients` read every string off the live page (login lines, overlay, How Alo works, What clients see, the three D-032 states, pending client, Client Settings, rationale hint, the Text invite message and the email body built by the same function, welcome card) |
| 4 | Ticking the box inserts the placeholder, not the canned sentence, in Copy P and Copy for EHR | PASS (`ehr_copy_placeholder`, 33 checks): SOAP, DAP and narrative, plus Copy P, for a ticked and an unticked note; a note saved from the Add Note form with the box ticked; both checkbox labels |
| 5 | No Sensitivity control; `submitHomework()` still writes a valid row | PASS (`homework_no_sensitivity`): no `#hwSensitivity`, tooltip or the word "sensitivity" in the form. The POST answered 201 with `sensitivity_level: "low"`. The stub enforces a low/medium/high check constraint, and the row was stored and listed in Homework History |
| 6 | No "Themes:" line and no `primary_mode` read; no dangling label | PASS (`prep_pack_no_themes`; static: `primary_mode` 0). The only "Themes:" left is the therapist's own note field in LAST SESSION NOTE (section 4, B4) |
| 7 | No `showDefaultSettingsModal` / `saveDefaultSettings` / `alo_default_settings` | PASS. Static: 0 / 0 / 0. Dynamic: `no_default_settings_modal` |
| 8 | A 6-character reset password refused with the 12-character message in all three places | PASS. The modal (`password_reset_modal`): the placeholder says 12; 6 and 11 characters are refused with the message and nothing is sent; 12 is sent. The fallback (`password_reset_fallback`, the modal forced to fail): the prompt says 12; 6 characters are refused with the message and nothing is sent; 12 is sent, then "Password updated!" |
| 9 | Inline scripts pass `node --check`; the harness loads the login page, home and a client page with no console error | PASS. There is one inline script (dev.html 809-5122); `node --check` OK. `console_clean` loads the login page, home, a client (both tabs) and a pending client: 0 console errors, 0 exceptions, 0 browser-log errors. `real_cdn_load` does the same with the two pinned driver.js files from cdnjs under their SRI hashes and the page's CSP. Every other scenario also ends with 0 of each |
| 10 | `scan-invisible.py` CLEAN; zero U+200B/U+200C/U+200D/U+FEFF/U+2060 in the diff | PASS. `dev.html: CLEAN`, `dev.css: CLEAN`, exit 0, before every commit. The diff's added lines hold none of those, and no NBSP or smart quote |

---

## 6. Harness results

**Method** (the S7-dash pattern, rebuilt). Headless Google Chrome 153.0.8010.54 is driven over the DevTools protocol by a zero-dependency Node 25.8.0 script (built-in `http`, `child_process`, `WebSocket`). A local static server serves the two builds straight from git (`HEAD` = `dd111fd` for the branch, `main` = `a782b14`), so only committed content is tested, and every scenario runs against both in a fresh browser context. Nothing in either page is patched, with one exception: `password_reset_fallback` forces the Set New Password modal to throw, which is the only way to reach its catch. Before any page script, two scripts are injected:
- **a Supabase emulation** that answers `window.fetch` for the project's origin. It covers the GoTrue password and refresh grants, the recovery PUT, and PostgREST over an in-memory database: eq/neq/in/is/gt/lt filters, `select` projection (only the selected columns come back), `order`, `limit`, `Prefer`, and upsert with `on_conflict`. It returns PostgREST's 400 for any unknown column, and enforces a check constraint on `homework_cards.sensitivity_level`. It has no row-level security, on purpose. In the seeded `session_metadata` rows, every column the dashboard does not ask for holds a canary value: the seed's columns, plus the one the brief's section 2 names;
- **a clipboard recorder**, so copied text (the prep pack, Copy for EHR, Copy P, the invite message) can be read back without touching the machine's clipboard.

**Hermetic by construction:** every request the page makes is intercepted at the network layer and failed, unless it goes to the harness's own server or is one of the two pinned driver.js URLs. Those two are answered from local copies whose SHA-512 matches the page's SRI pins; `real_cdn_load` lets them go to cdnjs. Requests that tried to leave the machine: 0. Calls to the live Supabase API: 0. Real credentials used: none.

| Scenario | What it checks | Branch | `main` |
|---|---|---|---|
| `a24_session_columns` | Verifications 1-2: the request shape, the delivered row keys (12 rows), canaries absent from every response, the DOM, every input, localStorage, the caches and copied text; home and client-page rendering | PASS 24/24 | FAIL 20/24: no `select=`, full rows delivered, canaries in `clientSessions`, the DOM and the prep pack |
| `prep_pack_no_themes` | B4 | PASS 15/15 | FAIL 11/15: a themes line carrying the mode labels |
| `ehr_copy_placeholder` | B2 + D-048 | PASS 33/33 | FAIL 16/33: the canned sentence in all four copies, the old header, the old labels |
| `homework_no_sensitivity` | B3 + D-008 | PASS 18/18 | FAIL 12/18 |
| `no_default_settings_modal` | B5 | PASS 12/12 | FAIL 8/12 |
| `password_reset_modal` | B6, modal | PASS 12/12 | FAIL 7/12: 6 and 11 characters sent |
| `password_reset_fallback` | B6, prompt() fallback | PASS 10/10 | FAIL 7/10: "(min 6 characters)"; "abc123" sent |
| `copy_login_help` | D-001/D-002, S1, D-021, D-024 kept, D-065 | PASS 15/15 | FAIL 8/15 |
| `copy_home_client` | D-032 x3, D-046, D-010, D-017, D-015 kept, D-008, D-058 | PASS 15/15 | FAIL 6/15 |
| `copy_zero_clients` | D-030 | PASS 7/7 | FAIL 5/7 |
| `console_clean` | Verification 9 | PASS 6/6 | PASS 6/6 (control) |
| `real_cdn_load` | Verification 9 against the real CDN | PASS 8/8 | PASS 8/8 (control) |

**Totals:** branch 175/175; `main` 114/175. Every scenario includes the five hygiene checks: it ran to the end, no uncaught exception or unhandled rejection, no console error, no network/CSP/SRI error, and no request left the machine.

**Phone width** (`CLAUDE.md`: mobile-first). The surfaces whose copy grew were screenshotted at 375x812 and each image was looked at: the login page, the first-screen overlay, home in its all-clear state, How Alo works, What clients see (screen 3), a pending client, Client Settings (the Next Session hint and the footnote), and the zero-client welcome. The homework form and the Add Note form were also checked at 1280x900. No horizontal scroll anywhere (scrollWidth equals viewport width). No colour, spacing or CSS change was needed.

**Where it is:** the session scratchpad, not committed (the S7 precedent). The files are `harness/stub.js`, `lib.mjs`, `scenarios.mjs`, `run.mjs` and `shots.mjs`, plus `copycheck.py` and `precommit.sh`. It needs only Node and Chrome. It can go on this branch under `scripts/harness/` if wanted.

---

## 7. Sequencing, needs a ruling, found along the way, declined

### 7.1 Sequencing

1. **Production.** `index.html` was out of scope and is unchanged. It still reads `session_metadata` with no `select=` (index 1562, 2225), and still has the canned sentence (2 places), the themes line and the 6-character reset. A-24's app half reaches therapists only when `dev.html` is next promoted. Until then, production relies on the database-side guard (Migration v1.6). The S7 sequencing note stands: this branch builds on S7's `dev.html`, and neither has been promoted.
2. **"What clients see" and the chat app.** Screen 3 now shows the new D-065 sentence. The chat app's onboarding screen (client 1503) still shows the old one until the client brief lands; its C-044 row carries the same sentence, em dash included. The dashboard constant's contract is to show what clients see. For the window between the two merges, the dashboard shows the ruled sentence, not the live one. Merging both briefs together closes the window.

### 7.2 Needs a ruling

1. **"Billing justification included"** (the note card's meta line, `main` 3187 -> 3141). This line is not in the ruling. It still appears under a note whose box is ticked, but what that tick now adds is a placeholder, not a justification. It is left alone under "nothing else changes". A candidate wording: "Medical-necessity line to complete".
2. **The three em dashes** (deviation 4), if Kano prefers `--` in those places.

### 7.3 Found along the way, pre-existing, not changed

1. **Date-only fields show a day early west of UTC.** `new Date('2026-09-22')` is UTC midnight, so the page shows notes and next-session dates one day early in the Americas. The harness saw a note dated 2026-09-22 as "Sep 21" in the prep pack's LAST SESSION NOTE, and a note dated 2026-09-15 as "Monday, September 14" on its note card. Next-session date 2026-09-29 appears as "Monday, September 28" in Copy P. This is on `main` too. It is EHR-bound text, so it is worth a ticket; it is out of this brief's scope.
2. **Copy for EHR, SOAP and DAP:** three blank lines before the medical-necessity line, because each section already ends in a blank line and the line starts with its own. It is on `main` too, and a one-line `trimEnd()` if wanted.
3. **Four other reads have no `select=`:** `client_messages` (home 1975, client 2823), `session_notes` (2821) and `homework_cards` (2822). All four return rows the therapist is meant to see: their own notes and cards, and messages written to them. No column in them is known to be sensitive. They are noted only because they have the same shape A-24 fixes for `session_metadata`.
4. **Now-unused CSS:** `.tooltip-trigger`, `.tooltip-icon`, `.tooltip-content` (dev.css 1306-1358) and `.modal-intro` (1558). They are left in place because the brief allows `dev.css` changes only when a fix needs them, and none did. They are candidates for the R-B-34 dead-code pass.
5. **The note forms' checkbox stacks above its label** (`.form-group-inline` computes to a flex column). This is the same on `main`. The new, longer label wraps to two lines at desktop width; it reads fine.

### 7.4 Declined, as the brief says

The alerts query's archived-client gap (R-B-28, S5). D-032's copy now scopes itself to "your active clients", which is what the query covers; the query itself is Gate B. Realtime alerting and P-4 email (Gate B). Anything in the client app or the bot. S3/S4 UX polish. `index.html`. `dev.css`, since no fix needed it. No change to the hosted password minimum, which is auth configuration and needs approval plus a test plan.

---

## 8. Not verifiable here

1. **The hosted minimum for a password reset** (inventory section 3, Q16c). Both reset paths now refuse under 12 in the browser. The recovery PUT goes to GoTrue, which applies whatever minimum the hosted Auth settings hold. If that is still 6, a request sent outside the dashboard could set a shorter password.
2. **"our database encrypts it at rest"** (D-001). This is a Supabase platform property; the wording is the ruling's.
3. **"they can delete it from there"** (D-002). The client app has delete paths for conversations and memories. Whether the live policies let those deletes land is the inventory's Q4 (R-C-08).
4. **S1, "Alowen doesn't page, text or email you".** This is true of the code. The dashboard has no notification, push, service-worker or polling code (grep: 0). Per the inventory, no Edge Function sends mail and the only outbound mail is account email. It stays true until P-4 or realtime alerting ships (Gate B); the sentence has to change in the same change.
5. **`homework_cards.sensitivity_level`'s live constraint.** `'low'` is a value the seed itself inserts. The live CHECK and default are Studio facts.
6. **Real devices.** Headless Chrome at 375 px is not an iPhone or an iPad.

---

## 9. Kano's check on dashboard.alowen.ai/dev.html after merging (test account, about ten minutes)

1. **Sign in.** Home with no open alerts reads "All clear — no open alerts from your active clients." in the headline and in Needs Attention.
2. **A-24.** DevTools, Network, filter `session_metadata`: both requests (home, then open the test client) end in `select=client_id,created_at`, and each response row has exactly those two keys. "Last Alo", "N sessions this week" and the calendar dots look as before.
3. **Copy Prep Pack** on the test client: no line between Engagement and Homework.
4. **A note with the box ticked** (the box now reads "Add a medical-necessity line to complete"): Copy for EHR, SOAP, ends in "Medical necessity: [add your justification]", and the first line is the new header.
5. **Assign Homework > More options:** Type, Treatment Plan Goal, Clinical Rationale. No Sensitivity.
6. **Help > How Alo works**, and the login page: the alerts sentence is there.
7. **Invite > Generate > Text invite** (desktop copies it): the new message.
8. **While in Studio (section 8):** the hosted password minimum, and `homework_cards.sensitivity_level`'s constraint.

---

*S8 dash claims fixes v1.0 -- 2026-09-26. Branch `auto/s8-dash-claims`, eight commits with this one. `dev.html` and this report. Nothing on `main`, nothing promoted, no live call.*
