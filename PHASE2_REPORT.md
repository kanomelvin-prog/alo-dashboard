# PHASE2_REPORT — Phase 2: therapist tour + installable app + carryover

## Session Log

**Model:** Sonnet 5 (`claude-sonnet-5`).

**Interruptions and restarts:** One. The connection dropped mid-session
immediately after the `docs/decisions.md` ledger-housekeeping commit
(`alo-supabase` `89039e5`, end of task 5). The session resumed from that
exact point: ground-truthed all three repos (`git status` + `git log
--oneline -3` on each `auto/phase2-*` branch, plus `git ls-remote origin
<branch>` to confirm nothing had been pushed anywhere yet — all three
working trees were clean and every prior commit was present locally, so
nothing was lost or re-done) before continuing with task 6. No other
compaction, crash, or retried step occurred in the session.

**Deviations from the brief:**

1. **No PIL available in this environment; icons generated via macOS `sips`
   rasterizing hand-written SVG masters instead**, and task 7's "icons open
   (PIL verify)" was run as a pure-Python PNG signature + IHDR-dimension
   check instead of a PIL open — `pip install pillow` was denied when
   attempted. Same functional guarantee (every icon is a structurally valid
   PNG at its declared size); noted per file below and in task 7.
2. **Task 4 (chat paging) was not stopped or specced-only** — the brief's
   own condition for that ("if the `api()` helper or query shape makes this
   non-trivial, STOP and spec it instead") did not apply: the existing query
   already used `limit=N` via a plain URL param, so adding `offset=N` was a
   direct, low-risk extension of the same shape. Built and verified, not
   deferred.
3. **Task 5's D1-D5 renumbering target was D9-D18 per the brief; the ledger's
   actual high-water mark is D14, not D18** (verified by reading the whole
   series in `docs/decisions.md` — there is no D15 through D18 anywhere in
   the repo before this session). Renumbered to continue from the real
   number, D14, i.e. the colliding D2-D5 became D15-D18, and the brief's own
   D19/D20/D21 labels for the three new entries were kept exactly as given
   (they were never in the renumbered range to begin with — see
   `docs/decisions.md` "Batch Day 2" section for the full reasoning). Flagged
   here rather than silently reconciling the discrepancy without saying so.
4. **The onboarding manual-test addendum (task 6) is explicit that both
   `dev.html` files must be tested locally** (`python3 -m http.server`
   against the unmerged `auto/phase2-onboarding` branch), not against the
   hosted `dashboard.alowen.ai` / `alowen.ai`, because neither is served at
   its production host until this branch merges. This wasn't spelled out in
   the brief but is necessary for the steps to actually be runnable before
   merge.

Nothing else differs. No backup checkpoint was needed in any repo — all
three were clean at kickoff (recorded as clean skips, per the brief's own
instruction).

### Checklist

| # | Task | Status |
|---|---|---|
| 0 | Preflight (notification skip, backup checkpoint) | **COMPLETED** — all 3 repos clean, no checkpoint needed |
| 1 | Branch base: `auto/phase2-onboarding` ×2 off `auto/phase3-audit-fixes`, `auto/phase2-docs` off `auto/phase3-docs` | **COMPLETED** |
| 2 | Dashboard: driver.js therapist tour | **COMPLETED** |
| 3 | Both apps: installable-app work (manifest, icons, Android banner, iOS card) | **COMPLETED** |
| 4 | Chat: server-side paging past the 50-conversation batch | **COMPLETED** — not stopped/specced; see deviation 2 |
| 5 | Carryover: D20 (CLAUDE.md scan refs → script) + ledger housekeeping (D1-D5 renumbering, D19-D21) | **COMPLETED** — see deviation 3 on the renumber target |
| 6 | Docs: onboarding manual-test steps in `PRE_PILOT_TEST_PASS.md` | **COMPLETED** |
| 7 | Static verification | **COMPLETED** — see deviation 1 on the icon-verify substitution |
| 8 | This report; commit; push all branches; `pbcopy`; byte count; `git ls-remote` all repos | **COMPLETED** |

---

## 1. Branch base

All three branches were created from the *unmerged* Phase 3 branches, not
`main`, per the brief's own reasoning (Phase 3 touches the same `dev.html`
files and merging into it here avoids a conflict later):

| Repo | New branch | Based on |
|---|---|---|
| `alo-dashboard` | `auto/phase2-onboarding` | `auto/phase3-audit-fixes` (tip `4900506`) |
| `alo-client-chat-full_v1` | `auto/phase2-onboarding` | `auto/phase3-audit-fixes` (tip `ea965a9`) |
| `alo-supabase` | `auto/phase2-docs` | `auto/phase3-docs` (tip `02fa7d3`) |

Recorded here as the brief required. This means: **`auto/phase2-onboarding`
in each app repo cannot merge to `main` before its own
`auto/phase3-audit-fixes` does** — it carries that branch's commits too.
Same for `alo-supabase`'s `auto/phase2-docs` and `auto/phase3-docs`. Not a
problem, just a merge-order fact for whoever reviews these.

---

## 2. Dashboard — driver.js therapist tour (item 9)

**Library:** driver.js **1.8.0**, vanilla/IIFE build, from cdnjs, pinned
with SRI (`sha512-...`), `crossorigin="anonymous"`,
`referrerpolicy="no-referrer"`. Both the JS (`driver.js.iife.min.js`) and CSS
(`driver.min.css`) resolve with `200` from cdnjs — checked live before
committing. CSP `script-src` and `style-src` both gained
`https://cdnjs.cloudflare.com` (verbatim CSP in DATA FOR ARCHITECT below).

**Walk order — one tour, ten steps**, in the file's own build order
(`_aloTourSteps` in `dev.html`):

1. Dashboard home (no element — centered intro popover)
2. Needs Attention (`#attentionSection`)
3. Client list, test-client row (`#clientsList .client-item` — the first
   rendered row; clicking it via its own existing `onclick` handler is how
   the tour navigates into it, rather than duplicating that logic)
4. Client detail (`#clientName`)
5. Session notes (`#notesContainer`; tour switches to the "In Session" tab
   first)
6. Homework (`#homeworkForm`)
7. Shared journal (`#sharedJournalCard` — **only if not `display:none`**)
8. Client Settings (`.client-header-settings-btn`)
9. Crisis alerts, resolved (`#crisisAlertsPrepCard details` — **only if the
   card isn't `display:none`**; tour switches back to the "Session Prep" tab)
10. Emergency contact (`#editClientModal .emergency-title`; tour opens the
    modal via `showEditClientModal()`)

**Copy:** every title/description is a functional placeholder, each marked
`/* COPY: Kano review */` in the source immediately after the string —
structure (one WHAT sentence, one WHY sentence, no step over two lines) is
final; wording is not.

**Skip-gracefully mechanism:** every step's `element` is either a plain
selector (steps 2, 4-6, 8 — these always exist in the DOM regardless of
page/tab state, so nothing to gracefully skip) or a **pure, side-effect-free
function** (steps 3, 7, 9, 10) that returns `undefined` when its target
genuinely isn't available (no clients at all; no shared journal entries; no
resolved-or-unresolved crisis alert). driver.js's own `skipMissingElement:
true` config (confirmed in the library's own source before relying on it —
see the session's tool-call trail) then advances past that step
automatically, with no broken overlay and no thrown error. All side effects
(tab switching, opening the sidebar, opening the settings modal) live in each
step's `onHighlightStarted` hook instead, which fires exactly once per real
(non-skipped) step activation — never inside the `element` function itself,
which the library calls twice per step (once to check, once to highlight)
and which must therefore stay pure.

**Behavior:**
- Skippable at any step: driver.js's default `allowClose: true` covers the
  × button, Escape, and a click on the darkened overlay
  (`overlayClickBehavior: 'close'`).
- Dashboard fully usable underneath at every step — this is an overlay, not
  a modal that traps the page; nothing in the implementation disables the
  rest of the page.
- **Dismiss-permanently:** any skip/close writes `onboarding_skipped_at`
  (fire-and-forget PATCH via the existing `apiMutate()` pattern, console-warn
  only on failure, never a toast, never blocks the tour). Since auto-offer
  only fires when *both* timestamps are null, one skip is permanent.
- **Finish:** the last step's `Done` button is intercepted
  (`popover.onNextClick`) to set a `_aloTourFinished` flag and close the
  settings modal before calling `driverObj.moveNext()`, so the global
  `onDestroyed` hook can tell finish from skip and write
  `onboarding_completed_at` instead.
- **Auto-offer:** `showApp()` calls `_aloMaybeAutoOfferTour()` after
  `loadDashboard()` resolves; it fetches `onboarding_completed_at,
  onboarding_skipped_at` for the signed-in therapist and only calls
  `startAloTour()` if both are `null`.
- **Re-trigger:** `window.startAloTour()` is exposed globally, plus a
  temporary `Show me around` link in the header
  (`/* TEMP: replaced by help menu */` in both the HTML and CSS), wired to
  the same function.

**Files:** `dev.html` (CSP, driver.js `<link>`/`<script>` tags, header link,
tour module, `showApp()` hook), `dev.css` (`?v=33`, themed popover overrides
using existing CSS custom properties, `.tour-temp-link`).

**Commit:** `27d9971`.

---

## 3. Both apps — installable app (item 13a)

**Manifest:** `manifest.json` in each app root — `name`, `short_name`,
`display: "standalone"`, `theme_color`/`background_color` taken from each
app's own CSS custom properties (not shared/copy-pasted values). Linked
*only* from `dev.html` (`<link rel="manifest" href="manifest.json">`);
`index.html` in both repos is untouched and does not reference it, per the
brief and per standing file-discipline rules in both `CLAUDE.md`s.

**Icons:** generated programmatically — no PIL in this environment (denied
`pip install`), so SVG masters (one rounded-square shape, one full-bleed
shape for maskable/Apple) were rasterized via macOS `sips` at high
resolution and downsampled per target size. Full set per app: `icon-72`
through `icon-512` (8 regular sizes), `icon-maskable-192`/`-512` (80% safe
zone), `apple-touch-icon.png` (180, full-bleed — iOS applies its own
corner mask), `favicon-16`/`-32`. All 26 files (13 per app) verified this
session as structurally valid PNGs at their declared dimensions (pure-Python
PNG signature + IHDR check, see task 7). **Functional placeholders — a sage
background with a plain "A" mark, matching the existing `.login-logo` /
`.brand-icon` treatment already in both apps. Final art is Kano's call.**

**Android — `beforeinstallprompt`:** captured and `preventDefault()`'d in
both apps; surfaced only as a dismissible banner (`#installBanner`), never
auto-prompted. Dismissal persists via `localStorage` (`alo_install_dismissed`)
so it doesn't reappear every load. `appinstalled` also marks it dismissed.

**iOS — chat app only:** `#iosInstallCard`, gated on
`_aloIsIOSNotInstalled()` (UA + touch-point check, `navigator.standalone` /
`matchMedia('(display-mode: standalone)')` for "already installed") **and**
on `loadThreads()` confirming at least one conversation with `turn_count >
0` exists. Shown once — persisted to `localStorage`
(`alo_ios_install_card_shown`) at show-time, not dismiss-time, so it can
never reappear even if the page reloads before the user interacts with it.
Dismissed by any click/tap/keydown anywhere on the page (deferred by one
tick so the interaction that revealed the card doesn't also dismiss it).

**CSP:** verified, not changed beyond item 9's dashboard change — manifest
and icon paths are same-origin in both apps, already covered by `default-src
'self'` / `img-src 'self'`.

**Commits:** `8abdb07` (dashboard), `4de8672` (chat).

---

## 4. Chat — stretch task: paging past 50 conversations

**Not stopped or specced** — see Session Log deviation 2. `PHASE3_REPORT.md`
§5 (chat repo) had flagged that `loadThreads()`'s own `limit=50` meant
`loadMoreThreads()`'s client-side paging could never reach a 51st+
conversation. Fixed with straightforward PostgREST `offset=` pagination:

- `loadThreads()` now tracks `threadsExhausted` (true once a fetch returns
  fewer rows than `THREAD_FETCH_SIZE` = 50, meaning the server has nothing
  left).
- `loadMoreThreads()` is now `async`: when revealing the next local page of
  30 would run past what's already been fetched, and the server might have
  more, it fetches the next 50-row batch via `&offset=${allThreads.length}`
  (same `order=pinned.desc,created_at.desc` clause, so the offset continues
  the same ordered sequence correctly) before re-rendering, deduping by `id`
  against what's already loaded. A failed fetch leaves `threadsExhausted`
  untouched so the next click retries instead of silently giving up.
- The "Load older conversations" button disables itself and reads "Loading…"
  during the fetch.

**Commit:** `f0d8a71`.

---

## 5. Carryover one-liners

**D20 — CLAUDE.md scan references moved to the script.** All three
`CLAUDE.md` files (root, `alo-dashboard`, `alo-client-chat-full_v1` — two
sites in the chat repo's) now call `python3 scripts/scan-invisible.py
dev.html dev.css` instead of carrying the heredoc inline. Root `CLAUDE.md`
is not a git repo (`git rev-parse --is-inside-work-tree` confirmed before
editing) — direct file edit, no branch/commit possible for it. Commits:
`cca0942` (dashboard), `b7563d8` (chat).

**Ledger housekeeping (`alo-supabase/docs/decisions.md`), commit `89039e5`:**
see Session Log deviation 3 for the D9-D18 vs. D14 correction. Final state:

- Batch Day 2's D2, D3, D4, D5 renumbered to **D15, D16, D17, D18** (mapping
  recorded in the section itself; every internal cross-reference within that
  section and the standing "Invisible-character scan convention" section
  updated to match). The A7 dashboard-audit finding codes (`A7/D2`, `A7/D3`,
  `A7/D6`, `A7/D8` in `alo-dashboard/dev.html`) and the therapist-signup
  D2-D8 series earlier in the same file are **untouched** — different
  namespaces, not part of this collision or this fix.
  `PHASE3_REPORT.md` (chat repo) keeps its original D2-D5 section-header
  labels as a dated historical record; the mapping note is what a reader
  needs to cross-reference it.
- **D19** recorded: `alo-dashboard`'s hidden/superseded messages card
  (`#inSessionMessagesCard`) — keep the Phase 3 button-id fix, defer full
  deletion of the card + `renderClientMessages()` to post-pilot.
- **D20** recorded in the ledger too (same number as the CLAUDE.md carryover
  task itself, per the brief's own numbering): the three `CLAUDE.md` files
  left on the heredoc by D18's stated scope now call the script — done this
  session, see above.
- **D21** recorded: dashboard's trap-release asymmetry (8 per-open overlays
  release their focus trap only via Escape, not X/backdrop/Cancel) accepted
  as inert/GC-handled per `PHASE3_REPORT.md` §8's own reasoning; full
  unification backlogged to post-pilot.

---

## 6. Docs — onboarding manual-test steps

Appended **section 10** to `alo-supabase/PRE_PILOT_TEST_PASS.md` (commit
`a60f22e`), same click-path / **Success:** / KANO-ONLY-SQL format as the
existing sheet: tour end-to-end, tour skipping missing steps cleanly,
dismiss-permanently across reload, re-trigger (header link and
`window.startAloTour()`), `onboarding_completed_at` /
`onboarding_skipped_at` landing correctly in both directions (KANO-ONLY
SQL), auto-offer firing exactly once, the Android install banner (emulated
via `chrome://flags/#bypass-app-banner-engagement-checks`, both apps), and
the chat app's iOS card (real-device steps, gated on a real conversation,
dismissed-by-any-interaction, shown-once). Explicitly local-testing
instructions (`python3 -m http.server` against the unmerged branch), since
neither `dev.html` is hosted yet — see Session Log deviation 4.

---

## 7. Verification (static only)

All commands run from each repo's own `auto/phase2-*` branch, this session:

- **`node --check` on every inline `<script>` block:** extracted
  programmatically from both changed `dev.html` files (1 block in
  `alo-dashboard`, 2 in `alo-client-chat-full_v1`) — all pass.
- **Every `onclick`/`onsubmit`/`oninput`/`onchange`/`onkeydown` handler
  resolves:** 63 distinct call targets in `alo-dashboard/dev.html`, 44 in
  `alo-client-chat-full_v1/dev.html`. The handful the automated scan
  couldn't immediately match against a `function` declaration (`closest`,
  `querySelectorAll`, `then`, `reload`) were hand-checked in context and are
  all chained built-in DOM/JS methods (`this.closest(...)`,
  `document.querySelectorAll(...)`, `.then(...)`,
  `window.location.reload()`/`location.reload()`) — not missing
  page-defined functions.
- **Manifest JSON parses:** `python -m json.tool` on both `manifest.json`
  files — both OK.
- **Icons open:** see Session Log deviation 1 — verified via a pure-Python
  PNG signature (`\x89PNG\r\n\x1a\n`) + IHDR-chunk dimension check instead of
  a PIL `Image.open()`, since PIL is not installed and could not be
  installed in this environment. All 26 icon files (13 per app) confirmed
  structurally valid at their declared dimensions.
- **`scan-invisible.py` clean on every changed file** in all three repos —
  every `.md`, `.html`, `.css`, `.json`, and `.py` file touched this
  session, individually listed and CLEAN.
- **CSS brace balance:** `dev.css` open/close brace counts equal in both
  apps (632/632 dashboard, 517/517 chat).
- **Deno suite in `alo-supabase` still green:** `npx -y deno@2 test
  supabase/functions/_shared/` — **18 passed, 0 failed** (unchanged by this
  session's work; run as the brief's own final check).

---

## MANUAL TEST CHECKLIST

*(Browser steps, same style as `PHASE3_REPORT.md`. The full versions with
KANO-ONLY SQL live in `alo-supabase/PRE_PILOT_TEST_PASS.md` section 10 —
this is the compact index.)*

- [ ] **Tour end-to-end** — sign in, run the tour through all 10 steps,
      finish on the last step, confirm the overlay and settings modal both
      close cleanly.
- [ ] **Tour skip-gracefully** — on a client with no shared journal entries
      and/or no resolved crisis alert, confirm those steps are silently
      absent, not broken.
- [ ] **Dismiss-permanently** — close the tour early, reload, confirm it
      does not auto-offer again.
- [ ] **Re-trigger** — click "Show me around" (or run
      `window.startAloTour()` in the console) after a finish or a skip;
      confirm it restarts from step 1.
- [ ] **Timestamps** — KANO-ONLY SQL against `public.therapists` after a
      finish and after a skip; confirm the right column lands and the other
      stays `null`.
- [ ] **Auto-offer** — on an account with both timestamp columns `null`,
      confirm the tour opens on its own on first load, and does not on a
      second load.
- [ ] **Android install banner** — both apps, via the DevTools
      engagement-bypass flag: banner appears, never auto-prompts, dismissal
      persists across reload, Install opens the native browser dialog.
- [ ] **iOS card** — chat app, real iPhone/iPad Safari: does not appear
      before a real conversation exists; appears after one; dismissed by any
      tap; never reappears after reload.

---

## DATA FOR ARCHITECT

### Final CSP meta line (`alo-dashboard/dev.html`, verbatim)

```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' 'unsafe-inline' https://cdn.botpress.cloud https://cdn.jsdelivr.net https://files.bpcontent.cloud https://cdnjs.cloudflare.com;
    style-src 'self' 'unsafe-inline' https://cdn.botpress.cloud https://cdnjs.cloudflare.com;
    img-src 'self' data: blob: https://files.bpcontent.cloud https://*.supabase.co;
    connect-src 'self' https://lelsdezstbnzyxvsvbyx.supabase.co https://*.botpress.cloud wss://*.botpress.cloud https://files.bpcontent.cloud;
    frame-src 'self' https://cdn.botpress.cloud;
    font-src 'self' data:;
    object-src 'none';
    base-uri 'self';
  ">
```

(`alo-client-chat-full_v1/dev.html`'s CSP was **not** changed — manifest/icon
paths there are already same-origin-covered, no third-party script added.)

### Tour step list with selectors (`_aloTourSteps`, `alo-dashboard/dev.html`)

| # | Step | Selector | Conditional? |
|---|---|---|---|
| 1 | Dashboard home | *(none — centered intro)* | No |
| 2 | Needs Attention | `#attentionSection` | No |
| 3 | Client list, test-client row | `#clientsList .client-item` (first) | Skips if no clients |
| 4 | Client detail | `#clientName` | Skips if no clients |
| 5 | Session notes | `#notesContainer` | Skips if no clients |
| 6 | Homework | `#homeworkForm` | Skips if no clients |
| 7 | Shared journal | `#sharedJournalCard` | Skips if hidden/no entries |
| 8 | Client Settings | `.client-header-settings-btn` | Skips if no clients |
| 9 | Crisis alerts, resolved | `#crisisAlertsPrepCard details` | Skips if no resolved/unresolved alert |
| 10 | Emergency contact | `#editClientModal .emergency-title` | Skips if no clients |

### Manifest contents

**`alo-dashboard/manifest.json`:**
```json
{
  "name": "Alowen Clinical Dashboard",
  "short_name": "Alowen",
  "description": "Therapist-facing clinical dashboard for Alowen.",
  "start_url": "./dev.html",
  "scope": "./",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#3A7C86",
  "background_color": "#F3EDE4",
  "icons": [ /* 8 regular (72-512) + 2 maskable (192/512), see file */ ]
}
```

**`alo-client-chat-full_v1/manifest.json`:**
```json
{
  "name": "Alowen — Someone to think out loud with",
  "short_name": "Alowen",
  "description": "Client-facing chat interface for Alowen.",
  "start_url": "./dev.html",
  "scope": "./",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#3A7C86",
  "background_color": "#FFFFFF",
  "icons": [ /* same shape as above */ ]
}
```

Both `start_url` point at `./dev.html` since that's what's actually served
under this branch today — **this needs revisiting at promotion time**, when
Kano decides whether/how this becomes part of production.

### Icon file list (13 per app, identical set, different brand colors already matched to each app)

```
icons/icon-72.png    icons/icon-96.png    icons/icon-128.png
icons/icon-144.png   icons/icon-152.png   icons/icon-192.png
icons/icon-384.png   icons/icon-512.png
icons/icon-maskable-192.png   icons/icon-maskable-512.png
icons/apple-touch-icon.png (180×180, full-bleed)
icons/favicon-32.png   icons/favicon-16.png
```

### Task 4 exact state

**Fully implemented, not specced.** `loadThreads()` and `loadMoreThreads()`
in `alo-client-chat-full_v1/dev.html` now page past the original 50-row
fetch via PostgREST `offset=`. See section 4 above for the mechanism.
Commit `f0d8a71`. No open questions or partial work left on this task.

---

## KANO SITTING LIST

*(In execution order — what only Kano can do.)*

1. **Review and merge order.** Every `auto/phase2-*` branch in this report
   is stacked on its repo's unmerged `auto/phase3-*` branch (section 1).
   Phase 3 needs review/merge first, or these three PRs should be reviewed
   together as one combined diff against `main`.
2. **Run the section 10 manual test pass** in
   `alo-supabase/PRE_PILOT_TEST_PASS.md` — all of it needs a human (or
   Cowork) clicking through a real browser, including the KANO-ONLY SQL
   checks for the two new `therapists` timestamp columns.
3. **Decide the tour copy.** Every title/description in
   `alo-dashboard/dev.html`'s tour is marked
   `/* COPY: Kano review */` — ten steps, structure is final, wording is
   not.
4. **Decide the icon art.** All 26 icon files across both apps are
   functional placeholders (sage background, plain "A" mark). Final art is
   a design call, not an engineering one.
5. **Decide `manifest.json`'s `start_url`** in both apps before any future
   promotion — currently `./dev.html` because that's what's live under this
   branch; wrong for a production PWA install.
6. **D19 follow-up (ledger):** decide whether/when to delete
   `alo-dashboard`'s `#inSessionMessagesCard` + `renderClientMessages()`
   outright, post-pilot, per the ruling recorded in
   `alo-supabase/docs/decisions.md`.
7. **D21 follow-up (ledger):** decide whether dashboard's trap-release
   asymmetry (8 per-open overlays) is worth a full refactor to match the
   chat app's single-close-function design, post-pilot.
8. **The help menu** that will eventually replace the temporary "Show me
   around" header link (marked `/* TEMP: replaced by help menu */` in both
   `dev.html` and `dev.css`) — not built here, out of this brief's scope.
