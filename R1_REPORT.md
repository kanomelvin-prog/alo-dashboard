# R1_REPORT — therapist signup rewired to the Edge Function

**Repo:** `alo-dashboard` (PUBLIC). **Branch:** `auto/r1-signup-rewire` from `main` @ `67cf877`.
**Date:** 2026-09-08.
**Files changed: `dev.html` only.** `index.html`, `styles.css` and `dev.css` are untouched — verified with `git diff --quiet` on each.
**Not merged. Not pushed to main. Nothing deployed.**

> **Branching note.** This repo's `CLAUDE.md` says "Work directly on `main`. No
> feature branches." The session instruction was explicit branch-only work, so
> that instruction was followed and the repo convention deliberately set aside
> for this change. Flagging it rather than silently picking one.

---

## 1. What changed, in one paragraph

`handleSignup()` no longer POSTs to `/auth/v1/signup` and no longer writes
`profiles` or `therapists` from the browser. It POSTs once to
`/functions/v1/therapist-signup`, checks the status explicitly, and on `201`
signs the user in with the password grant and runs the existing post-login
checks. On anything other than `201` it renders the function's own message in
the form's existing error element and **never reaches `showApp()`**. A
`license_state` `<select>` was added to the form because the function requires
it. No fire-and-forget `fetch` remains anywhere in the signup path.

---

## 2. The extracted function contract

Read from `~/code/alo-supabase` at `auto/sb06-trigger-retirement`:
`supabase/functions/therapist-signup/index.ts`, `_shared/validate.ts`,
`_shared/cors.ts`.

**Endpoint:** `POST {SUPABASE_URL}/functions/v1/therapist-signup`
**Headers:** `apikey: <anon>`, `Authorization: Bearer <anon>`, `Content-Type: application/json`
(`verify_jwt = false` for this function, so the anon key is not an auth gate — it is what the platform gateway expects.)

**Request body — all four fields required, exactly these names:**

| Field | Rule |
|---|---|
| `display_name` | string, trimmed, **2–80** characters, no control characters |
| `email` | string, trimmed and lowercased server-side, RFC-shaped, ≤254 chars, local part ≤64 |
| `password` | string, **12–72** characters, never trimmed, rejected if in the common-password list |
| `license_state` | string, trimmed and uppercased server-side, must be one of **56** codes (50 states + DC + PR, GU, VI, AS, MP) |

**Responses:**

| Status | Body | Meaning |
|---|---|---|
| **201** | `{"ok":true,"therapist_auth_id":"<uuid>"}` | Success — the only success |
| 400 | `{"error":"validation","field":"<name>","message":"<safe text>"}` | Field-level failure. The message never contains the submitted value, so it is safe to show verbatim |
| 403 | `{"error":"forbidden"}` | `Origin` not on the allowlist |
| 405 | `{"error":"method_not_allowed"}` | Non-POST |
| 409 | `{"error":"exists"}` | Email already registered |
| 500 | `{"error":"internal","request_id":"<uuid>"}` | Server-side failure; the id is safe to surface and is what support needs |
| 204 | *(empty)* | CORS preflight |

**CORS — the one that will bite during testing.** The allowlist is
`https://dashboard.alowen.ai` and `https://www.alowen.ai`, with an empty
dev-origins list. **A request from `file://` or `localhost` gets a 403 before
any validation runs.** `dev.html` is served from the production origin on
GitHub Pages, so testing at `https://dashboard.alowen.ai/dev.html` works
without any change to the function. Test there and nowhere else.

**Two behaviour changes that follow from the contract:**

1. **No confirmation email.** The function creates users with `email_confirm`
   already set, so the account is live immediately and the user is signed in on
   the spot. The old "Account created! Check your email…" branch is gone. See
   Q1.
2. **Password minimum rises from 6 to 12.** That is the function's rule, not a
   choice made here. The form placeholder and hint were updated to match.

---

## 3. Changes to `dev.html`

### 3.1 Form markup

- Password placeholder `Min. 6 characters` → `Min. 12 characters`, plus a
  `<span class="form-hint">` using the class already defined in `dev.css:961`.
- New **License state** `<select id="signupLicenseState">` in a standard
  `.form-group`, with a `Select a state` placeholder option and all 56 codes.

The 56 codes were **generated from `validate.ts` itself**, not retyped, so the
values cannot drift from the validator. They are ordered by state name for
usability; the `value` attributes are the two-letter codes the function expects.

**No CSS was needed.** `.form-group select` is already styled at `dev.css:942-958`,
so `dev.css` is unchanged and **no cache-buster increment is required** — the
`?v=32` reference stays as it is.

### 3.2 `handleSignup()` — replaced

Deleted: the `/auth/v1/signup` call, the `redirect_to` parameter, the
`data: { display_name, role: 'therapist' }` metadata, the `access_token`
absent/confirmation branch, and **both** client-side inserts
(`/rest/v1/profiles` and `/rest/v1/therapists`) with their swallowing
`try/catch` blocks.

Added: one POST to the Edge Function, an explicit `res.status !== 201` check, and
`await completeSignIn(email, password)` on success. Client-side pre-checks now
mirror the validator (required fields, name 2–80, password 12–72) so ordinary
mistakes cost no round trip — the server re-validates everything regardless.

A comment block above the function records *why* the old inserts were removed,
so nobody reintroduces them:

> the `profiles` RLS INSERT policy pins role to `'individual'`, the `therapists`
> insert set `id` instead of `auth_user_id`, and neither checked its response —
> `fetch` does not reject on 4xx, so both failed silently and the UI called
> `showApp()` anyway.

### 3.3 `signupErrorMessage(res)` — new

Maps a non-201 to user-facing text. Never throws: a missing or non-JSON body
falls through to a status-based message. Shows the validator's `message`
verbatim for 400 (safe by contract), a "sign in instead" line for 409, and the
`request_id` for 500 so support has a reference.

### 3.4 `completeSignIn(email, password)` — new, shared

Password grant → session → `localStorage` → therapist-role check →
`isVerificationLocked` → `showApp()`. **`handleLogin()` was refactored to call
it**, so there is exactly one implementation of "you are signed in" rather than
two copies drifting apart. The steps and their order are unchanged from what was
inline in `handleLogin`.

Two deliberate micro-changes came with the move, both improvements, both
affecting login as well as signup:

- The error-body `res.json()` is now wrapped in `try/catch`. Previously a
  non-JSON error response threw a `SyntaxError` and the user saw
  `Unexpected token …`; now they see `Login failed`.
- Network failures in the signup path are mapped to
  `Could not reach the server. Check your connection and try again.` instead of
  the raw `Failed to fetch`.

**This is the one judgment call worth your review** (Q2). The alternative was to
duplicate the post-login sequence inside `handleSignup` and leave `handleLogin`
byte-identical. Duplication was rejected because this file already suffers from
exactly that failure mode — see §5.

### 3.5 SILENT-1 is gone

Every `fetch` in the signup path has its status checked and every failure path
ends in a visible error. `showApp()` is unreachable unless the function returned
201 **and** the password grant succeeded **and** the profile really has
`role = 'therapist'`.

---

## 4. Verification run

| Check | Result |
|---|---|
| `index.html` / `styles.css` / `dev.css` modified? | **No** — `git diff --quiet` clean on all three |
| Inline JS parses (`node --check` on the extracted `<script>`) | **SYNTAX OK** — and the pre-edit file was checked the same way as a control, so the test is meaningful |
| Zero-width / NBSP / smart-quote scan, `dev.html` + `dev.css` | **0 hits** |
| Tag balance | `<div>` 439/439, `<select>` 9/9, `<option>` 93/93, `<script>` 1/1 |
| License select contents | 57 options = 56 codes + placeholder; codes generated from `validate.ts` |
| Anon-key occurrences | `dev.html` **4**, `index.html` **4** — unchanged, matches baseline |
| Distinct JWT literals in `dev.html` | **1** (the public anon key, already present) |
| `sk-ant` / `service_role` strings | **none** |

**Not verified: nothing was executed in a browser.** No signup was performed,
no network call was made, and the Edge Function was not invoked. Parse-level and
static checks only — the behavioural proof is §6, which is yours to run.

---

## 5. Drift found between `dev.html` and `index.html`

Asked for in the brief. `dev.html` is **ahead** of `index.html` in three
unpromoted places. All three predate this session.

| # | Drift | Location | Fate |
|---|---|---|---|
| 1 | `?redirect_to=` on the signup call | old `handleSignup` | **Moot** — inside code this change deletes |
| 2 | `verify_deadline` on the `therapists` insert | old `handleSignup` | **Moot** — inside code this change deletes |
| 3 | `isVerificationLocked` / `showPendingVerification` block after the role check | `handleLogin` | **Rides along on promotion** — a real unpromoted feature |

The signup **form markup** was byte-identical between the two files before this
change, and `setButtonLoading` still is.

**Drift 3 matters.** Promotion in this repo is a whole-file copy
(`dev.html` → `index.html`), so the verification-lockout feature lands in
production whenever this is promoted — with or without the R1 change. That is
pre-existing and not caused by this work, but it should be a conscious decision
rather than a surprise. Confirm you want lockout live before promoting.

---

## 6. Manual test steps — run at `https://dashboard.alowen.ai/dev.html`

**Prerequisites, both mandatory:**

1. The `therapist-signup` Edge Function must be **deployed**. It was, as of the
   2026-09-07 e2e.
2. **Test on the live dev URL, not a local file.** `file://` and `localhost`
   are not on the CORS allowlist and will 403 before validation runs (§2).

Migration `20260908150000` does **not** need to be applied first — this change
works against the current trigger too, because the Edge Function bypasses it
either way. Order between R1 and that migration is free.

---

**Test 1 — happy path.**
Full Name `Dr Test Eleven`, Email `kano+test11@gmail.com`, Password a fresh
20-character string, License state `California`. Submit.

*Pass:* the button shows a spinner and `Creating account...` and is disabled;
then the dashboard loads signed in as that therapist. No error text appears.
In DevTools → Network: one POST to `therapist-signup` returning **201**, then one
POST to `token?grant_type=password` returning 200. **There must be no POST to
`/rest/v1/profiles` or `/rest/v1/therapists`.**
*Fail:* any 4xx/5xx, or the dashboard loading while an error is also visible.

**Test 2 — duplicate email.** Repeat Test 1 with the same address.

*Pass:* stays on the form; error reads
`An account with that email already exists. Sign in instead.` Network shows
**409**. The dashboard does **not** load.
*Fail:* the dashboard loads, or the error is a raw JSON blob.

**Test 3 — short password (client-side).** New address, password `short123`.

*Pass:* error `Password must be at least 12 characters` appears **with no
network request at all** — this is the local pre-check.
*Fail:* a request is sent, or the message quotes the password.

**Test 4 — weak password (server-side).** New address, password `password1234`
(12 chars, so it passes the local check and reaches the function).

*Pass:* one POST returning **400**; error reads
`password is too common; choose a less predictable one`. The password itself is
never echoed.
*Fail:* a 201, or the submitted password appearing anywhere on screen.

**Test 5 — missing field.** Fill everything except **License state**.

*Pass:* `Please fill in all fields`, no network request.
*Fail:* a request is sent with an empty `license_state`.

**Test 6 — function unreachable.** DevTools → Network → set throttling to
**Offline**, then submit a valid form.

*Pass:* error reads `Could not reach the server. Check your connection and try
again.` The button returns to `Create Account` and is re-enabled. The dashboard
does **not** load.
*Fail:* a raw `Failed to fetch`, a stuck spinner, or `showApp()` running.

**Test 7 — login still works (regression, because §3.4 refactored it).**
Sign in with an existing therapist account.

*Pass:* identical to before — dashboard loads, or the pending-verification
screen if that account is locked.
*Fail:* any change in login behaviour. This is the one thing this change could
break that is not signup.

**Test 8 — non-therapist rejected.** Sign in with a client (individual) account.

*Pass:* `This account is not a therapist account`; dashboard does not load.

**Cleanup:** `npx tsx scripts/teardown-therapist.ts kano+test11@gmail.com`
in the `alo-supabase` repo, after a `--dry-run`.

---

## 7. Promotion diff summary — what moves to `index.html` later

Promotion is a whole-file copy of `dev.html` → `index.html`, then rewriting the
stylesheet reference from `dev.css?v=32` to `styles.css?v=29` (per `CLAUDE.md`).
**`styles.css` needs no change and no version bump** — this work touched no CSS.

What production gains:

1. Signup calls the Edge Function; both client-side inserts are gone.
2. `license_state` select; password minimum 12; new form hint.
3. `completeSignIn` + `signupErrorMessage`; `handleLogin` refactored onto the
   shared helper.
4. No confirmation-email step — new therapists are signed in immediately.
5. **Drift 3 rides along**: the verification-lockout branch in `handleLogin`
   (§5). Decide on this deliberately.
6. Drift 1 and 2 disappear, since they lived in the deleted code.

**Do not promote until Test 1–8 pass on the dev URL.**

---

## 8. Open questions

- **Q1 — no confirmation email is now sent on signup.** The function creates the
  user pre-confirmed, so the address is never proven to belong to the person.
  Previously GoTrue sent a confirmation link. Acceptable for pilot, or should
  the function be changed to send one? This is a product decision and a change
  to `alo-supabase`, not something to paper over here.
- **Q2 — the `handleLogin` refactor (§3.4).** One shared `completeSignIn` versus
  leaving login byte-identical and duplicating the sequence in signup. I chose
  sharing; say the word and I will split them.
- **Q3 — license state is collected but not shown back.** The function stores it
  on the `therapists` row. Nothing in the dashboard displays or edits it yet, so
  a typo is currently uncorrectable from the UI.
- **Q4 — branch versus `main`.** This repo's `CLAUDE.md` mandates working on
  `main`. This session was branch-only by instruction. Merge, or adopt branches
  for this repo going forward?
- **Q5 — the 6→12 character password change affects the sign-up form only.**
  Existing therapists with shorter passwords keep them and can still log in.
  Worth a forced reset before pilot, or leave it?
