# "Open as Client" Report — D29

**Date:** 2026-09-09
**Branches:** `auto/open-as-client` in both `alo-dashboard` and
`alo-client-chat-full_v1` (both off `auto/phase2b-surfaces`)
**Nothing deployed. No SQL applied. No `main` touched in either repo.
`alo-supabase` was read-only this entire session — contract source, not a
work target; nothing there was written.**

---

## Session Log

### Model

**Opus, for the design and implementation work** — the security analysis in
section 4, the chat-side consumer in section 3, and the two fixes in
section 4.3 were all done on Opus, across two work spans separated by a
connection drop. **This resume, and everything from section 5 onward
(docs, verification, this report), ran on Sonnet 5** (`claude-sonnet-5`) —
a `/model opus` request mid-resume was declined by the harness ("Kept model
as `Sonnet 5`"), so the remainder of the session stayed on Sonnet rather
than switching back. Flagged per the standing convention that a model
mismatch between the work and the report must never be silently absorbed.

### Interruptions and restarts — two connection drops, both recorded

**Drop 1.** Occurred while starting the chat-side consumer: dashboard
D29-01 was already committed (`bb09efa`); the session was mid-way through
checking whether Supabase's `detectSessionInUrl` client option could
interfere with the `?t=` query-param flow, before writing
`_aloExchangeClientToken()`. Between the drop and the resume, **the app was
updated and restarted**. Resumed from ground truth (`git status` +
`git log --oneline -3` in both repos), confirmed D29-01 was committed and
the chat branch had no D29-03 work yet, reconstructed the brief from
`WORKQUEUE.local.md` in both repos (the resume message itself did not carry
the original task text), then completed the `detectSessionInUrl` check and
wrote and wired `_aloExchangeClientToken()` and `_aloHandleTokenParam()`.

**Drop 2.** Occurred mid-way through applying two security-review fixes —
clearing the token from memory after use (chat side) and adding
`noopener,noreferrer` to the dashboard's `window.open()` call. Resumed from
ground truth again: `git status` showed chat's `dev.html` modified but
**uncommitted** (the D29-03 implementation from before Drop 1's resume,
complete but never landed), and dashboard still clean at `bb09efa` — neither
of the two queued fixes had been applied yet. Completed the token-clearing
fix as originally planned. **The `noopener,noreferrer` fix was NOT applied
as planned** — re-examining the code on this resume surfaced that it would
have broken an existing, deliberately-reasoned mechanism (pop-up-block
detection). See section 4.3 for the full reasoning; this is a genuine
correction to the plan stated at Drop 2, not an oversight.

### Deviations from the brief

1. **The `noopener,noreferrer` fix, planned at Drop 2, was not applied.**
   Corrected on this resume once the code was back in front of me — see
   section 4.3.
2. **The exact original task-numbering brief text is not preserved
   anywhere** (not in this conversation's surviving context, not committed
   to any file). Reconstructed from `WORKQUEUE.local.md` in both repos
   (the authoritative task specs, per this project's own convention) plus
   the two resume messages' own task lists (3=chat, 4=security review,
   5=docs, 6=verification, 7=report). If the reconstruction misses
   something the original brief asked for, that's a real gap — flagged
   here rather than silently assumed complete.
3. **"Docs" (task 5) was scoped narrowly**: one addition to
   `alo-client-chat-full_v1/CLAUDE.md` flagging the new `?t=` surface,
   since `alo-supabase/docs/decisions.md` — the usual home for a ruling like
   D31 — is off-limits this session (read-only). D31 is durably recorded
   only in this report and in `WORKQUEUE.local.md` (which is explicitly
   "local only, never committed" by its own header, so it will not survive
   being overwritten by the next batch of pasted tasks). **Kano should
   record D31 in `alo-supabase/docs/decisions.md` himself**, or ask a
   future alo-supabase-scoped session to.

### Checklist

| # | Task | Status |
|---|---|---|
| 1 | Dashboard D29-01 button | **COMPLETED** (before either drop) |
| 2 | *(no separate task 2 in the reconstructed brief — see deviation 2)* | — |
| 3 | Chat D29-03 token exchange | **COMPLETED** |
| 4 | Security review (D29-02 + chat side, combined) | **COMPLETED** |
| 5 | Docs | **COMPLETED** — scoped as above |
| 6 | Verification | **COMPLETED** — all green, section 6 |
| 7 | This report | **COMPLETED** |

---

## 1. Contract restatement (verbatim-with-citations)

Source: `alo-supabase` (read-only this session) —
`supabase/functions/exchange-client-token/index.ts`,
`supabase/functions/open-as-client/index.ts`, `_shared/{tokens,cors,http}.ts`,
`Alo_Therapist_Signup_Edge_Function_Spec_v1_0.md` §6/§7/§8/§9/§10.

### `open-as-client` (dashboard → server)

> "open-as-client mints a single-use, 60-second token server-side and
> returns the whole chat URL carrying it." — `alo-dashboard/dev.html`,
> comment above `openAsClient()`

- `POST /functions/v1/open-as-client`, `Authorization: Bearer <therapist
  access token>` (verify_jwt = true; the function calls `auth.getUser()`
  itself), `apikey: <anon key>`.
- 200 body: `{ url: "<full chat URL with ?t=<token>>" }` — opened verbatim,
  never constructed client-side.
- Token minted with a 60-second expiry, one `client_session_tokens` row per
  call (`supabase/functions/open-as-client/index.ts:154`).

### `exchange-client-token` (chat → server)

> "POST /functions/v1/exchange-client-token — Authorization: Bearer
> \<anon key\> (spec section 7, "Request") — body: { "token":
> "<43-char base64url>" } — 200 -> { token_hash: "<GoTrue magiclink hashed_
> token>" }" — `alo-supabase/supabase/functions/exchange-client-token/
> index.ts:5-9`

- The caller runs `supabase.auth.verifyOtp({ token_hash, type: 'magiclink'
  })` to get a real session (same file, line 11).
- **Single-use, enforced server-side, atomically**: "Set used_at = now()
  BEFORE anything else. A concurrent second request sees used_at set and
  fails" — a compare-and-swap `UPDATE ... WHERE used_at IS NULL AND
  expires_at > now()`, not read-then-write (`index.ts`, the "Steps 1-3"
  comment block). "EXPIRY AND SINGLE USE ARE BOTH ENFORCED HERE,
  SERVER-SIDE, IN THE DATABASE. Nothing about either depends on the
  caller."
- **All failure modes return the identical 401 `{"error":"unauthorized"}`**
  — malformed body, wrong token shape, not found, already used, and
  expired are indistinguishable by design (`index.ts`, "Spec section 7
  step 2" comment) — server-side mechanics for the same cause-neutrality
  ruling D31 states for the client-side message.
- **No retries, ever**: "Past this line the token is SPENT, whatever
  happens next... Fail closed, not convenient" (`index.ts`, "Past this
  line" comment).

### Ruling D31 (Kano, 2026-09-09)

Recorded verbatim from `WORKQUEUE.local.md` (chat repo), since this is its
only committed-adjacent home (see deviation 3): *"Per D31 (Kano,
2026-09-09): spec §10's intent (never reveal WHY) is preserved by one
constant message; per-cause messages remain FORBIDDEN."*

---

## 2. Dashboard — `openAsClient()` (D29-01, already committed pre-drop)

Restated for completeness, since the security review (section 4) covers it
alongside the chat side. No changes made to this function this session —
reviewed as-is.

- Opens `about:blank` synchronously inside the click handler (before any
  `await`), navigates it after the token returns. Avoids the pop-up-blocker
  problem `window.open()`-after-`await` has on Safari/iPad.
- `tab.opener = null` set manually, achieving the `noopener` effect without
  passing `noopener` as a window feature (which would make `window.open()`
  always return `null`, making a real pop-up block indistinguishable from
  the deliberate security posture — see section 4.3).
- 401 on the therapist's own (possibly stale) access token triggers one
  session refresh and one retry, mirroring the existing `api()` helper.
- Failure: bare `catch {}`, fixed toast string, nothing interpolated.

---

## 3. Chat — `_aloHandleTokenParam()` / `_aloExchangeClientToken()` (D29-03)

**Files:** `alo-client-chat-full_v1/dev.html`. **Commit:** `593c9f4`
(implementation), token-lifetime fix folded into the same function on this
resume (not a separate commit — see section 4.3, the fix predates any
commit of this file this session).

### Flow

1. `_aloHandleTokenParam()` — the literal first line of `DOMContentLoaded`,
   ahead of `_aloHandleInviteParam()`, the Botpress init, and
   `getSession()`. Reads `?t=`, strips it via `history.replaceState`
   (never `pushState` — the token must not become a "back"-reachable
   history entry), returns the token or `null`.
2. If a token was present: `_aloExchangeClientToken(token)` — the one and
   only POST to `exchange-client-token`, `Authorization: Bearer <anon
   key>` (not a user JWT — none exists yet), body `{ token }`.
3. On 200: `token_hash` from the body → `supabase.auth.verifyOtp({
   token_hash, type: 'magiclink' })`. On success this fires `SIGNED_IN` on
   the `onAuthStateChange` listener already registered earlier in the same
   handler, which already calls `loadConnectedState()` — **not duplicated
   here**, deliberately, to avoid a race that would double-fire every
   downstream side effect (Botpress identity sync, the `session_metadata`
   write, homework/memory loads).
4. On **any** failure (network error, non-2xx, malformed body,
   `verifyOtp` rejection) — a single bare `catch {}` shows the one constant
   message via `showToast(ALO_TOKEN_EXCHANGE_FAILURE_MESSAGE, 6000)`
   (6-second duration, longer than the 3-second default, since this is a
   more consequential message than a routine confirmation) and returns.
   The code never falls through to the ordinary `getSession()` check
   afterward, on either outcome — see step 3's reasoning.

### Why `detectSessionInUrl` doesn't matter here

Checked before writing anything (the exact question the first drop
interrupted). `createClient()`'s `detectSessionInUrl: true` only recognises
hash-fragment tokens (`#access_token=...`, used by the existing
password-recovery flow in this same file) for the implicit flow, and a
`?code=` query param for PKCE. `?t=` matches neither pattern — Supabase's
own client never inspects it, so there is no collision to guard against and
no reason to set `detectSessionInUrl: false` or change its configuration.

### Guest UI on failure

No new UI was built for the failure state. "The normal login available
beneath it" (spec §10 / D31) is satisfied by the app's existing default:
`resetToGuestUI()`'s territory (`guestAuthActions`, the header's "Sign In"
button) is the default visible state and is never touched by this code —
on failure the page simply proceeds through the same path an ordinary
signed-out visit already takes (including the guest-benefits-card
fallback), with the one toast layered on top.

---

## 4. Security review (D29-02 + chat side, combined)

Every finding named in the review's own scope, in order.

### 4.1 CORS allowlist — CRITICAL, blocks this feature end to end

**This is the finding flagged as "worth flagging loudly" across the drops
and must not be lost — full detail below, nothing summarized away.**

`alo-supabase/supabase/functions/_shared/cors.ts` (read-only this session,
confirmed unmodified):

```ts
const PROD_ORIGINS = ['https://dashboard.alowen.ai', 'https://www.alowen.ai'];
```

`alo-client-chat-full_v1/CNAME` (confirmed this session):

```
alowen.ai
```

**`https://alowen.ai` — the chat app's actual deployed origin, bare, no
`www.` — is not in `PROD_ORIGINS`.** `isOriginAllowed()` and
`corsHeaders()` both key off this exact set; a browser request from
`https://alowen.ai` carrying an `Origin` header (every real browser
request does) will not match, `corsHeaders()` returns `{}`, and the
browser — not the server, the *browser itself* — blocks the response
before any JavaScript on the page ever sees it, token valid or not.

**Confirmed this is a genuinely new gap, not a pre-existing one just
now noticed:** `therapist-signup` and `open-as-client` share this same
`cors.ts` but are both called exclusively from `dashboard.alowen.ai`
(allowlisted). `exchange-client-token` is the **first and only** caller
from the chat app's origin. This gap has been latent since the CORS module
was written and D29-03 is what newly exposes it.

**Compounding effect with ruling D31:** a browser CORS block manifests to
JavaScript as an opaque `TypeError` from `fetch()` — no status code, no
body, nothing to distinguish it from a real 401. It falls into the exact
same bare `catch {}` as every other failure mode and produces the exact
same cause-neutral toast. **This means the feature can appear to work
correctly in every respect except actually working** — the client-side
code is correct, will pass every local check in section 6, and will still
fail 100% of the time in production until the allowlist is fixed, with a
failure message that gives no hint why.

**Not fixed this session** — `alo-supabase` is read-only. The fix itself is
one line, well inside the pattern the file's own comments already
describe:

```ts
const PROD_ORIGINS = ['https://dashboard.alowen.ai', 'https://www.alowen.ai', 'https://alowen.ai'];
```

**This is the top item on the Kano sitting list (section 7) and blocks
manual testing entirely** — none of the browser-based checks in section 6's
Manual Test Checklist involving the actual exchange can produce a real
"it worked" result until this lands and the function is redeployed.

### 4.2 Token lifetime in memory

**Dashboard:** the newly-minted token exists only inside `body.url`
(D29-01), passed directly to `tab.location.replace()`, never assigned to a
longer-lived variable, never stored. Effectively zero extra lifetime beyond
the function call itself. No finding.

**Chat:** `_aloClientToken` was originally declared `const` and held in the
`DOMContentLoaded` closure for the entire remaining life of that (long-
running, never-returning-until-page-unload) async function — reachable in
memory far longer than the one `fetch()` call that needs it, even though
the server-side token is spent on the very first attempt regardless. **Fix
applied**: `const` → `let`, explicitly set to `null` immediately after the
single exchange attempt returns (success or failure alike). Folded into the
same, not-yet-committed-at-the-time implementation, so it landed in commit
`593c9f4` rather than as a separate patch.

### 4.3 Referrer leakage — considered, corrected mid-review, not fixed

**This is where the plan stated at Drop 2 was wrong, corrected on this
resume before anything was committed.**

The plan going into this resume was to add `'noopener,noreferrer'` to the
dashboard's `window.open('about:blank', '_blank')` call. Re-reading the
surrounding code before touching it surfaced two things that made that the
wrong fix:

1. **`noopener` is already handled, deliberately not via the window-
   features string.** The function's own comment: *"with the 'noopener'
   window feature window.open() always returns null, so a blocked pop-up
   would be indistinguishable from a successful one."* `tab.opener = null`
   achieves the same effect manually, while preserving a real tab
   reference the code needs for pop-up-block detection. Adding `noopener`
   to the features string would have **broken that detection outright** —
   every call would look identical to a blocked pop-up, defeating the
   `if (!tab)` check entirely.
2. **`noreferrer` in that same features string causes the identical
   problem** — the spec behavior is that `window.open()` returns `null`
   when either `noopener` or `noreferrer` is present as a window feature.
   So the fix as planned would have broken the SAME detection mechanism a
   second way, for a benefit that turns out not to exist:
   - The Referer header describes the **source** page's URL, never the
     destination's. The chat token lives in `body.url` — the destination —
     so it was never at risk via Referer regardless of any header setting.
   - The only thing a Referer header on this navigation could disclose is
     that the visitor came from `dashboard.alowen.ai`. Under the modern
     browser default (`strict-origin-when-cross-origin`, and there is no
     `<meta name="referrer">` override in either app widening this —
     checked), a cross-origin navigation like
     `dashboard.alowen.ai → alowen.ai` sends **origin only** — no path, no
     query string. Not sensitive, and not the token.
   - No `Referrer-Policy` HTTP response header is achievable here anyway —
     both apps are static GitHub Pages sites with no server-side header
     control.

**Conclusion: not fixed, correctly.** Adding it would have broken a
working, deliberately-engineered detection mechanism in exchange for
protection against a leak that doesn't exist under the platform's actual
constraints. This is exactly the "no speculative hardening beyond the
contract" the review's own scope forbids.

### 4.4 Error-message contents

**Dashboard:** bare `catch {}`, one fixed toast string, nothing
interpolated — confirmed by reading the code, matching its own inline
comment ("nothing that could carry the token is interpolated here").

**Chat:** bare `catch {}`, `ALO_TOKEN_EXCHANGE_FAILURE_MESSAGE` (a `const`,
never built from any response data), nothing logged, nothing written to
the DOM. Mirrors the dashboard's pattern exactly, deliberately.

One caveat, stated rather than glossed over: `supabase.auth.verifyOtp()`'s
*internal* SDK behavior (whether it ever logs to console on its own,
independent of this code's handling of its return value) was not
independently verified by reading the SDK's source — trusted third-party
behavior, not confirmed. Worth a note if this is ever audited more deeply.

### 4.5 Replay behaviour

A second use of an already-spent token is server-side indistinguishable
from "no such token" or "expired" — all three hit the same
`used_at IS NULL AND expires_at > now()` `WHERE` clause and return zero
rows, which the function maps to the same 401 as everything else (section
1). Client-side, the chat consumer has exactly one `fetch()` call per
invocation, invoked from exactly one call site, guarded by the token having
been present at all — there is no code path in `_aloExchangeClientToken()`
or its caller that could re-POST the same token, by design (no retry logic
anywhere in the function) and by construction (the URL param is stripped
before the exchange even starts, so a page reload after a failed or
successful exchange finds no `?t=` to re-attempt). Confirmed there is
exactly one `document.addEventListener('DOMContentLoaded', ...)` in the
file, ruling out a double-registration replay path too.

**A replay shows the user the exact same cause-neutral message as a
first-time-invalid token — D31 satisfied precisely.**

---

## 5. Docs

One addition: `alo-client-chat-full_v1/CLAUDE.md`, "Critical Warnings" —
flags the new `?t=` surface, the cause-neutral-message rule, and points at
this report. Commit `39a0780`.

**Not durably recorded:** ruling D31 itself has no home in
`alo-supabase/docs/decisions.md` (the usual place for a ruling like this),
since that repo was read-only this session. Its only committed-adjacent
record is `WORKQUEUE.local.md` (explicitly "local only, never committed" by
its own file header) and this report. **Section 7 covers what Kano should
do about that.**

**Noticed, not touched:** `alo-client-chat-full_v1/CLAUDE.md`'s "Git
workflow" section still reads *"Work directly on `main`. No feature
branches at this stage"* — stale; this entire session (and several before
it) worked on branches. This is the pre-Phase-3 content of the file (see
the Session Log deviation note on branch divergence) — `auto/phase3-audit-
fixes`, a separate, still-unmerged branch, already corrects this same
section. Not fixed here to avoid a second, conflicting edit to a line
already being fixed on another branch; flagged so the eventual merge of
both branches doesn't quietly drop one correction in favor of the other.

---

## 6. Verification

| Check | Chat | Dashboard |
|---|---|---|
| `node --check` on every `<script>` block | **2/2 OK** | **1/1 OK** |
| Every `onclick="fn("` resolves to a defined function | **35/35** | **52/52** |
| `scripts/scan-invisible.py` on every file changed this branch | **CLEAN** (`CLAUDE.md`, `dev.html`) | **CLEAN** (`dev.css`, `dev.html`) |

No `.ts`/`.sql` file changed in either repo this session (`alo-supabase`
untouched, confirmed read-only throughout).

**What this verification does NOT and cannot cover**: the actual exchange
against the live Edge Function. That requires a browser, a real therapist
account, a real `is_demo` test client, and — per section 4.1 — will not
succeed until the CORS allowlist is fixed and `exchange-client-token` is
redeployed. Every item in the Manual Test Checklist below involving an
actual click-through is therefore currently **expected to fail at the CORS
layer**, not a defect in what's been built.

---

## Manual Test Checklist

**Blocked on section 4.1 (CORS fix + redeploy) for every item that reaches
the real exchange endpoint.** Items 1-2 can be checked today; items 3+
cannot produce a real pass until then.

1. **Dashboard button presence.** Open a therapist's client list containing
   an `is_demo` test client. Confirm "Open as client" appears in both the
   list row and the detail view (spec §9), and nowhere else.
2. **Pop-up-block detection.** In a browser configured to block pop-ups,
   click "Open as client". **Expect:** a toast — *"Allow pop-ups for this
   site to open your test client."* — not a silent no-op.
3. **Happy path** (after the CORS fix lands and is redeployed): click
   "Open as client". **Expect:** a new tab opens, briefly shows
   `about:blank`, then navigates to the chat app. **Expect:** the URL bar
   settles on a `?t=`-free URL almost immediately (check history: pressing
   "back" should not return to a URL containing `?t=`). **Expect:** the
   chat app loads signed in as the test client — trust badge, threads
   button, etc., exactly as an ordinary signed-in load.
4. **Expired token.** Mint a token (open the tab), wait 61+ seconds, then
   manually navigate the SAME tab to that same `?t=` URL again (or copy it
   before it expires and paste it after). **Expect:** the one constant
   message — *"That link didn't work — ask your therapist for a new
   one."* — with the ordinary Sign In button visible beneath/around it.
5. **Replay.** Successfully complete the happy path (item 3), copy the
   now-spent URL from browser history (or from having saved it before the
   redirect), open it in a fresh incognito window. **Expect:** the exact
   same constant message as item 4 — no different wording, no hint that
   this case is "already used" versus "expired" versus "never existed."
6. **Malformed token.** Manually edit the `t=` value in a valid URL to
   something clearly wrong (truncate it, change a character). **Expect:**
   same constant message again.
7. **`?t=` + `?invite=` together** (edge case, low priority): craft a URL
   with both params present. **Expect:** the token exchange attempt fires
   first (win or lose), and the invite-code prefill/modal still appears
   correctly afterward for whichever outcome — confirm no interference
   between the two handlers.
8. **DevTools sweep, all of items 3-7**: Network tab — confirm no
   plaintext token appears in any REQUEST outside the one POST to
   `exchange-client-token`'s own body, and no token/token_hash appears in
   any RESPONSE body logged to console. Console tab — confirm no
   `console.*` output anywhere in this flow mentions a token value.
   Application/Storage tab — confirm no token or token_hash was ever
   written to `localStorage`, `sessionStorage`, or a cookie.

---

## Data for architect

Facts a future planning session needs, not simply a to-do:

- **The CORS allowlist gap (section 4.1) is the only reason this feature
  doesn't work today.** Every client-side line is written and passes every
  check available without a browser. One line in
  `alo-supabase/_shared/cors.ts` plus a redeploy of
  `exchange-client-token` is the entire remaining path to a working
  feature.
- **D31's cause-neutral-message ruling has no durable home yet.** It
  exists in `WORKQUEUE.local.md` (not committed, not durable) and this
  report. If `alo-supabase/docs/decisions.md` is where rulings normally
  live, this one is missing from it.
- **This session made a considered decision NOT to add
  `noopener`/`noreferrer`** to the dashboard's `window.open()` call,
  because doing so would break existing pop-up-block detection for a
  Referer-leak protection that isn't needed given how Referer headers
  actually work. If a future review re-raises this finding, section 4.3
  has the full reasoning already worked out — no need to re-derive it.
- **Two branches with unreconciled `CLAUDE.md` edits to the same file**
  (`alo-client-chat-full_v1/CLAUDE.md`): this branch's docs addition layers
  on pre-Phase-3 content, while `auto/phase3-audit-fixes` (unmerged, from
  an earlier session) corrects that same file's Stack/File-Structure
  sections and its stale "work directly on main" line. An ordinary merge
  will reconcile both once both land on `main` — nothing special needed,
  just worth knowing before either merge is done.

---

## KANO SITTING LIST — in execution order

1. **Fix the CORS allowlist** (section 4.1) — add `'https://alowen.ai'` to
   `PROD_ORIGINS` in `alo-supabase/supabase/functions/_shared/cors.ts`,
   redeploy `exchange-client-token`. **Nothing else on this list produces a
   real result until this is done.**
2. **Record ruling D31** in `alo-supabase/docs/decisions.md` (or have a
   future alo-supabase-scoped session do it) — its only other home is a
   file that is explicitly never committed.
3. **Run the Manual Test Checklist above**, items 3 onward, once #1 lands —
   or hand it to Cowork.
4. **Merge `alo-client-chat-full_v1` → `auto/open-as-client`.** 3 commits
   on top of `auto/phase2b-surfaces` (already merged): D29-03, the docs
   addition. Note the `CLAUDE.md` reconciliation with
   `auto/phase3-audit-fixes` (data-for-architect, above) when merging both.
5. **Merge `alo-dashboard` → `auto/open-as-client`.** 1 commit
   (`bb09efa`, D29-01) on top of `auto/phase2b-surfaces` (already merged).
6. Once merged and the CORS fix has landed and been redeployed, promote to
   production per each repo's normal promotion process — this session made
   **no** production changes anywhere.
