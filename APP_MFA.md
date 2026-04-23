I see the gap now. You’re **not** using a session table—you’re using **HTTP‑only cookies that hold the actual IDM tokens**. The FE never sees them; the BE just reads the cookie on every request and forwards the token to the IDM for validation.

That’s a critical difference. Let me rebuild the *entire* picture from the ground up, with the exact sequence, database changes, and cookie handling.

---

## 1. What each side owns (unchanged)

### IDM (the SaaS token service)
- Database table: `applications` – `id (app_id)`, `app_name`, `allowed_ips`, `deleted`.
- Database table: `tokens` – `token_jti`, `access_token_hash` (or the full token if you store it encrypted), `refresh_token_hash`, `user_id`, `app_id`, `expires_at`, `revoked`.
- Endpoints: `POST /token`, `POST /validate`, `POST /refresh-token`.
- Every request is guarded by IP whitelist (`app_id` → allowed IPs).

### BE (your application server)
- Database table: `users` – `id`, `email`, `password_hash`, `mfa_enabled` (boolean). **No secrets, no tokens stored here.**
- Environment variables: `APP_ID`, `IDM_BASE_URL`, `TOTP_MASTER_KEY`, `TEMP_TOKEN_SECRET`.
- Endpoints: `POST /login`, `POST /login/verify-mfa`, `POST /mfa/setup`, `POST /mfa/verify-setup`, `POST /refresh-token`, `POST /logout`, plus all your protected API routes.

### FE (browser)
- No token in `localStorage` or `sessionStorage`.
- Cookies are automatically sent with every request.
- When a request returns 401, the FE automatically calls `POST /refresh-token` and then retries the original request once.

---

## 2. The complete low‑level flow with exact DB changes

I'll break it into phases. **Every single sentence below is something you physically code.**

---

### A. Login (password only, MFA disabled)

1. FE sends `POST /login` with `{ email, password }`.
2. BE queries `SELECT id, password_hash, mfa_enabled FROM users WHERE email = $1`.
3. BE runs `bcrypt.compare(password, password_hash)`.
4. If mismatch → respond `401 Unauthorized`.
5. If match and `mfa_enabled = false`:
   - BE calls **IDM** `POST /token` with body `{ app_id: <from env>, user_id: <user.id> }`.  
   - IDM checks: source IP ∈ allowed IPs for `app_id`. If not → 403.
   - IDM generates an access token (JWT, short‑lived, e.g. 15 min) and a refresh token (opaque, long‑lived, e.g. 7 days).
   - IDM inserts a row into `tokens` table: `jti, access_token_hash, refresh_token_hash, user_id, app_id, expires_at, revoked=false`.
   - IDM returns `{ access_token, refresh_token }`.
6. BE takes those tokens and **sets two HTTP‑only cookies** on the response:
   ```
   Set-Cookie: access_token=<access_token>; HttpOnly; Secure; SameSite=Strict; Path=/
   Set-Cookie: refresh_token=<refresh_token>; HttpOnly; Secure; SameSite=Strict; Path=/refresh-token
   ```
7. BE returns `200 OK` (body can be user info or empty – not needed because cookies hold the session).

**No database row created in BE.** The tokens live only in the browser cookies.

---

### B. Login (MFA enabled)

Steps 1–4 same as above.

5. If `mfa_enabled = true` after password match:
   - BE creates a `temp_state` JWT (signed with `TEMP_TOKEN_SECRET`):
     ```json
     { "sub": "<user.id>", "purpose": "mfa_challenge", "exp": "<now+5min>" }
     ```
   - BE returns `200 { mfa_required: true, temp_state }`. **No cookies set yet.**
6. FE stores `temp_state` in a JavaScript variable (not in cookie).
7. FE renders the TOTP input.
8. User types 6‑digit code → FE sends `POST /login/verify-mfa` with `{ code, temp_state }`.
9. BE verifies `temp_state` signature and expiry using `TEMP_TOKEN_SECRET`. If invalid → 401.
10. BE extracts `user_id` from temp_state.
11. BE computes `per_user_secret = HMAC-SHA256(TOTP_MASTER_KEY, user_id)`.
12. BE computes TOTP: let `t = floor(unix_now / 30)`. For `counter` in `[t-1, t, t+1]`:  
    `expected_code = truncate(HMAC-SHA1(per_user_secret, counter))`.  
    If user’s code matches any → valid.
13. If invalid → 401.
14. If valid → BE proceeds exactly like step A.5 above (call IDM `/token`, get tokens, set cookies). **Then** return `200`.

**DB change:** None during verification. The `mfa_enabled` flag was already set during setup.

---

### C. MFA Setup (one‑time enrollment)

1. FE calls `POST /mfa/setup` (user must be logged in first; we have user context from a temporary `temp_state` or from a previous password‑only temporary auth. To avoid complicating, assume this endpoint is protected by the normal cookie flow after login but before MFA is enabled. Actually, to set up MFA, the user must already be authenticated. So we need a special path: maybe the user logs in with password, gets an `mfa_setup_temp_state` that allows access only to `/mfa/setup` and `/mfa/verify-setup`.  
   *Simpler:* In your current architecture, you can require the user to have completed a normal login (maybe with a temporary access token if MFA is not yet enabled) to hit setup. We'll just describe the math.)
2. BE gets `user_id` from the authenticated request (e.g., from `temp_state` or from a short‑lived token).
3. BE computes `per_user_secret = HMAC-SHA256(TOTP_MASTER_KEY, user_id)`.
4. BE base32‑encodes `per_user_secret` → `base32_secret`.
5. BE builds the TOTP URI:  
   `otpauth://totp/AppName:email?secret=base32_secret&issuer=AppName`
6. BE returns the URI. FE converts it to a QR code and shows it.
7. User scans with authenticator app. Then enters a code.
8. FE calls `POST /mfa/verify-setup` with `{ code }`.
9. BE again computes `per_user_secret`, runs TOTP verification (steps 11‑13 from login). If match, BE updates the user record:  
   `UPDATE users SET mfa_enabled = true WHERE id = $user_id`.
10. Return success. **Now the DB has `mfa_enabled = true` for that user.**

**DB change:** `users.mfa_enabled` from false → true. No secret stored.

---

### D. Every protected request after login

1. FE makes any API call (e.g., `GET /api/data`). The browser automatically sends cookies `access_token` and `refresh_token`.
2. BE middleware for protected routes reads the `access_token` cookie.
3. BE calls **IDM** `POST /validate` with body `{ access_token }` (and implicitly IP).
4. IDM looks up the token in its `tokens` table (or verifies JWT signature and checks revocation). If token is valid & not revoked → `200`.
5. If IDM returns `200`, BE proceeds with the request. (Optionally, BE can cache the token validation for a short time to reduce IDM calls.)
6. If IDM returns `401` (token expired/invalid/revoked):
   - BE responds to FE with `401 Unauthorized` and a specific marker (e.g., body `{ error: "token_expired" }`).
7. FE’s global HTTP interceptor sees a `401` response. It does **not** redirect to login yet. Instead:
   - FE calls `POST /refresh-token` (with cookies; the browser will send the `refresh_token` cookie because we previously set its path to `/refresh-token`).
8. BE’s `/refresh-token` handler:
   - Reads `refresh_token` from cookie.
   - Calls **IDM** `POST /refresh-token` with `{ refresh_token, app_id }` (and IP checked).
   - IDM validates the refresh token against its `tokens` table (check if revoked, not expired). If valid, IDM invalidates the old token pair (or keeps rotation) and generates a new `access_token` and optionally a new `refresh_token`. It updates the `tokens` row: either deletes old entry and creates new, or marks old as revoked.
   - IDM returns `{ access_token, refresh_token }`.
9. BE sets new cookies with the new tokens (same cookie settings).
10. BE returns `200 OK` to FE.
11. FE then automatically retries the original failed request (now with the new access token cookie). The request succeeds.

**IDM DB change:** Old token revoked, new token row inserted.
**BE DB change:** None.

---

### E. Logout

1. FE calls `POST /logout`.
2. BE reads the `access_token` cookie (and optionally `refresh_token`).
3. BE calls **IDM** `POST /revoke` with the tokens (or just tells IDM to mark them revoked).
4. IDM updates its `tokens` table: sets `revoked = true` for those tokens.
5. BE clears the cookies by setting `Set-Cookie` with `Max-Age=0`.
6. Return success.

---

## 3. Summary of database changes (all points where something is written)

| When | Table | Row | Change |
|------|-------|-----|--------|
| App registered by admin | IDM `applications` | new row | INSERT with app_id, IPs |
| User enables MFA | BE `users` | existing row | SET `mfa_enabled = true` |
| Log in (password+MFA) | IDM `tokens` | new row | INSERT access/refresh token pair |
| Validate request | IDM `tokens` | read | (no write) |
| Token expired → refresh | IDM `tokens` | old row | SET `revoked = true` for old token; INSERT new token pair |
| Logout | IDM `tokens` | row(s) | SET `revoked = true` |
| MFA setup (no secret stored) | BE `users` | same row | SET `mfa_enabled = true` |

**No MFA secrets ever touched the database.** The deterministic TOTP secret lives only in the math, bootstrapped from `TOTP_MASTER_KEY` in the environment.

---

## 4. Very low‑level task list for each team

### IDM developer
- `/token` endpoint: accept `{ app_id, user_id }`, validate IP, generate JWT access token + opaque refresh token, store in `tokens` table, return both.
- `/validate` endpoint: accept `{ access_token }`, validate IP, check token in DB (not expired, not revoked), return `{ active: true/false }`.
- `/refresh-token` endpoint: accept `{ refresh_token, app_id }`, validate IP, verify refresh token, revoke old token, issue new pair, return.
- `/revoke` endpoint (for logout): accept `{ token }`, mark token revoked.
- `tokens` table: store `jti, access_token_hash, refresh_token_hash, user_id, app_id, expires_at, revoked`.

### BE developer
- Add `mfa_enabled` (boolean, default false) to `users` table.
- Environment variables: `APP_ID`, `IDM_BASE_URL`, `TOTP_MASTER_KEY` (32 random bytes), `TEMP_TOKEN_SECRET` (32 random bytes).
- Implement `POST /login`:
  - Look up user, check password.
  - If `mfa_enabled` false: call IDM `/token`, set access/refresh cookies, return 200.
  - If true: generate `temp_state` signed JWT, return `{ mfa_required: true, temp_state }`.
- Implement `POST /login/verify-mfa`:
  - Verify `temp_state`. Derive per_user_secret. Compute TOTP. If match, call IDM `/token`, set cookies.
- Implement `POST /mfa/setup`: derive per_user_secret, produce TOTP URI, return it.
- Implement `POST /mfa/verify-setup`: verify OTP, then set `users.mfa_enabled = true`.
- Middleware for protected routes:
  - If cookie `access_token` missing → 401.
  - Call IDM `/validate` with token.
  - If IDM returns 401 active=false → return 401 to FE (with specific error code).
  - Otherwise continue.
- `POST /refresh-token` handler: read refresh_token cookie, call IDM `/refresh-token`, set new cookies.
- `POST /logout`: call IDM revoke, clear cookies.
- All IDM calls carry `app_id` (from env) and are made server‑side.

### FE developer
- Remove all localStorage/sessionStorage token logic.
- In login flow: on password submission, if `mfa_required` comes back, store `temp_state` in a variable, show TOTP input, then call `/login/verify-mfa`.
- On MFA setup page: call `/mfa/setup`, show QR (use a library to render), accept code, call `/mfa/verify-setup`.
- Global `fetch`/axios interceptor:
  - On 401 response with code `token_expired` (or any 401): call `POST /refresh-token` with credentials (cookies). If refresh succeeds, retry the original request exactly once. If refresh fails (e.g., 401), redirect to login.
- Logout button: call `POST /logout`, then redirect to login.

---

This matches your manager’s requirements exactly:
- MFA logic is fully in BE, with zero secrets in DB.
- IDM is a pure token service, guarded by IP.
- Tokens are never exposed to JavaScript.
- Refresh flow is automatic and transparent to the user.

You now have the precise low‑level blueprint to build against.
