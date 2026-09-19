# S7 dash fixes session report v1.0

**Repo:** `kanomelvin-prog/alo-dashboard` -- branch `auto/s7-dash-fixes` off `main` @ `7e7bb95`.
**Date:** 2026-09-19. **Brief:** `claude_Brief_3_Dash_Fixes_v1_0.md` (Brief #3-dash). **Findings source:** `auto/r-b-review:docs/reports/R-B_dashboard_review_v1_0.md`.
**Files changed:** `dev.html`, `dev.css` (one block, for fix D), this report. `index.html`, `styles.css`, `manifest.json` and `CLAUDE.md` are byte-identical to `main`. Nothing was committed or pushed to `main`.

**No LIVE CHANGE in this session.** No promotion, no push to `main`, no SQL, no Edge Function, no auth or Supabase configuration change, and no request of any kind to the live Supabase project. GitHub Pages publishes `main`; this branch is not served.

---

## 0. Session log and deviations

1. **Model.** Claude Fable 5.1 (`claude-fable-5-1`), effort max.
2. **Start.** The launch message consisted only of pasted text. Its instruction (read and execute this brief on a new branch, no push to `main`, commit the report) was confirmed with Kano before anything was read or changed. Branch created from `main` before any file was touched; local `main` equalled `origin/main`.
3. **Read first, in order, as the brief requires:** `CLAUDE.md`; the R-B report in full (701 lines); `dev.html` in full (4,625 lines, every line). Also the three vault files `CLAUDE.md` names for session start.
4. **Interruptions and restarts.** None. No context compaction, no retried edit, no reverted commit.
5. **Deviations from the brief, each with its reason.**
   - **`SIGNED_OUT` event (fix C).** The brief lists it as a sign-out path. This file has no SDK, no `onAuthStateChange` and no `storage` listener (0 references), so there is no such path to reset. The three sign-out paths that do exist all go through one function now. Propagating a sign-out to *other open tabs* would be new behaviour, not a fix to an existing path, and it is auth-adjacent (`CLAUDE.md`: no auth changes without approval), so it was not added. It needs a ruling: section 8.2.
   - **Prep pack dash (fix I).** The brief writes `"Alert on <date>: <category> — open"` with an em dash. Every other line of the prep pack and of the EHR copy functions is ASCII (`' -- Status: '`, `'SESSION PREP PACK -- '`), which looks deliberate for text pasted into an EHR. The new lines use `' -- open'` and `' -- resolved via '`. The brief allows "the existing phrasing minus 'resolved via'", so this is inside its latitude; changing it to an em dash is a two-character edit.
   - **`therapist_clients` column list (fix F).** Stopping the reads of the five columns means `select=*` had to become an explicit list. `created_at` and `updated_at` are *not* in it: no migration, seed or query in any of the three repos confirms they exist on this table (the vault's March doc lists `linked_at` instead), and PostgREST fails the whole request on an unknown column, which here would mean a dashboard that does not load. Two cosmetic readers degrade (section 2, F). If Studio confirms the columns, adding the two names to the one constant restores both.
   - **Verification item 9** says "Supabase client + fonts". The dashboard loads neither from a CDN (raw `fetch`, `font-src 'self' data:`). Its only CDN resources are the two pinned driver.js files on cdnjs; those are what the real-CDN load checks.
   - **The harness is not committed**, following the chat-side S7 precedent (`S7_client_fixes_session_report_v1_0.md`, section 5). It lives in the session scratchpad and can be committed under `scripts/` if wanted (section 6).
   - **The report opens with this Session Log**, per the standing report convention.
6. **Commits** (each passed `scripts/scan-invisible.py dev.html dev.css` CLEAN, `node --check` on the inline script, and CSS brace balance first):

| Commit | Fixes |
|---|---|
| `68873b2` | E + H -- one verified mutation helper; therapist guard |
| `363521a` | A + B (+ crisis half of H) -- no crisis free text; guest branch removed |
| `882d99e` | F + G -- `therapist_client_private`; database-generated invite codes |
| `581a758` | D -- `loadDashboard()` rejects; load-error state (`dev.css`, `?v=40`) |
| `cd1f41c` | C -- sign-out resets everything |
| `1291c07` | I + J -- prep pack; copy corrections |
| `faa1e10` | F follow-up found in the pre-report read-through |

7. **Checklist.** A COMPLETED. B COMPLETED. C COMPLETED (one item declined, above). D COMPLETED. E COMPLETED. F COMPLETED. G COMPLETED. H COMPLETED. I COMPLETED. J COMPLETED. K NOTED (no change, as instructed). L COMPLETED before every commit. Verification 1-9 COMPLETED. Report COMPLETED.

---

## 1. Summary

| Fix | Finding(s) | What changed | Commit | Harness |
|---|---|---|---|---|
| A | R-B-02 (5.2, A-15) S1 | Every `crisis_events` read names nine label/timestamp columns; `action_notes` and `details` are neither fetched nor rendered; resolve modal shows severity, category, Alo's action, timestamp | `363521a` | `no_crisis_text` 10/10 |
| B | R-B-01 S1, R-B-07 S2 | `client_id.is.null` gone; query and answer both scoped to own links; "Unknown Client" gone; no `currentClient` fallback; a null-client alert cannot be opened or resolved | `363521a` | `home_scope` 16/16, `cap_displacement` 6/6 |
| C | R-B-03 S1 | One sign-out function; caches nulled; app and static modals restored to shipped markup; sign-in always lands on Home; late answers from a previous session are dropped | `cd1f41c` | `signout_signin` 19/19, `forced_logout` 9/9, `late_refresh_race` 8/8, `zero_clients` 9/9, `tour_signout` 8/8 |
| D | R-B-05 S2 | `loadDashboard()` rejects; error state with retry; all seven call sites handle it | `581a758` | `load_failure` 18/18 |
| E | R-B-06 S2 | `apiMutate()`: PATCH/DELETE/upsert use `return=representation`, empty result throws into the existing failure copy | `68873b2` | `zero_row_mutations` 22/22 |
| F | R-B-17 (5.2b) S2 | Five fields read from / upserted to `therapist_client_private`; never sent to or selected from `therapist_clients` | `882d99e`, `faa1e10` | `client_edit_private` 21/21, `private_unknown` 10/10 |
| G | R-B-18 S2 | No client-side code; insert without `invite_code`, show the returned one | `882d99e` | `invite_db_code` 10/10 |
| H | R-B-14, R-B-16 S2 | `selfTherapistId()` on every by-id mutation, the home messages query, the mark-read PATCH and every POST body; `crisis_events` scoped via the client's link | `68873b2`, `363521a`, `882d99e` | `guards` 18/18 |
| I | R-B-12 S2 | Open alerts written as open; "resolved via" only when acknowledged, followed by `action_taken` | `1291c07` | `prep_pack` 8/8 |
| J | R-B-08, -09, -10, -11, -13 S2 | Five sentences corrected (section 3) | `1291c07` | `copy_static` 12/12 |
| K | R-B-15 | Note only (section 2, K) | -- | -- |
| L | hygiene | CLEAN before each of the seven commits | all | -- |

Harness totals: the final branch passes **219 of 219 checks in 18 scenarios**, with 0 uncaught exceptions or unhandled rejections and 0 requests leaving the machine. `main`'s `dev.html` fails all 17 scenarios that encode a finding (101 of 218 checks) and passes the one control (`real_cdn_load`). Section 6.

**Three things found along the way that Kano should read first** (section 8.1):
1. The dashboard has always read and written a column called `phone`. Every migration calls it `client_phone`. If the live table has no `phone` column, **every Client Settings save on `main` has been failing** with PGRST204 -- display name and toggles included. Fix F removes the question.
2. **The password stayed in the login form after sign-out.** The form is only hidden while the app is up. Fixed as part of C.
3. **A token refresh that finished late was installed over whoever had signed in since.** Fixed as part of C.

**One sequencing hazard** (section 8.3): production `index.html` still reads `select=*` and writes the five fields to `therapist_clients`. Migration v1.5 **Step 3 must not run before this work is promoted.**

---

## 2. Fix by fix: before and after

Line numbers are `dev.html` on the final branch unless marked `main:`.

### A -- Never render `action_notes` or any excerpt (R-B-02, A-15)

**Before.** Both crisis reads were `select=*` (`main:1729`, `main:2439`), so `action_notes` -- bot-authored, carrying up to 200 characters of what the client typed -- reached the browser on every load, sat in `clientCrisis` and `window._allCrisisEvents`, and was visible in the Network panel. `renderCrisisAlerts` rendered it for resolved alerts (`main:3483`) and rendered a `details` column verbatim for open ones (`main:3467`).

**After.**
- `CRISIS_EVENT_COLUMNS` (1116): `id, client_id, severity, category, alo_response_action, acknowledged, acknowledged_at, action_taken, created_at`. Every one appears in the seed's `insert into public.crisis_events`. It is the `select=` of both reads (2021, 2873). The resolve PATCH returns `select=id` only (fix E), so no response of any kind carries the column.
- The "Recently resolved" line is `Action: <action_taken>` and nothing else. The open-alert item is category plus Alo's response; the `details` rendering is gone. The seed defines no `details` column, so on real data that line read "No details provided".
- The resolve modal (2490) now opens with the alert itself: `Flagged <timestamp>`, severity, category, `Alo Response: <alo_response_action>`, using the alert card's existing classes. It previously showed no event field at all.
- `action_notes` is still **written** by the resolve flow (2609): it holds what the therapist typed, and the write overwrites the bot's excerpt on that row. It is never read back. The notes field now says so (section 3, last row).

**Static check:** `action_notes` appears four times in the file: three comments and the PATCH body. It is in no DOM-writing path and in no `select=`.

### B -- Remove the guest branch (R-B-01, R-B-07)

**Before.** `or=(client_id.in.(<ids>),client_id.is.null)&...&limit=50` (`main:1729`). Null-client rows rendered as "Unknown Client" with a Resolve button (`main:1827`, `main:1856`); `openResolveCrisisModal` fell back to `currentClient` for them (`main:2111-2113`), which could put one client's phone and safety notes under a stranger's alert; and 50 newer guest rows pushed an own client's open alert out of the list and out of CRISIS status.

**After.**
- Query (2021): `crisis_events?client_id=in.(<own ids>)&acknowledged=eq.false&order=created_at.desc&limit=50&select=...`. Skipped entirely when no linked client has signed up (the zero-uuid placeholder is gone).
- The **answer** is filtered by the same rule before anything counts or lists it (2033-2034), so a row with no client, or another therapist's client, cannot be counted, listed or kept in `window._allCrisisEvents` even if a policy handed one back.
- `findOwnCrisisEvent()` (2480) is the only way to an alert: a known event whose `client_id` maps to one of this therapist's own links. `openResolveCrisisModal` refuses anything else with a toast; `submitResolveCrisis` refuses before any request and lands in the existing "Error resolving alert. Please try again."
- `renderAlerts` / `showAllAlerts` fall back to "Client" (as `renderMessages` always did) and the row is always clickable.

### C -- Sign-out resets everything (R-B-03)

**Before.** `handleLogout()` nulled `session` and `therapistId`, removed the stored session and called `showLogin()`, a visibility toggle (`main:1478-1483`). The client page stayed `active` with its notes, messages and emergency panel; `currentClient` and all five caches stayed populated; the edit form kept the last client's phone and safety notes in its hidden inputs. The next sign-in flipped `appContainer` back on and that is what appeared.

**After.**
- `signOutLocally()` (1262) is the one sign-out path: the Sign Out button and the pending-verification page's Log Out (`handleLogout`, 1694), a rejected token refresh in `api()` (1339), and a stored session that is not a therapist's at page load (1414). It bumps `authEpoch`, drops the session, then `clearAppState()` and `clearLoginForm()`.
- `clearAppState()` (1186): nulls `currentClient`, `allClients`, the five per-client caches, `clientPrivate`, the home alert/message lists and the calendar state; bumps the A7/D2 generation counter; destroys a running tour without writing an onboarding timestamp; removes every dynamically built overlay and the calendar popup (the two auth-flow overlays are left alone); closes the five static modals and releases their focus traps; and **restores `#appContainer` and the static modals to their shipped markup**, snapshotted at parse time (1179). That last step is what makes the reset complete by construction: a card or input added to the client page later is covered without anyone maintaining a list. Every handler in this file is an inline attribute and nothing holds a node reference into those containers, so the round trip is lossless.
- `completeSignIn()` bumps `authEpoch` and calls `clearAppState()` itself before `showApp()`, and `showApp()` asserts Home first, so "lands on Home with a fresh `loadDashboard()`" does not depend on a sign-out path having run.
- **`authEpoch` (1163).** `api()` re-checks it after every await; `loadDashboard()`, `loadClientData()`, `loadClientPrivate()`, `showApp()` and `openAsClient()` drop an answer that comes back under a different epoch instead of caching or rendering it. `loadSharedJournal()` checks the generation counter (the reset bumps it), which also closes a pre-existing race where a slow response for client X rendered under client Y.

**Brief's test** (open a client, sign out, sign in): Home, `currentClient === null`, nothing of the previous user anywhere in the document or in any input value. Harness `signout_signin`.

### D -- `loadDashboard()` stops swallowing errors (R-B-05)

**Before.** Its only `catch` logged and returned (`main:1804-1806`), so it never rejected: the three A7/D3 refresh-failure toasts were unreachable, and a failed first load sat on "Loading your dashboard..." with the static "All clear -- no items needing attention." under it and "No clients yet" in the sidebar.

**After.**
- It shows the failure on the page and rethrows (2107-2114).
- `#homePage` ships with class `home-not-loaded` (193), which hides the home sections until `markDashboardLoaded()` (1916). Neither a pending nor a failed load can read as "All clear".
- `#dashboardLoadError` (199, `role="alert"`, retry target 91x44) carries one of two constant messages (section 3). The headline summary is cleared on any failure, first load or refresh, because it is a claim about now. After a failed first load the sidebar says "Could not load clients".
- All seven call sites are wrapped. `submitResolveCrisis` and `saveClientEdit` awaited it bare inside their own `try`; with a rejecting callee, a refresh failure after a write that *landed* would have been reported as "Error resolving alert" / "Error updating client". They now handle it exactly like the A7/D3 sites.
- `showApp()` does not offer the tour over a dashboard that did not load; `retryDashboardLoad()` (1943) offers it once a load succeeds.
- One-line ride-along in the same spirit: the zero-client branch is a *successful* load and no longer leaves the headline on "Loading your dashboard...".

### E -- Mutations verify their effect (R-B-06)

**Before.** `apiMutate()` and three hand-rolled calls sent `Prefer: return=minimal`; a PATCH or DELETE filtered to zero rows came back 204 and was toasted as success.

**After.** `apiMutate(endpoint, method, body, opts)` (1362) is the one helper. PATCH, DELETE and upserts send `return=representation`, narrowed with `select=id` (or `opts.returning`) so a mutation never echoes note or safety text back; an empty array throws into each caller's **existing** constant failure copy. No new failure strings. `updateNote`, `deleteNote` and `markClientMessagesRead` no longer hand-roll. Plain POSTs keep `return=minimal`: an insert cannot match zero rows.

One site needed thought: the mark-read PATCH is filtered on `read=eq.false`, so with nothing unread "zero rows" is the normal answer. It is now sent only when the list `loadClientData()` just fetched contains unread messages; then an empty result does mean it did not land. Opening a client with nothing unread sends no PATCH and raises no false alarm (harness control).

Side effect, intended: `_aloWriteOnboardingTimestamp` goes through the same helper, so R-B-19's zero-row write is now a logged warning rather than a silent 204. R-B-19 itself is S3 and was not fixed.

### F -- Safety fields move to `therapist_client_private` (R-B-17)

**Before.** `saveClientEdit` PATCHed `phone`, `emergency_contact_name`, `emergency_contact_phone` and `safety_notes` onto the `therapist_clients` row the client's own JWT can read; every list read was `select=*`.

**After.**
- **Read** by `therapist_client_id=eq.<link id>`: a sixth request in `loadClientData()` (2875), and `loadClientPrivate()` (2432) for the resolve modal opened from Home. Per client, when that client's page or alert is opened -- never for the whole list, and the home load does not touch the table.
- **Write** (4539): `POST therapist_client_private?on_conflict=therapist_client_id` with `Prefer: resolution=merge-duplicates,return=representation`, through `apiMutate()`, before the settings PATCH. Body: `therapist_client_id` plus the four form fields. `clinical_context` is never sent, so an existing value survives. The `therapist_clients` PATCH body is now `display_name`, `next_session_date` and the four `allow_*` flags.
- **Every `therapist_clients` read names its columns** (`THERAPIST_CLIENT_COLUMNS`, 1139; three sites), so the five columns stop arriving while they still exist on that table. Cost of leaving out the two unconfirmed timestamps: the test-client row shows no relative time when it has no activity in the 100-row window (it used to fall back to `created_at`), and the archived list reads "Archived" without a date.
- **A failed private read is "unknown", not "none on file".** `clientPrivate[linkId]` is `{state:'loaded', row}` or `{state:'failed'}`. The emergency panel says it could not load; the missing-info banner stays quiet; the edit form will not open over blanks it could then save over real values.
- The emergency panel is one helper (`emergencyPanelHtml`, 2452) instead of two copies (R-B-35 half-closed as a by-product; both copies had to change identically), and it is redrawn after a save -- including when the private write lands and the settings PATCH then fails (`faa1e10`).

### G -- Stop minting invite codes client-side (R-B-18)

`Math.random` is gone from the file. `generateInvite()` (4389) inserts `{therapist_id, display_name, status:'pending'}` with `?select=id,invite_code` and `return=representation`, and shows the returned code. Copy / text / email all read `inviteCodeDisplay`, so that is their one source. A response with no code is "Error generating invite". The 12-character code fits the modal at 375 px on one line (measured 283 px wide in a 375 px modal; screenshot checked).

### H -- The therapist guard (R-B-14, R-B-16)

`selfTherapistId()` (1384) returns `session.user.id` and throws if there is none, rather than building a request without a guard. Every mutation the branch sent in the harness, in order (`<SELF>` = the signed-in therapist):

```
PATCH  client_messages?therapist_id=eq.<SELF>&client_id=eq.<client>&read=eq.false&select=id
PATCH  crisis_events?id=eq.<event>&client_id=eq.<own linked client>&select=id
POST   therapist_client_private?on_conflict=therapist_client_id&select=...   body.therapist_client_id = own link
PATCH  therapist_clients?id=eq.<link>&therapist_id=eq.<SELF>&select=id      (client edit)
POST   session_notes                                                         body.therapist_id = <SELF>
POST   session_notes                                                         body.therapist_id = <SELF>   (quick note)
POST   homework_cards                                                        body.therapist_id = <SELF>
PATCH  session_notes?id=eq.<note>&therapist_id=eq.<SELF>&select=id
DELETE session_notes?id=eq.<note>&therapist_id=eq.<SELF>&select=id
POST   therapist_clients?select=id,invite_code                               body.therapist_id = <SELF>
PATCH  therapist_clients?id=eq.<link>&therapist_id=eq.<SELF>&select=id      (archive)
PATCH  therapist_clients?id=eq.<link>&therapist_id=eq.<SELF>&select=id      (restore)
```

13 requests, all guarded (the mark-read PATCH ran for two clients, so the twelve lines above are thirteen requests). Reads: the home `client_messages` query now carries `therapist_id=eq.<SELF>` (2023); the other reads already did where the column exists. `therapist_client_private` has no `therapist_id` column, so the save first checks the link id is one of the therapist's own. The dashboard has no by-id mutation on `homework_cards`.

### I -- Prep pack tells the truth about open alerts (R-B-12)

| | Safety line for a client with one open and one recently resolved alert |
|---|---|
| `main` | `Safety: Alert on Sep 18: passive_ideation; resolved via provided_988_resources.` |
| branch | `Safety: Alert on Sep 18: passive_ideation -- open. Alert on Sep 15: passive_ideation -- resolved via reviewed_safety_plan.` |

`main` marked the **open** alert resolved, via Alo's in-chat response, and omitted the alert that actually was resolved. "resolved via" now appears only when `acknowledged` is set and is followed by the therapist's own `action_taken`. Resolved alerts are the same seven-day set the alert card shows. Without them a week containing a resolved alert would still have read "Safety: No alerts."

### J -- Copy corrections

Section 3.

### K -- R-B-15, note only

As instructed: the database policy on `journal_entries` already requires an active link **and** the therapist's sharing setting, so another therapist cannot reach a shared entry; no dashboard change. The dashboard filter stays `client_id` + `shared_with_therapist`.

### L -- Hygiene

`python3 scripts/scan-invisible.py dev.html dev.css`: CLEAN on both, exit 0, before each of the seven commits and on the final tree. `node --check` on the inline script: OK. `dev.css` braces 652/652. No smart quotes, NBSP or backtick fences introduced (all new copy uses straight quotes).

---

## 3. Copy: before / after

| # | Where | Before | After | Finding |
|---|---|---|---|---|
| 1 | Login page (37) | "Client data is encrypted at rest and in transit. Alowen conversations are not permanently saved — chat history is automatically cleared." | "Client data is encrypted at rest and in transit. Conversation history is saved for your clients, and they can delete it. You never see it." | R-B-10 |
| 2 | Invite modal, name hint (625) | "Friendly name visible only to you" | "Your client will see this as their name in Alo" | R-B-09 |
| 3 | Invite modal, under the code (631) | "Expires in 7 days • Click code to copy" | "Click code to copy" | R-B-08 |
| 4 | "What clients see", screen 2, last sentence (4974) | "Nothing is shared unless you share it." | "Beyond what you share, your therapist can see when you use Alo and when you view or complete homework they assign -- never what you say to Alo." | R-B-11 |
| 5 | Tour, crisis step (5101) | "Every alert, open or resolved, stays here with the action taken. The safety record never lives only in your memory." | "Open alerts stay here until you resolve them. Resolved alerts stay for 7 days with the action you took, then leave this card. The safety record never lives only in your memory." | R-B-13 |
| 6 | Prep pack, Safety line | "Alert on <date>: <category>; resolved via <alo_response_action>" (for open alerts) | "Alert on <date>: <category> -- open" / "... -- resolved via <action_taken>" | R-B-12 (fix I) |

**On row 4.** That modal's contract (its own code comment) is to show what clients actually see, mirroring the chat app's onboarding constant. The chat app's Brief 3 fix J (R-C-13, commit `482ed96`, on chat `main`) already replaced this sentence. The dashboard now carries **that same sentence**, so "What clients see" is still what clients see. It names presence and homework viewed/completed; crisis alerts are the subject of the next screen in the same modal. The dashboard constant keeps `--` where the chat app uses em dashes, as it did before.

**New strings introduced by the fixes** (none replaces approved copy):

| Where | String | Why |
|---|---|---|
| Load-error state | "Could not load your dashboard. Check your connection and try again." + button "Try again" | D |
| Load-error state, after a failed refresh | "Could not refresh your dashboard. What you see below may be out of date." | D |
| Sidebar after a failed first load | "Could not load clients" | D (was "No clients yet") |
| Emergency panel | "Loading emergency contact information..." / "Could not load emergency contact information. Reload the page to try again." | F |
| Toast | "Could not load emergency contact information. Try again." | F (edit form refused) |
| Load toast label | "emergency info" (in the existing "Couldn't load: ...") | F |
| Toast | "Could not open this alert. Reload the page and try again." | B (unreachable in normal use) |
| Resolve modal | "Flagged <timestamp>" | A |
| Resolve modal, under Notes | "Saved with the alert for your audit trail. Not shown again in this dashboard." | A -- **judgment call, flagged.** Fix A removes the only place those notes were displayed. Without this line the field would imply notes the therapist can come back to |

The tour's first step, "Your home page, every time you sign in." (R-B marked it false after R-B-03), is unchanged and is now true.

---

## 4. Grep results

**The five private columns (and the legacy `phone`), every reference in `dev.html`:**

```
1129  const PRIVATE_COLUMNS = 'therapist_client_id,client_phone,emergency_contact_name,emergency_contact_phone,safety_notes';
2461-2464  emergencyPanelHtml()            reads p.* where p = a therapist_client_private row
3546-3548  checkEmergencyContactWarning()  reads p.*  (same)
4503-4506  showEditClientModal()           fills the form from p.*  (same)
4520, 4536 comments
4541-4544  saveClientEdit()                body of the upsert to therapist_client_private
```

**References against `therapist_clients`: none** (expected: none). `clinical_context` appears once, in a comment explaining why it is never sent. `.phone` / `phone:`: 0.

**`action_notes`, every reference:** `1112` comment; `2600` comment; `2609` the resolve PATCH body (write); `3870` comment. In no `select=`, in no DOM-writing path.

**Strings that must be gone (count in `dev.html`):** `client_id.is.null` 0; `Unknown Client` 0; `Math.random` 0; `c.details` 0; `select=*` in code 0 (four comments say never to use it); `prefer: 'return=minimal'` 0; `Expires in 7 days` 0; `visible only to you` 0; `not permanently saved` 0; `automatically cleared` 0; `Nothing is shared unless you share it` 0; `Every alert, open or resolved` 0; `'; resolved via '` 0.

**Production files:** `git diff main -- index.html styles.css manifest.json CLAUDE.md` is empty.

---

## 5. Verification -- the brief's nine items

| # | Item | Result |
|---|---|---|
| 1 | Home query contains no `client_id.is.null`; no "Unknown Client" string in the file | PASS. Static: 0 and 0. Dynamic (`home_scope`): scoped to own ids, newest-first, `limit=50`; a guest row and another therapist's row that a permissive server returns anyway are not rendered, not counted, not kept in memory, cannot be opened, and no PATCH is ever sent for them |
| 2 | No crisis query selects `*`; `action_notes` in no DOM-writing code path | PASS. Static: section 4. Dynamic (`no_crisis_text`): no alert free text in the DOM, in any input, in the calendar popup, in the prep pack or in any cache -- and still none in the DOM or the pack when the server ignores `select=` and sends `action_notes` regardless |
| 3 | Sign-out then sign-in lands on Home with `currentClient === null` | PASS (`signout_signin`, 19 checks). Also after a forced sign-out (`forced_logout`), when a late token refresh races a different sign-in (`late_refresh_race`), after a zero-client therapist (`zero_clients`), and with a tour running (`tour_signout`) |
| 4 | A stubbed 200-with-empty-array PATCH gives failure copy, not a success toast | PASS for all seven: resolve, client edit (both writes), note update, note delete, archive, restore, mark-read (`zero_row_mutations`, 22 checks, with controls that the same paths succeed when a row changes) |
| 5 | Client edit writes to `therapist_client_private` only; the `therapist_clients` PATCH body has none of the five fields | PASS (`client_edit_private`, 21 checks): upsert headers and `on_conflict`; body is the link id plus four fields; `clinical_context` untouched in the stub database; the legacy columns on `therapist_clients` unwritten and never received |
| 6 | Invite insert sends no `invite_code`; the modal shows the code from the stubbed response | PASS (`invite_db_code`): body `{therapist_id, display_name, status}`; modal, copy-link and invite message all use the stub's `ALO-DBGEN231` |
| 7 | Every by-id mutation URL contains the therapist guard (list them) | PASS: all 13 mutation requests the branch sent, listed in section 2, H |
| 8 | `python3 scripts/scan-invisible.py` CLEAN | PASS: `dev.html: CLEAN`, `dev.css: CLEAN`, exit 0 |
| 9 | Real CDN page load with no console errors | PASS (`real_cdn_load`, no stub): driver.js and driver.min.css load from cdnjs under their SRI hashes and the page's own CSP; `dev.css?v=40` loads; login page shown; exactly two external fetches (the two pinned files); 0 console errors, 0 CSP refusals, 0 SRI failures, 0 exceptions. With no stored session the page makes no Supabase request on load, and the Supabase origin was blocked at the network layer throughout |

---

## 6. Harness results

**Method** (the S4A-min / R-C / chat-side S7 pattern). Headless Google Chrome 153.0.8010.50 driven over the DevTools protocol by a zero-dependency Node 25 script (built-in `http`, `child_process`, `WebSocket`); a local static server; a fresh browser context per scenario. Each scenario runs against the **unmodified** branch `dev.html` and the unmodified `main` `dev.html`; nothing in the page is patched. Two scripts are injected before any page script with `Page.addScriptToEvaluateOnNewDocument`:
- a **Supabase emulation** that answers `window.fetch` for the project's origin: GoTrue password and refresh grants, and PostgREST over an in-memory database with `eq` / `in` / `is` / `or`, `select`, `order`, `limit`, `Prefer`, upsert with `on_conflict`, the Migration v1.5 write guards as the brief states them, and **PostgREST's 400 for any unknown column**, with table columns taken from the seed, the migrations and the brief (so `phone`, and `created_at` / `updated_at` on `therapist_clients`, fail as they would live). It has **no row-level security** beyond token validity, on purpose: every scoping decision under test has to come from the dashboard's own filters. It can inject zero-row mutations, 5xx faults, delays, expired tokens, rejected refreshes, a server that leaks guest rows, and a server that ignores `select=`;
- the scenarios, which drive the real UI (the sign-in form, buttons, form submits).

**Hermetic by construction:** every request the page makes is intercepted at the network layer and *failed* unless it goes to the harness's own server. `real_cdn_load` and `tour_signout` additionally allow the two pinned cdnjs files. Requests that tried to leave the machine: 0. Calls to the live Supabase API: 0. Real credentials used: none. Clipboard permission is granted to the page, as a real click would, so the dashboard's pre-existing un-caught copy promises (R-B S3, out of scope) do not show up as noise.

| Scenario | What it does | Final branch | `main` |
|---|---|---|---|
| `home_scope` | home query shape; permissive server; then a server that leaks a guest row and another therapist's row; open and resolve attempts on the guest event | PASS 16/16 | FAIL 4/16: `client_id.is.null` in the query; "Unknown Client" rendered; counted; resolvable; the guest event was PATCHed |
| `cap_displacement` | 60 guest events newer than an own client's open alert | PASS 6/6 | FAIL 3/6: alert displaced, client not CRISIS, count reads 50 |
| `no_crisis_text` | every surface that shows an alert, with and without the server honouring `select=` | PASS 10/10 | FAIL 4/10: `CANARY_RESOLVED_NOTES` and the quoted client sentence in the DOM |
| `signout_signin` | the brief's test, plus the edit form, an open drawer and a half-typed quick note left behind | PASS 19/19 | FAIL 9/19: B lands on A's client page; A's notes, messages and homework on B's page; **password still in the form (20 characters)** |
| `forced_logout` | access token expired, refresh rejected | PASS 9/9 | FAIL 6/9: A's page left behind; next sign-in not on Home |
| `late_refresh_race` | A's refresh is slow and succeeds; A signs out and B signs in meanwhile | PASS 8/8 | FAIL 4/8: **`session`, `therapistId` and the stored session all become A's while B is signed in** |
| `zero_row_mutations` | seven mutations answered 200 `[]`; controls | PASS 22/22 | FAIL 8/22: "Crisis alert resolved", "Note updated", "Note deleted", "Client archived", "Client restored" on writes that changed nothing |
| `client_edit_private` | reads, the edit form, the two writes, a pending link with no private row | PASS 21/21 | FAIL 6/21: five fields in the `therapist_clients` PATCH; `select=*`; and the save itself fails with PGRST204 on `phone` |
| `private_unknown` | the private read fails | PASS 10/10 | FAIL 4/10 (no such state on `main`) |
| `invite_db_code` | invite insert and the three send actions | PASS 10/10 | FAIL 5/10: sends a `Math.random` code |
| `guards` | every mutation once; a client linked to two therapists | PASS 18/18 | FAIL 11/18: A's feed shows a message written to B, and opening the client marks B's message read with A's id |
| `load_failure` | failed first load, retry, and a failed refresh after each of five writes | PASS 18/18 | FAIL 4/17: "Loading your dashboard..." forever over "All clear"; no toast reachable |
| `prep_pack` | one open and one resolved alert | PASS 8/8 | FAIL 5/8 |
| `copy_static` | the five sentences | PASS 12/12 | FAIL 3/12 |
| `zero_clients` | a therapist with no clients signs out; another signs in | PASS 9/9 | FAIL 5/9: **the next therapist's Needs Attention section stays hidden and the alert count reads 2** |
| `tour_signout` | sign-out with the tour running (real driver.js); then a tour closed by hand | PASS 8/8 | FAIL 7/8: tour left over the login page |
| `happy_path` | an ordinary session end to end; zero console errors required | PASS 8/8 | FAIL 6/8 (the `phone` PGRST204 above) |
| `real_cdn_load` | verification 9, no stub | PASS 7/7 | PASS 7/7 (control) |

**Totals:** final branch 219/219 checks, 0 page errors or unhandled rejections; `main` 101/218.

**Phone width** (`CLAUDE.md`: mobile-first). Six states were screenshotted at 375x812 and looked at: login page, load-error state, invite modal with a 12-character code, resolve modal opened from Home, the client alert card, the alert card when the private read fails. No horizontal scroll in any; the code stays on one line; the retry target is 91x44. New colours are tokens only (`--danger-bg`, `--danger-border`, `--danger-text`; 7.6:1 on the error box, above AA).

**Harness bugs found and fixed while building it** (none in the app): a check that read the page's own script source as rendered text; and a tour scenario that closed driver.js inside its ~400 ms highlight transition, during which driver.js does not call `onDestroyed` at all, so a "writes no timestamp" check was passing trivially. Both scenarios now wait the transition out.

**Where it is.** Five files in the session scratchpad (`harness/stub.js`, `scenarios.js`, `run.mjs`, `shots.js`, `summary.mjs`; about 1,300 lines; needs only Node and Chrome). Not committed. Say the word and it goes on this branch under `scripts/harness/`.

---

## 7. Not verifiable without live credentials or Studio

1. **That `therapist_client_private` is exactly as the brief describes it**: the five column names (notably `client_phone`), `therapist_client_id` as primary key (the upsert's conflict target), and that `therapist_private_all` plus the table grants let `authenticated` SELECT, INSERT and UPDATE through PostgREST. The Step 1 SQL is in no repo and no local document. If a name differs: the client page toasts "Couldn't load: emergency info", the panel says it could not load, and the edit form will not open. Loud, and nothing is written.
2. **That every name in `THERAPIST_CLIENT_COLUMNS` and `CRISIS_EVENT_COLUMNS` exists live.** Each is evidenced by the seed, a migration or a working query in the chat app, but none was read from the live schema. A wrong name fails the request with a 400; after fix D that is the load-error state, not a silent blank.
3. **Whether `therapist_clients` has `created_at` / `updated_at`** (deviation 3), and **whether it has a `phone` column** (section 8.1).
4. **That `invite_code` has the database default live.** If it did not, the insert would either error (NOT NULL) or leave a pending row with no code and toast "Error generating invite". The brief states the default exists.
5. **"They can delete it" (login page).** The chat app has per-conversation delete and "Clear unpinned conversations", but R-C-05 records that under the RLS Lockdown v1.2 text those DELETEs may match zero rows. The sentence is the brief's wording and is true only if the DELETE policies exist live. Flagging it rather than weakening it.
6. **"Encrypted at rest"** (login page, unchanged): a Supabase platform property.
7. **RLS itself.** The harness has none by design. That the live policies agree with the dashboard's filters is R-A's territory.
8. **Whether the live `therapists` rows are keyed so that R-B-19's two calls match** (section 8.3).
9. **Real devices.** Headless Chrome at 375 px is not an iPhone or an iPad.

---

## 8. Found along the way, what needs a ruling, and what was declined

### 8.1 Found while doing the work, same class as the brief, fixed

1. **`phone` vs `client_phone`.** The dashboard read `currentClient.phone` and PATCHed `phone:`. Every migration that touches the column calls it `client_phone` (the seed, all five trigger-v3 versions); nothing anywhere defines `phone`. If the live table has no `phone` column then, on `main` and in production: the emergency panel has always said "No phone on file", the "Missing emergency info: client phone" banner has always fired, and **every Client Settings save has failed whole** with `PGRST204 Could not find the 'phone' column` -- display name and toggles included -- because one unknown key fails the entire PATCH. The roadmap still lists "Emergency contact save test" as not run. The harness reproduces this on `main`. After fix F the dashboard never sends `phone` and reads `client_phone` from the new table, so the question disappears, but it is worth thirty seconds in Studio to know whether pilot data entry has been silently failing.
2. **The password stayed in the login form.** The form is hidden, not cleared, while the app is up, so after Sign Out the previous therapist's email and password were still in the inputs: one click on "Sign In" for the next person at a shared computer. Same exposure class as R-B-03 and a shorter path to it. The password fields are cleared the moment the grant succeeds; the whole form at sign-out.
3. **A late token refresh overwrote the next user's session.** `api()` installed whatever `refreshSession()` returned, whenever it returned. A sign-out followed by a different sign-in while A's refresh was in flight ended with `session`, `therapistId` and `localStorage` all A's under B's hands (harness `late_refresh_race`, reproduced on `main`). The retry loop would also have spent the *next* user's refresh token. Closed by `authEpoch`; the refresh token is captured once.
4. **A zero-client therapist broke the next therapist's Home.** The welcome branch hides every home section with inline styles; nothing unhid them at sign-out, so the next sign-in had Needs Attention hidden. The cross-user half is closed by the markup restore. The within-session half (R-B-32) is untouched.
5. **A half-saved client edit left a stale emergency panel** (`faa1e10`), introduced by F's two-write save and caught in the read-through.

### 8.2 Needs a ruling

1. **Sign-out in one tab does not sign out other open tabs of the dashboard.** A second tab keeps its in-memory session and keeps refreshing it (sign-out is local only, R-B-21). On a shared computer that is R-B-03's exposure by another door. A faithful fix is small: `handleLogout` writes a marker key and other tabs reset on its `storage` event. It should key on an *explicit* sign-out, not on removal of `alo_session`, or one tab's refresh failure would sign out every tab and cost an unsaved note. Not added: new auth-adjacent behaviour, not in the brief.
2. **Resolution notes are now write-only in the dashboard** (consequence of A). If therapists should be able to read their own notes back, that needs a column the bot never writes (say `therapist_resolution_notes`): Studio work.
3. **The resolve modal does not name the client.** Pre-existing. From Home's "View all alerts" list the modal shows phone numbers with no name above them. The brief's list for the modal is exact, so the name was not added; it would be one line.

### 8.3 Declined as out of scope, and observations

- **Every S3/S4 in R-B** stands as it was, including the ones this work walked past: R-B-19 (`therapists` keyed on `id`; now a logged warning), R-B-20 (a rejected non-therapist login leaves its session stored), R-B-21 (no server-side logout), R-B-22, R-B-23 (no request timeouts), R-B-24 (every toast wears the success icon, including all the failure toasts this brief made reachable), R-B-26, R-B-27, R-B-28, R-B-32 (within-session half), R-B-36, R-B-37, R-B-38, R-B-39.
- **R-B-04** is the promotion sitting, as the brief says. Nothing here touches `index.html`.
- **Sequencing hazard.** Production `index.html` still has `select=*` on `therapist_clients` (1512, 3850), the guest branch (1559), the `action_notes` rendering (3243), `Math.random` codes (3712) and the PATCH of `phone` / emergency / safety fields onto `therapist_clients` (3775-3778). **Migration v1.5 Step 3 (dropping the five columns) must not run until this work has been promoted**, or production's Client Settings save breaks outright. This branch is ready for Step 3 whenever it runs: it never names those columns on that table.
- **Promotion notes for Session 8.** `dev.css` gained one block, so `styles.css` needs the next `?v=`; the manifest hazard (R-B-38) is unchanged; `dev.html` now references `dev.css?v=40`.
- **`clipboard` promises without `.catch`** (R-B S3) still reject unhandled when the browser denies the write.
- The realtime alert bar and alert email (Gate B, P-4) and the D22 promotion were not touched.

---

## 9. Kano's check on dashboard.alowen.ai/dev.html (after merging to `main`; about ten minutes, test account)

1. **It loads.** Sign in. Home shows the summary and the sections. If it shows "Could not load your dashboard", open the console: a 400 naming a column means one name in `THERAPIST_CLIENT_COLUMNS` or `CRISIS_EVENT_COLUMNS` is wrong (section 7.2).
2. **Private table.** Open the test client. No "Couldn't load: emergency info" toast. Settings: the seeded phone and safety notes are there. Change the phone, Save: "Client updated". In Studio, the change is in `therapist_client_private` and the `therapist_clients` row's five legacy columns did not move.
3. **Invite.** Generate one: the code is `ALO-` plus 8 characters, and the Network panel shows no `invite_code` in the request body.
4. **Crisis.** On the test client's seeded resolved alert, "Recently resolved" shows the action and no sentence. Network panel: the `crisis_events` responses have no `action_notes`.
5. **Sign-out.** Open a client, Sign Out: the login form is empty. Sign in: Home.
6. **Failure state.** DevTools, Network, Offline, reload: the error box with "Try again", no "All clear". Back online, Try again: it loads.
7. **While in Studio:** does `therapist_clients` have `phone`? `created_at`? `updated_at`? (section 8.1, item 1; deviation 3.)

---

*S7 dash fixes v1.0 -- 2026-09-19. Branch `auto/s7-dash-fixes`, eight commits with this one. `dev.html`, `dev.css`, this report. Nothing on `main`, nothing promoted, no live call.*
