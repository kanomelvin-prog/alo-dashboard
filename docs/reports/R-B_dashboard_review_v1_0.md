# R-B -- Dashboard Deep Review v1.0 (findings-only)

**Repo:** `kanomelvin-prog/alo-dashboard` -- branch `auto/r-b-review` off `main` @ `7e7bb95`.
**Date:** 2026-09-16. **Brief:** `claude_Brief_R-B_Dashboard_Review_v1_0.md`.
**Deliverable:** this file only. No app file, CSS file, or `CLAUDE.md` was modified. No live-DB call was made. Nothing was pushed to `main`.

---

## Session Log

1. **Model.** Claude Fable 5.1 (`claude-fable-5-1`), effort max, bypass permissions, unattended.
2. **Interruptions and restarts.** None in the review itself. The optional headless-Chrome rendering check (brief ground rules) timed out twice on `--dump-dom`; its console capture did complete (see section 6). No context compaction, no retried edit.
3. **Deviations from the brief, each with a reason.**
   - **Ledger D36 and D38 were not available.** The brief cites them for the A7 status summary and the scoping question. Every local copy of `alo-supabase/docs/decisions.md` (all branches, all three repos) tops out at D33; the only local mention of D37 is the consent-gate brief. D36/D38 therefore live only in the claude.ai project. A7 status was re-verified against the in-code `A7/Dn` markers and the A7 comments instead, and the D11-D17 list could not be enumerated by number (section 3, R-B.7).
   - **The March 31 audit (`Adversarial_Security_Audit_2026_03_31`) and `A7_Remediation_Tracker_v1_0.md` are not on this machine** (Spotlight, `find`, and `grep` over `~/code`, `~/Downloads`, `~/Desktop`, and the iCloud vault). Same consequence as above.
   - **Plan v1.9.2 is not local.** The A-nn finding definitions (A-07, A-15, A-23) and the A/S5 section-5 items were taken from `claude_Alo_Pilot_Fix_and_Improve_Plan_v1_5_EXT.md`, `claude_Alo_Bot_Logic_Review_v1_0_EXT.md`, and `claude_Alo_Client_UX_Review_v1_0_EXT.md` in `~/Downloads`; the RLS facts from `claude_Supabase_RLS_Lockdown_Migration_v1_2_EXT.md`.
   - **`CLAUDE.md` session-start instruction** ("ask what we are working on today") was not followed literally because the run was unattended with an explicit brief; the three vault files it names were read.
   - **The report opens with this Session Log** per the standing report convention in `alo-supabase/docs/decisions.md`, in addition to the six sections the brief specifies.
4. **Checklist.** R-B.1 COMPLETED. R-B.2 COMPLETED. R-B.3 COMPLETED. R-B.4 COMPLETED. R-B.5 COMPLETED. R-B.6 COMPLETED. R-B.7 COMPLETED (D2-D8 verified; D11-D17 not enumerable, see deviation). R-B.8 COMPLETED. R-B.9 COMPLETED. Report sections 1-6 COMPLETED.

**What was read.** `dev.html` in full (4,625 lines, every line); `index.html`, `dev.css`, `styles.css` via full `diff` plus targeted reads; both `CLAUDE.md` files; `alo-supabase` `docs/decisions.md`, `README.md`, all migrations, `open-as-client`, `exchange-client-token`, `_shared/tokens.ts`, `PRE_PILOT_TEST_PASS.md` excerpts; `alo-client-chat-full_v1/dev.html` excerpts (invite claim, token exchange, settings reads, consent write); the vault's Philosophy, Roadmap, Dupre notes, and Client-Architecture excerpts; the prior reports in this repo (R1, Phase 2, Phase 2B, Open-as-client, Session).

**Checks run.** `node --check` on the extracted inline script of both HTML files: both pass. `python3 scripts/scan-invisible.py dev.html dev.css index.html styles.css`: CLEAN on all four, exit 0. `diff dev.html index.html`: 803 differing lines; `diff dev.css styles.css`: 212. Headless Chrome (local `http.server`, fresh profile): no `Uncaught`, no CSP `Refused` on either page at load; DOM dump not obtained. Date-parsing behaviour confirmed with `node` under `TZ=America/Los_Angeles`.

---

## 1. Summary table

Severity per the brief's scale. "Regr." = regression or reopening of an existing A-/A7- finding.

| ID | Sev | Angle | file:line | Title | Regr. |
|---|---|---|---|---|---|
| R-B-01 | S1 | R-B.3 | dev.html:1729, 1738-1748 | 50-row cap on the home crisis query lets guest (null-client) events push a therapist's own clients' open alerts out of Needs Attention and out of the CRISIS status | -- |
| R-B-02 | S1 | R-B.2 | dev.html:3483, 1729, 2439 | `action_notes` (bot-authored per A-15, carries the client's verbatim excerpt) is rendered to the DOM for resolved alerts, and is fetched by `select=*` on every crisis query | A-15 |
| R-B-03 | S1 | R-B.1 | dev.html:1478-1483, 1526-1550, 2241-2244 | Sign-out does not reset the view: the next sign-in in the same tab shows the previous user's last-open client page (notes, messages, phone, emergency contact, safety notes) | -- |
| R-B-04 | S2 | R-B.9 / R-B.1 | index.html:1387-1397 | Production lockout is the pre-D22 version: filters `therapists.id`, selects a non-existent column, and defaults to unlocked on every account (fails open) | D22 unpromoted |
| R-B-05 | S2 | R-B.5 | dev.html:1804-1806, 3976-3981, 4129-4134, 4192-4197 | `loadDashboard()` swallows every error so it never rejects; the three A7-D3 refresh-failure toasts are unreachable; a failed initial load leaves "Loading your dashboard..." forever | A7-D3 |
| R-B-06 | S2 | R-B.5 | dev.html:1178-1184, 2192-2201, 4057-4071, 4116-4121, 4183-4187, 2978-2997, 3016-3024, 2085-2093 | Every mutation uses `Prefer: return=minimal`; a PATCH/DELETE that RLS or a stale id filters to zero rows returns 204 and is toasted as success | -- |
| R-B-07 | S2 | R-B.3 | dev.html:1729, 1826-1837, 1855-1865, 2111-2113, 2192-2198 | Guest crisis events are delivered to every therapist as "Unknown Client" with a Resolve button; any therapist can acknowledge them; the resolve modal's emergency panel falls back to `currentClient` | -- |
| R-B-08 | S2 | R-B.8 / R-B.4 | dev.html:622, 3954-3964 | Invite modal says "Expires in 7 days"; no expiry exists in the dashboard, the chat claim, or any migration | -- |
| R-B-09 | S2 | R-B.8 | dev.html:616; chat dev.html:1466-1474 | Invite modal says the display name is "visible only to you"; the chat app copies it into the client's own profile on claim | -- |
| R-B-10 | S2 | R-B.8 | dev.html:37 | Login page: "Alowen conversations are not permanently saved -- chat history is automatically cleared." Transcripts are stored (A-23) and pinned threads persist | A-23 sibling |
| R-B-11 | S2 | R-B.8 | dev.html:4441, 1797, 2043-2045, 2637-2648, 2613-2619 | "What clients see" screen 2: "Nothing is shared unless you share it." Presence, session counts, homework viewed/completed, and crisis alerts flow without client action | -- |
| R-B-12 | S2 | R-B.8 | dev.html:3268-3272 | Prep pack writes "Alert on <date>: <category>; resolved via <alo_response_action>" for **unresolved** alerts, into EHR-bound text | -- |
| R-B-13 | S2 | R-B.8 | dev.html:4566, 3417 | Tour: "Every alert, open or resolved, stays here with the action taken." Resolved alerts leave the card after 7 days | -- |
| R-B-14 | S2 | R-B.6 | dev.html:1730, 2085-2093 | Home `client_messages` query and the mark-read PATCH filter on `client_id` only; the per-client query (2437) filters on `therapist_id` too | -- |
| R-B-15 | S2 | R-B.6 | dev.html:2374-2380 | `journal_entries` is filtered by `client_id` and a boolean `shared_with_therapist`; there is no therapist target, so any therapist who can reach the row sees the entry | -- |
| R-B-16 | S2 | R-B.6 | dev.html:2978, 3016, 4057, 4116, 4183, 2869-2871, 3157-3159, 3383-3385, 3956-3963 | By-id mutations omit the `therapist_id` guard the tables offer; POST bodies carry a client-supplied `therapist_id`; scoping rests entirely on RLS | -- |
| R-B-17 | S2 | R-B.6 / R-B.8 | dev.html:682, 4057-4068; chat dev.html:1385, 1973, 2059 | `safety_notes`, phone, and emergency contact are written to the same `therapist_clients` row the client's own JWT reads and updates; "Never shared with client" holds only at the UI layer | -- |
| R-B-18 | S2 | R-B.4 | dev.html:3954, 3956-3963 | Invite code is `Math.random`, 6 base-36 characters (about 31 bits), never expires, and the chat claim endpoint is a yes/no oracle | -- |
| R-B-19 | S3 | R-B.5 / R-B.6 | dev.html:4276, 4614 | `therapists` is queried and patched by `id=eq.<auth uid>` but `therapists.id` is a random uuid; the first-screen overlay never appears for therapist-signup accounts and onboarding timestamps never persist (silent 204) | D22 class |
| R-B-20 | S3 | R-B.1 | dev.html:1345-1352, 1470-1472 | A rejected non-therapist login leaves its session in `localStorage` and in the `session`/`therapistId` globals | -- |
| R-B-21 | S3 | R-B.1 | dev.html:1478-1483 | Sign Out is local only; no `/auth/v1/logout`, the refresh token stays valid server-side | -- |
| R-B-22 | S3 | R-B.1 | dev.html:863-870, 894-925, 934, 958 | Recovery token: hash cleared only on success, token embedded in an inline `onclick`, modal has no cancel, reset accepts 6-character passwords against a 12-character signup policy | A7-D10 partial |
| R-B-23 | S3 | R-B.5 | dev.html:1120, 4370-4374, 4381-4389 | No request has a timeout; a stalled connection leaves every button spinning forever and `openAsClient` leaves a blank tab open | -- |
| R-B-24 | S3 | R-B.5 | dev.html:827-832, 1583-1588; dev.css:1278 | Every toast, including every failure toast, renders with the green success checkmark | -- |
| R-B-25 | S3 | R-B.5 | dev.html:2416-2419 | Shared-journal load failure hides the card silently; entries the client explicitly shared vanish with no signal | -- |
| R-B-26 | S3 | R-B.1 | dev.html:1123-1145, 1728-1733 | Four concurrent 401s trigger four parallel refreshes with the same refresh token; no mutex | -- |
| R-B-27 | S3 | R-B.5 | dev.html:2673, 2723, 2743, 3292, 3552; 2807, 3160 | `session_date` renders one day early in US time zones; new notes default to the UTC date (tomorrow after 5 pm Pacific) | -- |
| R-B-28 | S3 | R-B.2 | dev.html:1682, 1725-1729 | Archived clients' open crisis events are excluded from the home query; the dashboard reads "All clear" | -- |
| R-B-29 | S3 | R-B.8 | dev.html:1511 | Pending-verification screen tells a locked-out therapist to email `kano@alowen.ai`, which the plan lists as not yet live | -- |
| R-B-30 | S3 | R-B.8 | dev.html:3248-3249 vs 810, 847 | Prep pack labels Alo interaction modes (VENT, UNDERSTAND...) as "Themes", against "Not summaries, not themes" | -- |
| R-B-31 | S3 | R-B.8 | dev.html:2684 vs 2859-2862, 3146-3149, 3372-3375 | "You can still write session notes and assign homework" for a pending client; both actions are refused for pending clients | -- |
| R-B-32 | S3 | R-B.5 | dev.html:1692-1722 | Zero-client branch prepends a new welcome card on every call and hides the home sections permanently; after the first invite the page is stale until reload | -- |
| R-B-33 | S3 | R-B.9 | dev.html:2049, 1830, 1907, 2562, 3603-3604, 827 | Accessibility: client and alert rows are click-only `div`s with no keyboard path; calendar cells have `role=button` but no key handler; toast has no `aria-live`; dynamic modals have no `role=dialog` | -- |
| R-B-34 | S4 | R-B.9 | see section 2 | Dead code list (strip calendar, default-settings modal, `therapistAvatar`, reset `prompt()` fallback, unused locals, hidden card, `false_positive` never surfaced) | D19 |
| R-B-35 | S4 | R-B.9 | see section 2 | Duplicated code (emergency panel x2, anon key literal x4, first-screen copy x2, sidebar markup x2) | -- |
| R-B-36 | S4 | R-B.9 | dev.html:1508-1512, 2034, 2259, 2262, 2552, 2556-2559, 3534-3553, 3721-3750 | 32 hard-coded hex colours and inline styles in JS/markup against the custom-properties rule | -- |
| R-B-37 | S4 | R-B.9 | dev.html:1828, 1837, 1865, 1907, 1932, 1972, 2164, 2562, 2916, 3470, 4163; 1041 | `escJsAttr` applied to note ids only; forgot-password success interpolates the typed email unescaped (self-XSS only) | A7-D8 note |
| R-B-38 | S4 | R-B.9 | dev.html:19; manifest.json:5; CLAUDE.md | `manifest.json` `start_url` is `./dev.html`; a promotion that copies the manifest link installs an app that opens dev; `CLAUDE.md` line counts are stale | -- |
| R-B-39 | S4 | R-B.8 | dev.html:3115, 1774, 3098, 2565, 4423 | Copy nits: "Add in Edit" for a modal titled Client Settings; test client counted as a client in the greeting; emergency-info warning fires on the test client; Client Pulse ignores the "Your Test Client" label; "Try again" on the permanent D9 403 | -- |

Counts: S1 x3, S2 x15, S3 x15, S4 x6.

---

## 2. Findings

### R-B-01 -- S1 -- 50-row cap lets guest events displace own-client alerts (R-B.3)

**What.** The home page's only crisis query is

```
dev.html:1729  crisis_events?or=(client_id.in.(<ids>),client_id.is.null)&acknowledged=eq.false&order=created_at.desc&limit=50
```

Per-client status is then derived from that same array:

```
dev.html:1738  const clientCrisis = crisisEvents?.filter(e => e.client_id && e.client_id === link.client_id) || [];
dev.html:1745  const unresolvedCrisis = clientCrisis.filter(c => !c.acknowledged);
dev.html:1748  if (unresolvedCrisis.length > 0) status = 'crisis';
```

**Why it matters.** The query is ordered newest-first and capped at 50 across the therapist's clients *and* every unacknowledged guest event on the platform. Fifty newer guest rows push an older open alert for one of the therapist's own clients out of the result. That client then has no CRISIS pill, no Needs Attention row, no greeting count, and the "alerts need your attention" line is not shown. The Bot Logic Review already notes guest rows "accumulate" and are "orphaned forever" (never acknowledged), so the pool that does the displacing only grows. The client page's own query (`dev.html:2439`, no limit) still shows the alert if the therapist happens to open that client, which is the only reason this is not worse.

**Root cause.** One query serving two purposes (attention feed and per-client status) with a hard cap and a null-client branch that admits rows belonging to nobody.

**Fix.** Drop `client_id.is.null` from the therapist query (see R-B-07 for the ruling), query only the therapist's own client ids, and either remove the limit or compute per-client status from a separate count query. Guest events need their own surface.

**Regression.** No prior finding; original design (present since the first `index.html` commits that carry the string).

### R-B-02 -- S1 -- `action_notes` reaches the DOM and the browser (R-B.2, A-15 dashboard side)

**What.** The brief's A-15 rule: any rendering of bot-authored free-text `action_notes` is S1. A-15 (Bot Logic Review, Sep 11) establishes that Safety_Check writes `action_notes: "Tier X: " + userMessage.substring(0, 200)`. The dashboard renders that column:

```
dev.html:3483  Action: ${esc(c.action_taken || 'resolved')}${c.action_notes ? ' -- ' + esc(truncate(c.action_notes, 80)) : ''}
```

inside the "Recently resolved" list of `renderCrisisAlerts`, on both the Session Prep and In Session tabs, for every event acknowledged within the last 7 days (`dev.html:3417`).

**Exposure conditions, stated exactly.** The dashboard's own resolve flow overwrites the column first:

```
dev.html:2197  action_notes: notes        // the therapist's textarea, '' when empty
```

so an event resolved through the modal renders the therapist's words, not the bot's. The excerpt renders when a row is acknowledged by any other path: the seeded demo row (`seed_therapist_account.sql:372-383`, `acknowledged = true`, `action_notes` ending in `(Original flag: client said "what's the point some days" ...)`, cut off here only because `truncate(..., 80)` stops before the quote), any row acknowledged in Studio or SQL, and any future change to the PATCH body. The rendering path exists today and is one data condition away from showing a client's words to a therapist.

**Second half.** Both crisis fetches use `select=*`:

```
dev.html:1729  (home)      dev.html:2439  crisis_events?client_id=eq.${clientId}&order=created_at.desc
```

so the excerpt is delivered to the therapist's browser on every load regardless of what is rendered. It is visible in the Network panel, held in `clientCrisis[...]` and `window._allCrisisEvents`, and read by the prep pack generator. "The therapist never sees what the client said" is currently true of the DOM, not of the therapist's machine.

**Also rendered.** `dev.html:3467` renders `c.details` verbatim for unacknowledged alerts. No migration or seed defines a `details` column; if Botpress writes one, it is displayed as-is. Unverifiable without Studio (section 6).

**Fix.** Never render `action_notes` authored before acknowledgement (render only what the dashboard wrote, or nothing); replace `select=*` with the columns actually shown (`id, client_id, severity, category, alo_response_action, acknowledged, acknowledged_at, action_taken, created_at`); run the A-15 data scrub (plan item 2.8) including the seed. Answers plan item 5.2: yes, the crisis UI renders `action_notes`.

**Regression.** Dashboard side of A-15 (open ship-blocker).

### R-B-03 -- S1 -- Sign-out leaves the last client page in place for the next sign-in (R-B.1)

**What.**

```
dev.html:1478-1483  handleLogout(): session = null; therapistId = null; localStorage.removeItem('alo_session'); showLogin();
dev.html:1519-1524  showLogin(): loginPage display flex; appContainer display none
```

Nothing resets `homePage` (set to `display:none` at `dev.html:2241` by `showClient`) or removes `active` from `clientPage` (`dev.html:2244`), and nothing clears `currentClient`, the per-client caches, or the rendered notes, messages, and emergency panel. On the next successful sign-in, `showApp()` (`dev.html:1526-1528`) simply flips `appContainer` back to `display:block`. The client detail page is what appears, populated with the previous user's data, until Home is clicked. `loadDashboard()` refreshes the hidden home lists and the sidebar, not the detail column.

**Why it matters.** On a shared device (a clinic front desk, a therapist's office computer), therapist A signs out and therapist B signs in: B sees A's last client's name, session notes, client messages, phone, emergency contact, and safety notes. That is data exposure across therapists, which the brief defines as S1. The precondition is narrow (same tab, sequential users) but it is an ordinary thing to do with a shared computer. The session-expiry path (`dev.html:1161-1167`) has the same shape for a single user (stale page after re-login), which is a lesser version of the same bug.

**Root cause.** Logout and login are treated as visibility toggles on the same live DOM.

**Fix.** `handleLogout()` and the forced-logout branch in `api()` should call `showHomePage()`, clear the five per-client caches, `window._allCrisisEvents`, `window._allHomeMessages`, and blank the detail containers; `showApp()` should assert the home view.

**Regression.** No prior finding.

### R-B-04 -- S2 -- Production lockout fails open (R-B.9, R-B.1; D22 not promoted)

**What.** `index.html` on `main` still carries:

```
index.html:1387  async function isVerificationLocked(id) {
index.html:1389    const rows = await api(`/rest/v1/therapists?id=eq.${id}&select=license_verified,verify_deadline`);
index.html:1391    if (!row) return false;
index.html:1392    if (row.license_verified) return false;
index.html:1396    console.log('Verification check failed, defaulting to unlocked:', e.message);
index.html:1397    return false;
```

`therapists.id` is not the auth id (spec test 1: "one therapists row with random `id` and correct `auth_user_id`"), and `license_verified` does not exist (ledger, step-4 review: "No verification or approval column exists"), so the request fails or matches nothing and the function returns `false` on every account. The `verify_deadline` gate is a no-op in production.

**Why it matters.** The brief's S2 includes "a path that fails open". This is the live dashboard. It is already known (ledger D22: "ships with the next promotion under a LIVE CHANGE banner") and recorded here so it is not lost in triage, and because the parity diff (section 5) shows every other unpromoted change riding along with it.

**Fix.** Promote `dev.html` (D22 is in it at `dev.html:1485-1499`).

**Regression.** D22 pending promotion; not a code regression.

### R-B-05 -- S2 -- `loadDashboard()` never rejects; A7-D3 wrappers are dead (R-B.5, A7-D3)

**What.**

```
dev.html:1804-1806  } catch (err) { console.error('Dashboard load error:', err); }
```

is the only handler in `loadDashboard()`, and it does not rethrow. The three A7-D3 call sites

```
dev.html:3976-3981  try { await loadDashboard(); } catch (refreshErr) { ... toast('List refresh failed -- reload the page to see latest'); }
dev.html:4129-4134  (archive)      dev.html:4192-4197  (restore)
```

can never enter their `catch`. The toast they were added to deliver is unreachable. On a failed initial load (network, RLS, 5xx) `showApp()` (`dev.html:1548`) resolves normally and the home page keeps the placeholder "Loading your dashboard..." (`dev.html:192`) with no toast, no retry, and an empty client list.

**Why it matters.** The audit finding was "loadDashboard awaited with error handling"; the await is there, the error handling is neutralised by the callee. A therapist whose first load fails sees a page that looks like it is still loading, indefinitely.

**Fix.** Rethrow from `loadDashboard()` after logging (or return a status) and let `showApp()` render an explicit error state with a retry.

**Regression.** A7-D3 is present in both files but ineffective in both.

### R-B-06 -- S2 -- Zero-row PATCH/DELETE reported as success (R-B.5)

**What.** `apiMutate()` (`dev.html:1178-1184`) and the in-place PATCH/DELETE calls all send `Prefer: return=minimal`. PostgREST returns 204 for a filtered-to-zero-rows update exactly as for a successful one. The dashboard cannot tell them apart and toasts success:

| Call | Line | Toast |
|---|---|---|
| resolve crisis | 2192-2201 | "Crisis alert resolved" |
| client edit (safety notes, emergency contact, allow_*) | 4057-4071 | "Client updated" |
| archive / restore | 4116-4121 / 4183-4187 | "Client archived" / "Client restored" |
| note update / delete | 2978-2997 / 3016-3024 | "Note updated" / "Note deleted" |
| mark messages read | 2085-2093 | (silent) |
| onboarding timestamps | 4276 | (silent) -- see R-B-19 for a confirmed zero-row case |

The chat app already handles this correctly for its consent write (`chat dev.html:1388-1394`: "a PATCH that RLS filters down to zero rows is still a 204 ... Re-read exactly what the gate will read").

**Why it matters.** The brief asks specifically about "a 204 that RLS filtered to zero rows". Under a misconfigured or tightened policy (the RLS lockdown v1.2 is scheduled and has two phases), the resolve flow would say "Crisis alert resolved" and the alert would come straight back on the reload. That is a false success on a safety table. It is S2 rather than S1 because the reload does re-render the truth for crisis and client edits; R-B-19 is the case where it does not.

**Fix.** Use `Prefer: return=representation` (or `Prefer: count=exact` and read `Content-Range`) on every mutation that matters and treat an empty result as failure.

### R-B-07 -- S2 -- Guest crisis events go to every therapist and can be resolved by any of them (R-B.3)

**What.** The `client_id.is.null` branch at `dev.html:1729` returns every unacknowledged crisis event with no client. They are rendered with a Resolve button:

```
dev.html:1827  const name = client?.display_name || 'Unknown Client';
dev.html:1834  ${esc(e.severity)} . ${esc(e.category)} . Alo: ${esc(e.alo_response_action)}
dev.html:1837  <button class="list-item-action" onclick="openResolveCrisisModal('${e.id}', event)">Resolve</button>
```

and the same in `showAllAlerts` (`dev.html:1855-1865`). `submitResolveCrisis` (`dev.html:2192-2198`) then writes `acknowledged_by: therapistId` on a row that has no relationship to the therapist.

**What a second therapist sees.** Identical rows. Therapist A and therapist B (any two accounts) both receive "Unknown Client . medium . passive_ideation . Alo: provided_988_resources . 2h ago [Resolve]" in Needs Attention, both count it in the greeting ("2 alerts -- alerts need your attention"), and whichever clicks Resolve first removes it for the other and becomes `acknowledged_by`. Neither has a phone number or contact to act on; the resolve modal's emergency panel reads "No phone on file". A therapist is being asked to "resolve" a stranger's crisis.

**Fallback hazard.** `dev.html:2111-2113` resolves the panel's client as `eventObj?.client_id ? allClients.find(...) : currentClient`. On the home page `currentClient` is null (`dev.html:2361`), so today the panel is empty. If `openResolveCrisisModal` is ever invoked for a null-client event while a client page is open, the panel shows *that* client's phone, emergency contact, and safety notes under a guest's alert.

**RLS, as written locally.** No migration in `alo-supabase` defines a policy on `crisis_events`. The only written policy text is the vault's `Client-Architecture.md:131` ("Therapists: read linked clients' crisis events"), which predates the current schema. If that policy is what is live, NULL rows are filtered out and the whole "Unknown Client" branch is dead code (and no one is alerted about guest crises, by design). If a broader SELECT exists (the RLS lockdown v1.2 shows a "System can insert crisis events" policy still in place and calls the export's state "alert fabrication"), every therapist sees every guest event. Which is true cannot be determined from the repos (section 6).

**Recommended scoping rule (for the D38 section 3 ruling; not implemented).** The therapist dashboard shows a crisis event only when `client_id` is in the therapist's active or pending links. Null-client events are an operations concern, routed to an admin surface or email under the service role, never to therapists. Concretely: the home query becomes `crisis_events?client_id=in.(<ids>)&acknowledged=eq.false`, the `or=` form and the "Unknown Client" rendering are removed, and the RLS SELECT policy on `crisis_events` is written to the same predicate so the client-side filter and the policy agree. Archived links are a separate ruling (R-B-28).

**Regression.** No prior finding; original design.

### R-B-08 -- S2 -- "Expires in 7 days" is false (R-B.8, R-B.4)

**What.** `dev.html:622`: `Expires in 7 days . Click code to copy`. `generateInvite` (`dev.html:3956-3963`) inserts `therapist_id, invite_code, display_name, status: 'pending'` and nothing else. The chat claim (`chat dev.html:1451`) filters `invite_code=eq.X&status=eq.pending` with no date check. No migration adds an expiry column to `therapist_clients`; the pg_cron "delete expired invite codes" in the vault refers to the retired `invites` table (`RLS Lockdown v1.2` drops its last policy). `PRE_PILOT_TEST_PASS.md:387` treats the text as expected display, not as verified behaviour.

**Why it matters.** A claim about what Alo does with a credential, shown to the therapist, false on this build. A pending invite is claimable indefinitely, which also feeds R-B-18.

**Fix.** Either implement expiry (a column written at mint time and checked by the claim policy) or change the copy.

### R-B-09 -- S2 -- "Friendly name visible only to you" is false (R-B.8)

**What.** `dev.html:616`: `Friendly name visible only to you`. On claim the chat app runs:

```
chat dev.html:1466-1474  // Seed client profile with therapist's chosen name (client can change later)
                          await api(`/rest/v1/profiles?id=eq.${user.id}`, 'PATCH', { display_name: therapistDisplayName });
```

The client's profile, and therefore the client's own header, shows whatever the therapist typed.

**Why it matters.** A privacy claim on a field therapists are invited to treat as private. A therapist who types a note-like label ("Sarah M. -- court mandated") hands it to the client.

**Fix.** Copy change ("The client will see this name") or stop seeding the client profile from it.

### R-B-10 -- S2 -- Login page retention claim is false (R-B.8)

**What.** `dev.html:37`: "Client data is encrypted at rest and in transit. Alowen conversations are not permanently saved -- chat history is automatically cleared."

**Evidence.** Plan v1.5 item 4.14 (A-23) records that `conversation_messages` stores full transcripts and the chat's About copy saying otherwise is false. The chat app pages through 50+ stored conversations (Phase 2 item 4) and distinguishes "Clear unpinned conversations", i.e. pinned threads are never cleared. The auto-expiration job's scope is not in any repo. The sentence is on the therapist's login screen and is the same class of claim as A-23.

**Encryption half.** In transit: true (HTTPS). At rest: a platform property of Supabase's managed Postgres; not verifiable from code.

**Fix.** Rewrite to what is true (conversations are stored for the client, deletable by the client, never shown to the therapist).

### R-B-11 -- S2 -- "Nothing is shared unless you share it" is false on this build (R-B.8)

**What.** `dev.html:4441` (What-clients-see modal, screen 2; approved Copy Pack v1.0): "If you want to share something ... you choose that, one thing at a time. Nothing is shared unless you share it."

**What flows without client action.** Session presence and counts: `dev.html:1797` ("N of your M clients have opened Alo"), `2043-2045` ("last active N days ago" / "hasn't opened Alo yet"), `2637-2648` ("Used Alo 3 times this week"), `3534` calendar check-in dots. Homework `viewed_at` and `completed_at`: `2613-2619`, `3322-3323`. Crisis alerts (screen 3 discloses these). The therapist's tour step 6 (`dev.html:4529`) even says "You'll see when it's marked done".

**Why it matters.** Claims-freeze item 5.15; this is the sentence a client reads before consenting. Copy Pack approval does not make it true.

**Fix.** Copy: "Nothing you say is shared unless you share it. Your therapist can see whether you've been using Alo and whether homework was done, never what you talked about." Or remove the presence surfaces.

### R-B-12 -- S2 -- Prep pack marks open alerts "resolved via ..." (R-B.8)

**What.**

```
dev.html:3232  const unresolvedCrisis = crisis.filter(c => !c.acknowledged);
dev.html:3268-3272  text += 'Safety: ' + unresolvedCrisis.map(c => ... 'Alert on ' + date + ': ' + c.category + '; resolved via ' + c.alo_response_action ...
```

`alo_response_action` is Alo's in-chat response (`provided_988_resources`), not a resolution, and the events in this loop are by construction the unacknowledged ones.

**Why it matters.** The Copy Prep Pack button (`dev.html:3280-3305`) exists to paste this into an EHR. The exported record states that an open crisis was resolved. The UI still shows it open, which is why this is S2 and not the S1 "misrendered as safe".

**Fix.** "Alert on <date>: <category>; Alo responded with <action>; NOT YET RESOLVED", and include resolved ones separately with the therapist's `action_taken`.

### R-B-13 -- S2 -- Tour claim about resolved alerts (R-B.8)

**What.** `dev.html:4566`: "Every alert, open or resolved, stays here with the action taken. The safety record never lives only in your memory." `renderCrisisAlerts` keeps resolved alerts only for 7 days: `dev.html:3417  recentResolved = crisis.filter(c => c.acknowledged && ... > weekAgo)`. After that they leave the card; the calendar day popup (`dev.html:3752`) still shows "<category> -- Resolved" without the action taken.

**Why it matters.** False as a storage claim per the rubric. Low harm: the row is in the database and partially reachable via the calendar. Triage may reasonably hold this at the copy level.

**Fix.** Either show all resolved alerts behind the existing `<details>` or change the sentence.

### R-B-14 -- S2 -- Home messages query and mark-read omit `therapist_id` (R-B.6)

**What.**

```
dev.html:1730  client_messages?client_id=in.(<ids>)&read=eq.false&revoked_at=is.null&order=created_at.desc&limit=50
dev.html:2085  client_messages?client_id=eq.${clientId}&read=eq.false   (PATCH read=true, read_by=therapistId)
dev.html:2437  client_messages?therapist_id=eq.${therapistId}&client_id=eq.${clientId}&revoked_at=is.null   (per-client, correct)
```

**Why it matters.** The schema allows a client to be linked to more than one therapist (`therapist_clients` is a link table). With two links, or with a permissive policy, therapist A's home feed shows messages the client wrote to therapist B, and opening the client marks B's messages read with A's id. The file already knows the right filter; it uses it in one of three places.

**Fix.** Add `therapist_id=eq.${therapistId}` to both.

### R-B-15 -- S2 -- Shared journal entries have no therapist target (R-B.6)

**What.** `dev.html:2374-2380`: `journal_entries?client_id=eq.X&shared_with_therapist=eq.true&...`. `shared_with_therapist` is a boolean.

**Why it matters.** "Shared with the therapist" cannot mean one therapist when the model permits several. Whether another therapist can reach the row is an RLS question; the dashboard adds no constraint of its own, and the consent copy (`dev.html:4441`) speaks of "your therapist", singular.

**Fix.** Schema-level (a `shared_with_therapist_id`, or a policy joining `therapist_clients`); dashboard-side, filter through the link. Studio work, flagged, not for A/S7 alone.

### R-B-16 -- S2 -- By-id mutations drop the available `therapist_id` guard (R-B.6)

**What.** `session_notes` PATCH/DELETE (`dev.html:2978`, `3016`) and `therapist_clients` PATCH (`4057`, `4116`, `4183`) filter on `id=eq.` only, although both tables carry `therapist_id` and the read paths already filter on it. POST bodies for `session_notes` (`2869-2871`, `3157-3159`), `homework_cards` (`3383-3385`) and `therapist_clients` (`3956-3963`) carry `therapist_id: therapistId` from JavaScript, which RLS must pin to `auth.uid()` for the value to mean anything.

**Why it matters.** Defense in depth only: with correct RLS none of this is exploitable. With a permissive UPDATE policy, a therapist who edits a URL edits another therapist's note or client. The brief rates this class S2.

**Fix.** Append `&therapist_id=eq.${therapistId}` to every by-id mutation on tables that have the column; keep the POST bodies but rely on RLS `with check` (verify in Studio).

### R-B-17 -- S2 -- Therapist-private fields live on the client-readable row (R-B.6, R-B.8)

**What.** `saveClientEdit` (`dev.html:4057-4068`) writes `safety_notes`, `phone`, `emergency_contact_name`, `emergency_contact_phone` and the `allow_*` toggles to `therapist_clients`. The client app reads its own row (`chat dev.html:1973`, `2059`) and PATCHes it (`chat dev.html:1385`); the RLS lockdown v1.2 records "client can PATCH fields on own link row (column-guard trigger later)" as an accepted residual. RLS is row-level; a client's JWT that can `select=id,...` can `select=*`.

**Why it matters.** `dev.html:682` promises "Never shared with client. For therapist reference only." That is true of the chat UI and untrue of the API a client can call from DevTools. A client can also, on the same row, flip `allow_homework` or `allow_messages` back on, or set `status`. Not exploitable in the UI; one crafted request away.

**Fix.** Move therapist-private fields to a therapist-only table (or restrict columns via a view or column grants) and add the column guard the lockdown plan already names. Schema change: Studio, Kano.

### R-B-18 -- S2 -- Invite code strength, no expiry, and an oracle (R-B.4)

**What.** `dev.html:3954`: `'ALO-' + Math.random().toString(36).substring(2, 8).toUpperCase()` -- six base-36 characters, about 31 bits, from a non-cryptographic generator. Combined with R-B-08 (never expires) and a claim endpoint that returns a distinguishable "Invalid invite code" (`chat dev.html:1454`), the space is enumerable in principle. The server-side seed uses `md5(random()...)` for the same format (`seed_therapist_account.sql:142`) and a unique index prevents duplicates (`20260902120400_invite_code_unique.sql`), so collisions are handled; guessing is not.

**Why it matters.** The code is the only secret on the link path. RLS finding S-03 (unconstrained claim policy) is the known, larger half of this, tracked in the RLS lockdown; the dashboard's contribution is the entropy and the missing expiry.

**Fix.** `crypto.getRandomValues`, 8+ characters, an expiry written at mint time and enforced by the claim policy; rate limiting on the claim is a Supabase-side item.

### R-B-19 -- S3 -- `therapists` keyed on the wrong column: first screen dead, timestamps never saved (R-B.5, R-B.6)

**What.**

```
dev.html:4276  apiMutate(`/rest/v1/therapists?id=eq.${therapistId}`, 'PATCH', { [field]: ... })
dev.html:4614  api(`/rest/v1/therapists?id=eq.${therapistId}&select=onboarding_completed_at,onboarding_skipped_at`)
```

`therapistId` is the auth uid (`dev.html:1346`). `therapists.id` is a random uuid: the seed inserts `(auth_user_id, profile_id, license_state, verify_deadline)` and lets `id` default (`seed_therapist_account.sql:205-211`; spec test 1; `BATCH_DAY_1_REPORT.md:98` "Do not set `id`. Let it default."). D22 fixed exactly this mismatch in `isVerificationLocked` (`dev.html:1489` now uses `auth_user_id`); these two calls were not repointed.

**Effect.** For every therapist-signup account (Dupre's will be one): `_aloMaybeAutoOfferTour` reads zero rows and never shows the first-screen overlay (P2B-03), and `_aloWriteOnboardingTimestamp` PATCHes zero rows and returns 204 (R-B-06 in the wild), so `onboarding_completed_at` / `onboarding_skipped_at` are never written. The designed first impression of the pilot does not happen; the tour remains reachable from Help.

**Why S3.** Functional, with a workaround; not a safety table. Demo-visible.

**Fix.** `auth_user_id=eq.${therapistId}` at both sites, and re-check the `therapists` UPDATE policy `with_check` (ledger notes it has no column restriction).

**Regression.** Same defect class as D22.

### R-B-20 -- S3 -- Rejected non-therapist session persists (R-B.1)

`completeSignIn` stores the session (`dev.html:1345-1347`) before the role check (`1349-1352`). On rejection the throw is caught in `handleLogin` (`1470-1472`), which shows the message and leaves `session`, `therapistId`, and `localStorage.alo_session` set. The next page load clears it (`1200-1221`). Not a bypass (`showApp` is unreachable), but a client's tokens sit in the dashboard's storage until then. Fix: clear all three in the failure branch.

### R-B-21 -- S3 -- Sign Out is local only (R-B.1)

`handleLogout` (`dev.html:1478-1483`) never calls `POST /auth/v1/logout`. The refresh token remains valid server-side; a copy taken before sign-out (shared device, extension, backup) still refreshes. For a clinical tool the server-side revocation is worth the one request. Fix: call logout with the access token, then clear locally.

### R-B-22 -- S3 -- Recovery-token handling (R-B.1, A7-D10)

The recovery `access_token` is read from the hash (`dev.html:863-867`), passed into `showPasswordResetModal`, and embedded in an inline attribute (`dev.html:914  onclick="submitPasswordReset('${accessToken}')"`). The hash is cleared only on success (`dev.html:958  window.location.hash = ''`); on failure or abandonment it stays in the URL, and the modal has no cancel or close control (`894-917`). The form accepts 6 characters (`906`, `934-935`) while signup requires 12, so the reset path is the weak-password path. The `prompt()` fallback (`872-887`) is unreachable in practice. A7-D10 status: partially addressed. Fix: clear the hash on modal open, keep the token in closure scope only, add cancel, align the minimum.

### R-B-23 -- S3 -- No request timeouts (R-B.5)

No `fetch` in the file has an `AbortController` (0 references). A connection that stalls without erroring never rejects: `setButtonLoading` never resets, `openAsClient` leaves its `about:blank` tab open (`dev.html:4370`) and the button on "Opening...". Fix: a shared timeout in `api()` and the direct `fetch` calls (10-15 s), with the existing catch paths.

### R-B-24 -- S3 -- Every toast looks like success (R-B.5)

The toast element carries a fixed checkmark SVG (`dev.html:827-832`), coloured green by `dev.css:1278`, and `toast()` (`dev.html:1583-1588`) has no variant. "Error saving note", "Session expired", "Could not open the test client" all render with the success icon. The brief asks whether any failure is rendered as success: visually, all of them. Fix: a `toast(msg, kind)` with an error style.

### R-B-25 -- S3 -- Shared-journal failure is silent (R-B.5)

`dev.html:2416-2419  catch (err) { console.error(...); card.style.display = 'none'; }`. A therapist looking for an entry a client explicitly shared sees "no card", indistinguishable from "nothing shared". Fix: show the card with an inline error and retry.

### R-B-26 -- S3 -- Parallel refreshes on concurrent 401s (R-B.1)

`loadDashboard` issues four requests in `Promise.all` (`dev.html:1728-1733`); when the access token has expired all four 401 together and each runs `refreshSession` with the same refresh token (`1123-1145`). Supabase rotates refresh tokens; whether the later calls succeed depends on the project's reuse interval. If they do not, `api()` forces a logout mid-load. Fix: a single in-flight refresh promise shared by callers.

### R-B-27 -- S3 -- Dates off by one (R-B.5)

`session_date` is a `date` column and arrives as `YYYY-MM-DD`; `new Date('2026-09-16')` is UTC midnight, and `toLocaleDateString` in a US zone prints the previous day. Confirmed: `TZ=America/Los_Angeles node -e ...` prints `Tuesday, September 15, 2026` for `2026-09-16`. Affected: `dev.html:2673` (Recent Session Notes card), `2723` (note card), `2743` (next session), `3292` (prep pack), `3552` and `3647`, `3700` (calendar placement). The default date for a new note is the UTC date (`dev.html:2807`, `3160`): after 5 pm Pacific a note is dated tomorrow. The seeded demo notes therefore show the wrong weekday in the demo. Fix: parse date-only strings as local (`new Date(y, m-1, d)`) and derive the default from local date parts.

### R-B-28 -- S3 -- Archived clients' crises never surface (R-B.2)

`dev.html:1682` fetches links with `status=in.(active,pending)`; `idFilter` (`1725-1729`) is built from them. A client the therapist archived can still be using Alo (archive changes only `therapist_clients.status`), and any crisis event they trigger is excluded from the home query while the greeting reads "All clear". Archive is explicit therapist intent and the copy says "Hides client from your dashboard" (`dev.html:741`), so this is a ruling, not a bug. Options: include archived links' events in the attention feed with an "archived" label, or state in the archive confirmation that alerts stop.

### R-B-29 -- S3 -- Lockout screen contact address (R-B.8)

`dev.html:1511` directs a locked-out therapist to `kano@alowen.ai`. Plan v1.5 lists "kano@alowen.ai inbox live" as a Session 8 item and a deferred track. If the inbox is not live when the first therapist is locked out, the only recovery path bounces. Unverifiable from here (section 6); S2 if still not live at pilot.

### R-B-30 -- S3 -- "Themes" in the prep pack (R-B.8)

`dev.html:3248-3249` writes `Themes: Client-reported focus areas: <primary_mode values>` where `primary_mode` is Alo's interaction mode (`VENT`, `UNDERSTAND`, `EXPAND`, `CLARIFY` per the seed). The first-screen and How-Alo-works copy promise "Not summaries, not themes, not 'insights'" (`dev.html:810`, `847`). The data is a mode label, not conversation content, so the promise holds in substance; the label contradicts it by name in a document meant for the EHR. Fix: rename the line ("Alo conversation modes:") or drop it.

### R-B-31 -- S3 -- Pending-client copy contradicts the guards (R-B.8)

`dev.html:2684`: "Client hasn't signed up yet. You can still write session notes and assign homework." `saveNote` (`2859-2862`), `saveQuickNote` (`3146-3149`) and `submitHomework` (`3372-3375`) all refuse with "available after client signs up". Fix: one of the two.

### R-B-32 -- S3 -- Zero-client branch leaves the page stale (R-B.5)

`dev.html:1692-1722`: hides every `.home-content .section`, prepends a welcome card, and returns. Every subsequent call with zero links prepends another card; the first call with one link (after "+ Invite Your First Client") skips the branch, renders into the hidden sections, and never removes the card or unhides them. Reachable only after archiving the seeded test client (seeded accounts always have at least one link). Fix: idempotent welcome card, and unhide on the non-empty path.

### R-B-33 -- S3 -- Accessibility (R-B.9)

Client rows (`dev.html:2049`), alert and message rows (`1830`, `1907`), triage rows (`1972`) and Client Pulse rows (`2562`) are `div`s with `onclick` and no `role`, `tabindex` or key handler: the client list cannot be opened from a keyboard. Calendar cells have `role="button" tabindex="0"` (`3603-3604`) but no `keydown` handler (0 in the file). The toast has no `aria-live`. Only the first-screen overlay has `role="dialog"`; every dynamically built modal and drawer has none. WCAG 2.1.1 / 4.1.3 defects.

### R-B-34 -- S4 -- Dead code (R-B.9)

- 14-day strip: `stripOffset` (`dev.html:3521`), the strip builder (`3623-3672`), `window._stripDays`, the `'strip'` branch of `showDayPopup` (`3709`), and the `calendarStrip` lookup (`3668`) -- no element with that id exists.
- Default client settings: modal markup (`751-799`), `showDefaultSettingsModal` (`4205-4208`, never called), `saveDefaultSettings` (`4210-4218`) writes `alo_default_settings` that nothing reads; `generateInvite` ignores it. The modal's own copy ("These defaults apply to all new clients") would be false if it were reachable.
- `therapistAvatar` lookup (`1540`): no such element.
- Reset `prompt()` fallback (`872-887`), reachable only if `showPasswordResetModal` throws synchronously.
- Unused locals: `body` (`965`), `result` (`3956`).
- `#inSessionMessagesCard` and `renderClientMessages` (`575-583`, `3913-3930`): D19, deferred deletion, listed for completeness.
- `triageSection` / `renderTriage` (`229-242`, `1961-1981`): hidden for pilot, still computed.
- `false_positive` column exists (seed) and is never read or written (0 references); the resolve modal has no false-positive outcome.
- `index.html` still carries `toggleDetailSidebar` and `navigateStrip` (`index.html:2270-2276`, `3289-3300`) deleted from dev.

### R-B-35 -- S4 -- Duplication (R-B.9)

Emergency panel markup twice (`dev.html:2130-2139`, `3446-3457`); anon key literal four times (`878`, `950`, `1032`, `1090`) because the reset IIFE runs before `const SUPABASE_ANON_KEY` initialises; first-screen copy twice (`809-811`, `846-848`); two full sidebars (`246-264`, `588-599`) rendered with the same HTML (`2079-2080`); `clientMeta` line built in two places (`2284-2286`, `4083-4085`).

### R-B-36 -- S4 -- Hard-coded colours and inline styles (R-B.9)

32 hex literals in JS and markup (`#3A7C86` x8, `#dc2626` x6, `#E8913A`, `#9CA3AF`, `#4A90D9` x5 each, `#fff3cd`, `#856404`, `#FFFFFF`) and large inline `style=` blocks (`dev.html:1508-1512`, `2034`, `2259`, `2262`, `2552`, `2556-2559`, `3534-3553`, `3721-3750`, `4158-4161`) against the "CSS custom properties only" rule. `!important` count is 10 in each stylesheet, unchanged by recent work.

### R-B-37 -- S4 -- `escJsAttr` applied inconsistently; self-XSS in forgot-password (R-B.9)

A7-D8's helper (`dev.html:1565`) is used for note ids (`2726-2742`) and the invite code (`2034`); client and event ids are interpolated raw into `onclick` attributes at `1828`, `1837`, `1865`, `1907`, `1932`, `1972`, `2164`, `2562`, `2916`, `3470`, `4163`. All are database uuids today, which is what the audit note at `1562-1564` says. `submitForgotPassword` (`1041`) interpolates the typed email into `innerHTML` unescaped: the user's own input, so self-XSS only.

### R-B-38 -- S4 -- Manifest promotion hazard; stale doc numbers (R-B.9)

`manifest.json` has `"start_url": "./dev.html"` and `dev.html:19` links it; the comment says dev-only. A promotion that copies line 19 into `index.html` ships an installable app whose start page is `dev.html`. The promotion parity checklist in `CLAUDE.md` does not cover this line. `CLAUDE.md` also states `index.html` at ~3,516 lines and `styles.css` at ~3,097; actual 3,920 and 3,215.

### R-B-39 -- S4 -- Copy nits (R-B.8)

"Add in Edit ->" (`dev.html:3115`) points at a modal titled "Client Settings" (`642`). The greeting counts the test client ("3 clients", `1774`). The missing-emergency-info banner fires on the test client (`3098` skips pending only). Client Pulse shows the raw `display_name` of the test client (`2565`) rather than "Your Test Client" (P2B-05). `openAsClient` toasts "Try again" (`4423`) for the permanent 403 legacy accounts receive under D9.

---

## 3. Angle-by-angle answers

### R-B.1 Auth and session

- **Where the session token lives and for how long.** The full GoTrue session object (access token, refresh token, user) is stored as JSON in `localStorage['alo_session']` at `dev.html:1347`, `1145`, `1198`, `4405`, and in the `session` global. It lives until Sign Out (`1481`) or a refresh rejection (`1162`). There is no idle timeout and no absolute lifetime; a refresh on every page load (`1194`) keeps it alive indefinitely. That is the A7-D9 Option A posture as described (localStorage plus CSP).
- **CSP.** Present in both files at lines 5-15. `default-src 'self'`; `script-src 'self' 'unsafe-inline'` plus `cdn.botpress.cloud`, `cdn.jsdelivr.net`, `files.bpcontent.cloud`, and (dev only) `cdnjs.cloudflare.com`; `style-src` likewise; `connect-src` limited to the Supabase project, `*.botpress.cloud`, `wss://*.botpress.cloud`, `files.bpcontent.cloud`; `img-src` includes `*.supabase.co` and `data:`/`blob:`; `object-src 'none'`; `base-uri 'self'`. What it does: blocks external scripts from other origins and limits where a script can send data. What it does not do: `'unsafe-inline'` means an injected inline script runs, so the CSP does not protect the localStorage token from an XSS on this page. Every user-supplied string in the file goes through `esc()` (`1555-1560`); I found no unescaped sink for third-party data (R-B-37 covers the one self-input case). Note the `img-src` and `connect-src` allowlists are exfiltration channels an injected script could still use.
- **Reset hash (A7-D10).** Cleared only on success at `958`; not on failure, not on abandonment, and there is no abandonment control. Token also sits in an inline attribute. See R-B-22.
- **D22 fail-closed lockout on every signup path.** Yes in `dev.html`: `isVerificationLocked` (`1485-1499`) returns locked on no row, on exception, and on an expired deadline, and is called from the restore path (`1223`) and from `completeSignIn` (`1354`), which both `handleLogin` and `handleSignup` use. The password-reset path never enters the app. In `index.html` the function is the pre-D22 version and fails open (R-B-04).
- **Can any path reach `loadDashboard()` without a verified session?** No. It is called only from `showApp()` and from in-app actions; `showApp()` is reached only after the password grant, the `profiles.role === 'therapist'` check, and the lockout check, in both entry paths. `api()` falls back to the anon key when `session` is null (`1112`), which under RLS returns nothing rather than someone else's data.
- **Token expiry mid-session.** `api()` retries once after a refresh (`1123-1160`); a network failure during refresh is retried twice with backoff (A7-D6, `1131-1140`); a real rejection clears storage, shows the login page, toasts, and throws (`1161-1167`). Fail closed: the app container is hidden, so no stale data is on screen. What is not cleared is the in-memory state and the DOM behind the login page (R-B-03), and concurrent refreshes are not serialised (R-B-26). `openAsClient` has its own one-shot refresh (`4395-4408`).
- **Other.** Sign Out does not revoke server-side (R-B-21); a rejected client login leaves its session behind (R-B-20); `signupErrorMessage` (`1297-1323`) never throws and never echoes the submitted value.

### R-B.2 Crisis-alert rendering (A-15)

- **Fields of `crisis_events` that reach the DOM.** Home list (`1825-1839`) and view-all modal (`1854-1867`): `severity`, `category`, `alo_response_action` (home only), `created_at`, `id` (attribute), `client_id` (attribute). Client card (`3460-3486`): `severity`, `category`, `details` (verbatim, `3467`), `alo_response_action`, `created_at`, `acknowledged_at`, `action_taken`, `action_notes` (`3483`, truncated to 80). Calendar popup (`3752`): `category`, `acknowledged`. Prep pack text (`3268-3272`): `category`, `alo_response_action`, `created_at`. Resolve modal (`2103-2172`): no event field; the emergency panel comes from the link row.
- **`raw_text`.** Not referenced anywhere. **Message excerpts.** Present in the data (`action_notes`, A-15) and rendered under the conditions in R-B-02. **`details`.** Rendered verbatim if present; existence unverifiable.
- **Is `acknowledged` the only resolution state?** Yes. Both queries and every filter use `acknowledged`; `false_positive` exists in the schema and is never touched.
- **Can an alert be dismissed without a resolution record?** No. The only write is `submitResolveCrisis` (`2174-2217`), which requires an `action_taken` selection client-side and writes `acknowledged`, `acknowledged_at`, `acknowledged_by`, `action_taken`, `action_notes` in one PATCH. Closing the modal or the view-all overlay changes nothing. (Whether the server enforces `action_taken` is an RLS/constraint question; not visible here.)
- **Is the Needs Attention count derived from the same query as the list?** Yes: `renderAlerts` sets `#alertsCount` from `crisisEvents.length` and renders the same array (`1813-1846`); `updateAttentionCard` (`2495-2516`) reads that count plus the messages count; the greeting uses the same array (`1770`). No drift between the count and the list. Two related drifts do exist: the count includes null-client rows that map to no client, so "2 alerts" can coexist with no CRISIS client in the sidebar; and the client page's count comes from the unlimited per-client query while the sidebar pill comes from the capped home query (R-B-01).
- **`goToCrisisAlerts`** (`3503-3513`) scrolls to whichever card is on the active tab; no data involved.
- **Archived clients.** Their events never surface (R-B-28).

### R-B.3 Scoping: the null-client query

Answered in R-B-01 and R-B-07. In one paragraph: every therapist receives every unacknowledged guest event, rendered as "Unknown Client" with Resolve; a second therapist sees exactly the same rows and either can acknowledge them; the only written policy text (vault, stale) would filter NULL rows out and make the branch dead, while the RLS lockdown record suggests policies on this table are loose; the 50-row cap does let guest rows push a therapist's own clients' open alerts off the list (S1). Recommended rule: therapist queries filter on the therapist's own client ids only; null-client events go to an operations surface under the service role; the RLS SELECT policy is written to the same predicate.

### R-B.4 Invite and impersonation paths

- **Invite entropy and expiry.** About 31 bits from `Math.random` (`3954`); no expiry anywhere (R-B-08, R-B-18). The server-side seed generates the same format with `md5(random()...)`.
- **Can an invite be re-claimed?** Through the chat app, no: the claim reads `status=eq.pending` (`chat dev.html:1451`) and sets `status: 'active'` (`1461-1464`), and `invite_code` is unique. Through the API it depends on the UPDATE policy: the RLS lockdown v1.2 replaces an "unconstrained claim policy" (S-03) with `client_claim_pending_link` (`using status='pending' and client_id is null`); whether Phase 1 has run is unknown (section 6). Until it has, S-03 stands and an active link can be re-pointed by any authenticated user. The dashboard cannot detect a link whose `client_id` changed underneath it.
- **Invite delivery.** `_aloInviteMessage` (`3997-3999`), `_aloCopyInviteLink` (`4001-4006`), `_aloTextInvite` (`4008-4023`, iOS `&body=`, Android `?body=`, clipboard otherwise), `_aloEmailInvite` (`4025-4029`), `copyInviteCode` (`3990-3993`): clipboard and OS handlers only, per D28; the link is `https://alowen.ai/?invite=<code>` and the chat app prefills it (`chat dev.html:3783-3797`). `copyInviteCode` has no `.catch` on the clipboard promise (rejection is silent); `_aloCopyInviteLink` has one. The message text's "Nothing you say there comes to me unless you choose to share it" is true of what the dashboard displays (see R-B-02 for the storage caveat) and narrower than the screen-2 claim in R-B-11.
- **`openAsClient` (`4362-4427`) and `_aloOpenTestClient` (`4332-4352`).** Can it be invoked for a client the therapist does not own? No: the request carries no client id; `open-as-client` (`alo-supabase`, `index.ts` step 2) selects `therapist_clients` where `therapist_id = auth.getUser(jwt).id and is_demo and client_id is not null` and 403s unless exactly one row matches. The button is rendered only for `is_demo` rows (`2062-2063`, `2277-2278`), but the function would be safe from anywhere. Token lifetime and single use: 32 CSPRNG bytes, base64url; only the SHA-256 is stored; `expires_at = now + 60 s` from the Edge runtime clock; consumed by one compare-and-swap `UPDATE ... where used_at is null and expires_at > now` (`exchange-client-token/index.ts`), so a replay and a race loser both match zero rows and receive the same 401. What the client app receives: a URL with `?t=`, stripped immediately by `history.replaceState` (`chat dev.html:3714-3725`), exchanged once for a GoTrue magic-link `token_hash`, then `verifyOtp` establishes a normal, persistent session for the test client's auth user in the chat origin (`3742-3778`). That session outlives the 60-second token by design; signing out of it is the chat app's normal path. Failure shows one constant message (D31). Dashboard side: the tab is opened before the await (pop-up blockers), `tab.opener = null`, `location.replace` so the token URL is not a history entry, generic toast, the URL is used verbatim (D29 forbids constructing it). Nothing on this path lets a therapist act as a client they do not own or replay a token. Minor: the blank tab on a stalled request (R-B-23); "Try again" on the D9 403 (R-B-39); the returned URL is trusted verbatim, which is the ruled design.
- **Legacy accounts.** `_aloOpenTestClient` picks the first `is_demo` row; trigger-v3 accounts have two, both now labelled "Your Test Client" (P2B-05), and `open-as-client` 403s them (D9). Cosmetic.

### R-B.5 Error handling and silent failure

- **Non-2xx.** `api()` throws `API <status>: <body>` (`1170-1173`) and every caller catches; the direct `fetch` calls (`947`, `1029`, `1330`, `1413`, `4381`) check `ok`/status. Error bodies are logged to the console in `api()`'s message, which for PostgREST errors can contain the filter values (uuids) but not user content.
- **Network failure.** `fetch` rejects; callers toast or show inline text. Stalls never reject (R-B-23).
- **204 with zero rows.** Undetectable everywhere (R-B-06); one confirmed live instance (R-B-19).
- **A-07 root cause.** Confirmed and closed. The old `handleSignup` posted to `/auth/v1/signup` and then made two client-side inserts whose responses were never checked ("fetch does not reject on 4xx, so both failed silently and the UI called showApp() anyway", `dev.html:1366-1371`; `R1_REPORT.md` section 3.2), and its generic error path hid the validator's field errors. R1 replaced it: one POST to `therapist-signup` (`1413`), `res.status !== 201` is the only success (`1430-1432`), `signupErrorMessage` (`1297-1323`) surfaces the validator's `message` verbatim, `completeSignIn` re-checks the role, and `showApp()` is unreachable otherwise. Present in `index.html` too (`index.html:1199-1349`). The plan's item 1.5 (whether a verification email ever arrived) is out of this review's scope.
- **`setButtonLoading` / `toast`.** `setButtonLoading` (`1072-1084`) is symmetric and used on every mutation in dev (SILENT-4); not in `index.html` (section 5). `toast` renders every message as a success (R-B-24).
- **Silent failures found.** `loadDashboard` (R-B-05, the important one), `loadSharedJournal` (R-B-25), `_aloWriteOnboardingTimestamp` (`4278`, by design, and now known to be a zero-row PATCH: R-B-19), `_aloMaybeAutoOfferTour` (`4619`, by design), clipboard promises without `.catch` (`3084`, `3208`, `3298`, `3992`). Writes on `crisis_events`, `therapist_clients`: no unconditional silent failure; the conditional one is R-B-06. `client_settings`: the dashboard never writes that table.

### R-B.6 Data scoping

Section 4. Defense-in-depth gaps: R-B-14, R-B-15, R-B-16, R-B-17, plus the null branch (R-B-07) and the wrong-key `therapists` calls (R-B-19).

### R-B.7 A7 status re-verification

| Item | dev.html | index.html | Status |
|---|---|---|---|
| D2 generation guard in `showClient` | `1102-1106`, `2226-2228`, `2318-2328`, `2353-2355` | `1004-1008`, `2017-2019`, `2105-2115`, `2140-2142` | Present in both; `showHomePage` bumps the counter; `loadClientData` failures respect it |
| D3 `loadDashboard` awaited with error handling | `3970-3981`, `4124-4134`, `4189-4197` | `3728-3737`, `3831-3839`, `3890-3896` | Present in both; **ineffective** in both (R-B-05) |
| D4 `deleteNote` awaits DELETE | `3006-3030` | `2784-2800` | Present in both |
| D5 mark-read failure surfaced | `2083-2101` | `1895-1900` | Present in both |
| D6 refresh network/auth distinction with backoff | `1122-1141`, `1255-1277` | `1024-1043`, `1157-1179` | Present in both |
| D7 `allSettled` in `loadClientData` | `2429-2459` | `2216-2246` | Present in both |
| D8 `escJsAttr` | `1562-1567` | `1461-1466` | Present in both; applied to note ids only (R-B-37) |
| D9 localStorage + CSP posture | `5-15`, `1347` | `5-15`, `1249` | As ruled; see caveat in R-B.1 |
| D10 reset hash cleared | `958` | `862` | Partial (R-B-22) |
| D12 | cited at `2095` as closed with D5 | `1895` | Closed |
| D11, D13-D17 | -- | -- | **Cannot be listed by number**: the audit and tracker are not on this machine and D36 is not in any local ledger copy. Of this review's findings, the ones that read as audit-class items are R-B-05, R-B-06, R-B-20, R-B-21, R-B-22, R-B-23, R-B-26 |

### R-B.8 Demo copy (claims-freeze, item 5.15)

Every therapist-visible sentence that makes a claim about privacy, storage, what clients see, or what Alo does. TRUE / FALSE / UNVERIFIABLE on this build. Functional copy that is wrong but not a claim in that sense is listed as S3 in section 2 (R-B-30, R-B-31, R-B-39).

| Where | Sentence (quoted) | Verdict | Note |
|---|---|---|---|
| Login `dev.html:37` | "Client data is encrypted at rest and in transit." | TRUE (transit) / UNVERIFIABLE (rest) | HTTPS; at-rest is a Supabase platform property |
| Login `37` | "Alowen conversations are not permanently saved -- chat history is automatically cleared." | **FALSE** | R-B-10; A-23 |
| Login `38` | "Alowen supports between-session reflection. It is not emergency care or a crisis service. In a crisis, follow your clinical protocol." | TRUE | -- |
| Invite modal `616` | "Friendly name visible only to you" | **FALSE** | R-B-09 |
| Invite modal `620`, `622` | "Share this code with your client" / "Expires in 7 days . Click code to copy" | TRUE / **FALSE** | R-B-08 |
| Invite message `3998` | "Nothing you say there comes to me unless you choose to share it." | TRUE (display) / caveat | Storage caveat R-B-02 |
| Client settings `653` | "Used for session prep and Alo context" (next session date) | UNVERIFIABLE | Whether Alo reads `next_session_date` is bot-side |
| Client settings `665` | "Displayed only during crisis alerts for immediate action" (emergency contact) | TRUE | Also displayed in this modal and for recently-resolved-only state (`3424-3432`); minor |
| Client settings `682` | "Never shared with client. For therapist reference only." (safety notes) | TRUE (UI) / UNVERIFIABLE (API) | R-B-17 |
| Client settings `695`, `705`, `715`, `725` | "Client can see assigned homework in chat" / "Client can send you async messages" / "Client can view their own past sessions" / "Client can share individual journal entries with you" | TRUE | Chat honours `allow_homework` (1056), `allow_messages` (902), `allow_history` (2065), `allow_journal_sharing` (3427) |
| Client settings `733` | "These settings control what features the client sees in their chat interface. Changes apply on client's next session." | TRUE | Read at chat load (`2059`) |
| Client settings `741` | "Hides client from your dashboard. Their notes and data are preserved. Restore anytime from the bottom of the Clients sidebar." | TRUE | Restore link is on the home sidebar only (`261`); crisis alerts stop (R-B-28) |
| Default settings `759`, `791` | "These defaults apply to all new clients." | FALSE but unreachable | R-B-34; not therapist-visible today |
| How Alo works / first screen `809`, `846` | "When a client chooses to share something -- a journal entry, a message, a summary -- it shows up here." | TRUE (journal, message) / UNVERIFIABLE (summary) | No summary surface in the dashboard; `allow_summary` unused (0 refs) |
| same | "Homework you assign, and whether it got done. A safety alert if Alo ever detects one." | TRUE | -- |
| `810`, `847` | "What you won't see. Your clients' conversations with Alo. Not summaries, not themes, not 'insights.'" | TRUE (substance) | Prep pack "Themes:" label contradicts by name (R-B-30); A-15 storage caveat (R-B-02) |
| `811`, `848` | "About fifteen minutes a week." | UNVERIFIABLE | Not a build property |
| Welcome card `1710-1712` | "They connect via Alowen, and you see their activity here" | TRUE | -- |
| Greeting `1780-1784` | "All clear. N clients." / "-- alerts need your attention." | TRUE | Counts guest events (R-B-07) and the test client (R-B-39) |
| Rollout `1797` | "N of your M clients have opened Alo. K haven't yet." | TRUE with D30 caveat | 100-row window (D30 accepted) |
| Client row `2043`, `2045` | "last active N days ago" / "hasn't opened Alo yet." | TRUE with D30 caveat | Same |
| Test-client line `2062`, `2277` | "This is your own test account. Open it as a client to see exactly what yours will see. Nothing here is a real person." | TRUE | Legacy accounts 403 on open (D9) |
| Pending screen `1511` | "This usually takes less than 10 days -- reach out to kano@alowen.ai" | TRUE (10 days = `verify_deadline`) / UNVERIFIABLE (inbox) | R-B-29 |
| Session prep `2606-2663` | "N unresolved alerts" / "Used Alo N times this week" / "No Alo activity yet" / "N unread messages" | TRUE | Derived from fetched rows |
| Session prep `2684` | "Client hasn't signed up yet. You can still write session notes and assign homework." | FALSE (functional) | R-B-31 |
| Prep pack `3245` | "Source: Client self-directed use of an emotional support tool. No session transcript available." | TRUE | -- |
| Prep pack `3249` | "Themes: Client-reported focus areas: ..." | misleading | R-B-30 |
| Prep pack `3271` | "Alert on <date>: <category>; resolved via <action>" | **FALSE** | R-B-12 |
| Prep pack `3058`, `3081`, `3204` | "Based on Alowen between-session data -- review and edit before adding to client record." / medical-necessity boilerplate | TRUE / UNVERIFIABLE | Boilerplate is a clinical assertion the therapist opts into |
| What clients see `4437` | "Alo asks more than it tells. It isn't a therapist, and it isn't trying to be one." | TRUE (design) | -- |
| What clients see `4441` | "Your conversations with Alo stay between you and Alo. Your therapist can't read them." | TRUE (dashboard) / caveat | R-B-02 storage caveat |
| What clients see `4441` | "Nothing is shared unless you share it." | **FALSE** | R-B-11 |
| What clients see `4445` | "If Alo notices you might be in trouble ... it will let your therapist know something happened. Not what you said. Just that you might need them." | TRUE (display) / **FALSE (storage)** | A-15; R-B-02 |
| What clients see `4449` | "Before their first conversation, clients confirm: 'I understand what Alo is and isn't, and how sharing works.' Then they tap 'Start talking to Alo.'" | TRUE | `chat dev.html:1344-1354` |
| Tour `4474` | "Your home page, every time you sign in." | FALSE after R-B-03 | Same-tab re-login lands on the client page |
| Tour `4482`, `4495`, `4509`, `4549`, `4575` | Needs Attention first; most urgent first; everything in one place; settings; emergency contact one click away | TRUE | -- |
| Tour `4521` | "Yours only -- client conversations with Alo never appear here or anywhere." | TRUE (dashboard) | -- |
| Tour `4529` | "You'll see when it's marked done -- not what they said about it." | TRUE | -- |
| Tour `4541` | "Only what they explicitly shared -- never their chat history." | TRUE | -- |
| Tour `4566` | "Every alert, open or resolved, stays here with the action taken." | **FALSE** after 7 days | R-B-13 |
| Empty states `216`, `1817`, `1895`, `2754`, `3314` | "All clear -- no items needing attention." etc. | TRUE | Subject to R-B-01 and R-B-28: "all clear" can be shown while an alert exists off-list |
| Toasts | "Crisis alert resolved", "Client updated", "Note saved", ... | TRUE when the PATCH matched a row | R-B-06 |
| Toast `1166` | "Session expired. Please log in again." | TRUE | -- |
| Install banner `836` | "Add this dashboard to your home screen for one-tap access." | TRUE (dev) | Start page would be dev.html (R-B-38) |

### R-B.9 Dev/prod parity and hygiene

Section 5. Zero-width scan CLEAN on all four files. Dead code and duplication: R-B-34, R-B-35. Hard-coded colours: R-B-36.

---

## 4. Query scoping table (R-B.6)

Every Supabase request in `dev.html`, in file order. "Client filter" = what the URL constrains. "Relies on" = what keeps another therapist's rows out: the client-side filter, RLS, or both. No policy text for these tables is in any repo; where the ledger or the RLS lockdown record says something, it is cited. "Gap" = a forged or missing filter would return another therapist's rows if RLS were permissive.

| # | Line | Endpoint / table | Method | Client filter | Relies on | Gap / note |
|---|---|---|---|---|---|---|
| 1 | 875 | `/auth/v1/user` | PUT | recovery token | GoTrue | Unreachable fallback (R-B-34) |
| 2 | 947 | `/auth/v1/user` | PUT | recovery token | GoTrue | Password reset |
| 3 | 1029 | `/auth/v1/recover` | POST | email | GoTrue | Anon key |
| 4 | 1202 | `profiles` | GET | `id=eq.<uid>&select=role` | both | Ledger: SELECT own row |
| 5 | 1257 | `/auth/v1/token?grant_type=refresh_token` | POST | -- | GoTrue | -- |
| 6 | 1330 | `/auth/v1/token?grant_type=password` | POST | -- | GoTrue | -- |
| 7 | 1349 | `profiles` | GET | `id=eq.<uid>&select=role` | both | As 4 |
| 8 | 1413 | `functions/v1/therapist-signup` | POST | body | server | Anon key; CORS allowlist |
| 9 | 1489 | `therapists` | GET | `auth_user_id=eq.<uid>&select=onboarded,verify_deadline` | both | Ledger: `therapists` policies granted to `{public}`; scope unknown |
| 10 | 1532 | `profiles` | GET | `id=eq.<uid>&select=display_name,email` | both | As 4 |
| 11 | 1682 | `therapist_clients` | GET | `therapist_id=eq.<uid>&status=in.(active,pending)&select=*` | both | Gap if RLS permissive; returns `invite_code`, `safety_notes`, contacts |
| 12 | 1729 | `crisis_events` | GET | `or=(client_id.in.(<ids>),client_id.is.null)&acknowledged=eq.false&limit=50` | client filter + RLS (unknown) | **Gap**: null branch (R-B-07); cap (R-B-01); `select=*` over-fetch (R-B-02); no therapist column |
| 13 | 1730 | `client_messages` | GET | `client_id=in.(<ids>)&read=eq.false&revoked_at=is.null&limit=50` | client filter + RLS | **Gap**: no `therapist_id` (R-B-14) |
| 14 | 1731 | `homework_cards` | GET | `therapist_id=eq.<uid>&is_active=eq.true&select=id,client_id` | both | -- |
| 15 | 1732 | `session_metadata` | GET | `client_id=in.(<ids>)&limit=100` | client filter + RLS | No therapist column exists; D30 cap |
| 16 | 2085 | `client_messages` | PATCH | `client_id=eq.<id>&read=eq.false` | RLS | **Gap**: no `therapist_id` (R-B-14); `return=minimal` |
| 17 | 2192 | `crisis_events` | PATCH | `id=eq.<event>` | RLS | By id only; no therapist column; `return=minimal` (R-B-06) |
| 18 | 2375 | `journal_entries` | GET | `client_id=eq.<id>&shared_with_therapist=eq.true&select=id,title,content,source,created_at,entry_type` | client filter + RLS | **Gap**: no therapist target (R-B-15) |
| 19 | 2435 | `session_notes` | GET | `therapist_id=eq.<uid>&client_id=eq.<id>` | both | -- |
| 20 | 2436 | `homework_cards` | GET | `therapist_id=eq.<uid>&client_id=eq.<id>` | both | -- |
| 21 | 2437 | `client_messages` | GET | `therapist_id=eq.<uid>&client_id=eq.<id>&revoked_at=is.null` | both | Correct form |
| 22 | 2438 | `session_metadata` | GET | `client_id=eq.<id>` | client filter + RLS | No therapist column |
| 23 | 2439 | `crisis_events` | GET | `client_id=eq.<id>` (no ack filter, no limit) | client filter + RLS | `select=*` over-fetch (R-B-02) |
| 24 | 2869 | `session_notes` | POST | body `therapist_id`, `client_id` | RLS `with check` | Client-supplied `therapist_id` (R-B-16) |
| 25 | 2978 | `session_notes` | PATCH | `id=eq.<note>` | RLS | **Gap**: `therapist_id` available, omitted (R-B-16) |
| 26 | 3016 | `session_notes` | DELETE | `id=eq.<note>` | RLS | Same |
| 27 | 3157 | `session_notes` | POST | body | RLS `with check` | As 24 |
| 28 | 3383 | `homework_cards` | POST | body `therapist_id`, `client_id` | RLS `with check` | As 24 |
| 29 | 3956 | `therapist_clients` | POST | body `therapist_id`, `invite_code`, `status: pending` | RLS `with check` | Code from `Math.random` (R-B-18) |
| 30 | 4057 | `therapist_clients` | PATCH | `id=eq.<link>` | RLS | **Gap**: `therapist_id` omitted (R-B-16); writes private fields (R-B-17) |
| 31 | 4116 | `therapist_clients` | PATCH | `id=eq.<link>` (archive) | RLS | Same |
| 32 | 4145 | `therapist_clients` | GET | `therapist_id=eq.<uid>&status=eq.archived&select=*` | both | -- |
| 33 | 4183 | `therapist_clients` | PATCH | `id=eq.<link>` (restore) | RLS | As 30 |
| 34 | 4276 | `therapists` | PATCH | `id=eq.<uid>` | -- | **Wrong key**: matches zero rows for therapist-signup accounts (R-B-19) |
| 35 | 4339 | `therapist_clients` | GET | `therapist_id=eq.<uid>&status=eq.archived&is_demo=eq.true&select=*` | both | -- |
| 36 | 4381 | `functions/v1/open-as-client` | POST | user JWT, empty body | server | Scoped server-side to the caller's own `is_demo` link |
| 37 | 4614 | `therapists` | GET | `id=eq.<uid>&select=onboarding_completed_at,onboarding_skipped_at` | -- | **Wrong key** (R-B-19) |

37 calls (the brief estimated ~30). Tables touched: `profiles`, `therapists`, `therapist_clients`, `crisis_events`, `client_messages`, `homework_cards`, `session_metadata`, `session_notes`, `journal_entries`. Not touched by the dashboard: `client_settings`, `conversations`, `conversation_messages`, `conversation_summaries`, `client_session_tokens`. The dashboard never inserts `homework_status` or `homework_engagement` rows (answers the RLS lockdown v1.2 Phase 2 question for the dashboard side).

---

## 5. Parity diff (R-B.9)

`dev.html` 4,625 lines vs `index.html` 3,920; `dev.css` 3,427 vs `styles.css` 3,215. `index.html` references `styles.css?v=29`, `dev.html` references `dev.css?v=39`. Every difference is `dev` ahead of `index`; nothing in `index` is newer than `dev`. Classification: **intentional dev-only** (never meant to promote as-is), **missing promotion** (finished work on `main` not yet copied to production), **drift** (unexplained divergence). There is no drift.

### 5.1 `dev.html` vs `index.html` (hunks in file order)

| Hunk (dev lines) | What | Class | Belongs to |
|---|---|---|---|
| 7-8 | CSP adds `cdnjs.cloudflare.com` to `script-src`/`style-src` | missing promotion (tied to the tour) | Phase 2 item 9 |
| 18-23 | manifest link, theme-color, apple-touch-icon, favicons | intentional dev-only per the comment; **hazard** if copied (R-B-38) | Phase 2 item 13a |
| 24 | `dev.css?v=39` vs `styles.css?v=29` | expected | -- |
| 25-28, 857-859 | driver.js CSS and JS with SRI | missing promotion | Phase 2 item 9 |
| 167-178 | Help menu | missing promotion | P2B-04 |
| 193 | `#rolloutSummary` | missing promotion | P2B-06 |
| 431, 740, 746, 2164 | button ids for `setButtonLoading` | missing promotion | Phase 3 SILENT-4 |
| 563-579 | SILENT-3 comment and `#viewAllMessagesToolsBtn` | missing promotion | Phase 3 SILENT-3 |
| 622-627 | "Click code to copy" and the three send buttons | missing promotion | P2B-07 |
| 801-825 | How-Alo-works and What-clients-see modals | missing promotion | P2B-04 |
| 832-853 | install banner, first-screen overlay | missing promotion | Phase 2 13a, P2B-03 |
| 1004-1005, 1602-1676, 2170-2171, 2789-2790, 2853-2854, 2966-2967, 3764-3765, 3803-3804, 3842-3843, 3937, 4046, 4207 | `trapFocus`, `addEscToClose`, `wireEscToStaticModal`, per-modal releases | missing promotion | Phase 3 COSMETIC-1/-2 |
| 1485-1497 | D22 fail-closed `isVerificationLocked` | **missing promotion; production fails open (R-B-04)** | P2B-01 / D22 |
| 1549 | `_aloMaybeAutoOfferTour()` call | missing promotion | Phase 2 item 9 |
| 1787-1803 | rollout summary logic | missing promotion | P2B-06 |
| 2002-2006, 2037-2045, 2056, 2246-2250, 2267-2278 | "Your Test Client" label, activity meta line, test-client line and banner | missing promotion | P2B-05, P2B-06 |
| 2062-2063, 2277-2278 | "Open as client" buttons | missing promotion | D29-01 |
| 2186-2190, 2214-2215, 2865-2867, 2893-2894, 2973-2976, 3001-3002, 3006-3014, 3028, 3152-3155, 3176-3177, 3378-3381, 3405-3406, 3946-3952, 3985-3986, 4052-4055, 4095-4096, 4109-4114, 4138-4139, 4163, 4177-4181, 4201 | `setButtonLoading` on all mutation handlers; `deleteNote`/`restoreClient` take the button | missing promotion | Phase 3 SILENT-4 |
| 2734 | `deleteNote(..., this)` | missing promotion | Phase 3 SILENT-4 |
| index 2270-2276 (`toggleDetailSidebar`), index 3289-3300 (`navigateStrip`) | dead functions removed in dev, still in prod | missing promotion of a deletion | Phase 3 SILENT-2 / COSMETIC-3 |
| 3993-4030 | `_aloInviteMessage`, `_aloCopyInviteLink`, `_aloTextInvite`, `_aloEmailInvite` | missing promotion | P2B-07 / D28 |
| 4219-4622 | install prompt, tour, help menu, `_aloOpenTestClient`, `openAsClient`, onboarding screens constant, tour steps, auto-offer | missing promotion | Phase 2, P2B-03/04, D29-01 |

Production therefore lacks: the fail-closed lockout, every double-submit guard, the focus traps and Escape handling, the tour, the help menu and its two modals, the first-screen overlay, the test-client labelling, the rollout surfaces, the invite send actions, and Open-as-client. It retains two dead functions. The R1 signup rewire and the A7 D2/D4-D8 fixes are in both.

### 5.2 `dev.css` vs `styles.css`

All seven hunks are additions in dev; nothing was removed or changed.

| Hunk (dev lines) | What | Class | Belongs to |
|---|---|---|---|
| 294-370 | `.test-client-line`, `.open-as-client-btn`, `.test-client-line-detail`, help menu | missing promotion | P2B-05, D29, P2B-04 |
| 902-908 | `.rollout-summary` | missing promotion | P2B-06 |
| 1152-1188 | first-screen overlay | missing promotion | P2B-03 |
| 1356-1362 | `.invite-send-actions` | missing promotion | P2B-07 |
| 2176 | mobile help-menu button | missing promotion | P2B-04 |
| 3344-3360, 3362-3427 | install banner; driver.js theme | missing promotion | Phase 2 13a, item 9 |

When promoted, `styles.css` needs `?v=30` (or higher) in `index.html` per the parity checklist; the manifest link (dev line 19) must not be copied unless `manifest.json` `start_url` is changed first (R-B-38).

### 5.3 Hygiene

- `python3 scripts/scan-invisible.py dev.html dev.css index.html styles.css`: all four CLEAN, exit 0.
- `node --check` on the single inline script of each HTML file: both pass.
- `!important`: 10 in each stylesheet.
- Anon key literal: 4 occurrences in each HTML file (R-B-35). No `service_role`, no `sk-ant`, no other JWT.
- CI (`.github/workflows/ci.yml`) checks CSS backticks, smart quotes, NBSP, brace balance, W3C validity, and HTML backtick fences only. It does not run the zero-width scan on HTML, and does not check the stylesheet reference after a promotion.

---

## 6. What could not be verified without live-DB or Studio access

1. **RLS policies on `crisis_events`, `therapist_clients`, `client_messages`, `journal_entries`, `session_metadata`, `session_notes`, `homework_cards`, `therapists`.** No `create policy` for any of them exists in a repo. The Sep 12 `pg_policies` CSV export the RLS lockdown cites is not local. R-B-07's "what RLS would filter" is argued from the vault's stale summary and the lockdown record, not from live policy text. Whether RLS lockdown v1.2 Phase 1 or Phase 2 has run is unknown; the record's STATUS lines are unchecked.
2. **Whether a `details` column exists on `crisis_events` and what Botpress writes to it.** `dev.html:3467` renders it verbatim. The seed defines no such column; the Bot Logic Review names `action_notes` as the excerpt carrier.
3. **Whether Kano's legacy account (`82bc478b`, trigger-v3) has `therapists.id = auth uid`.** R-B-19 is certain for therapist-signup accounts (the seed) and undetermined for legacy rows created by the old dashboard insert.
4. **Whether `kano@alowen.ai` is live** (R-B-29).
5. **Conversation retention and the pg_cron sweep's scope** behind the login-page sentence (R-B-10). The A-23 evidence is documentary.
6. **Whether `exchange-client-token` has been redeployed with the D32 CORS fix.** If not, `openAsClient` fails end to end from the chat origin with the constant D31 message; the dashboard side is unaffected.
7. **Whether the server enforces `action_taken` on acknowledgement**, and whether `acknowledged_by` is pinned to `auth.uid()` (R-B-07).
8. **The `profiles` role gate for legacy accounts** relies on `profiles.role`; the ledger records that check as the only server-side capability gate. Not re-tested here.
9. **Runtime rendering.** Headless Chrome (`--headless --dump-dom`) timed out on both pages twice; the console log it did emit shows no `Uncaught` and no CSP `Refused` at load, only Chrome's "Password field is not contained in a form" notice (which is itself a minor a11y/autofill nit at `dev.html:43-52`). No signed-in state was exercised: the brief forbids live calls beyond what the page performs on load, and a fresh profile has no session.
10. **Ledger D36 and D38, the March 31 audit, and Plan v1.9.2** are not on this machine (Session Log, deviations). The A7 D11-D17 list is therefore unmatched by number.

---

*R-B v1.0 -- 2026-09-16. Findings only. Fixes are A/S7's after triage. No app file was changed. Branch `auto/r-b-review`, one file.*
