# AI Agent Prompt — Update Auth Callback for Password Recovery (GitHub Pages)

## Context

I have an existing `auth/callback/index.html` on my **aeydevstudio.me** GitHub Pages site. It currently handles email verification callbacks from Supabase and shows an "Email Verified!" message with a deep link button to open the **Where2Go** mobile app.

**Problem:** Password reset links now also redirect here but the page always shows "Email Verified!" regardless of flow type. Password reset redirects now include a `flow=recovery` query parameter (e.g., `?flow=recovery&code=AUTH_CODE`).

## Task: Update `/auth/callback/index.html`

Modify the **existing** `auth/callback/index.html` file. Do NOT create a new file — update in place.

### What to Change

#### 1. Flow Detection Logic

The JavaScript currently checks for `type=recovery` from URL params. Supabase PKCE flow does **not** forward the `type` param — instead, our app passes `flow=recovery` as a custom query parameter.

Update the flow detection to check for **both**:
```javascript
const isRecovery = params.get('flow') === 'recovery' || params.get('type') === 'recovery' || hash.get('type') === 'recovery';
```

#### 2. State 2 (Success/Fallback) — Different Messages by Flow

**If recovery (`isRecovery === true`):**
- Green checkmark icon stays the same
- Title: **"Password Reset Ready!"** (instead of "Email Verified!")
- Body: **"Your identity has been confirmed. Open the app to set your new password."**
- Button: **"Open Where2Go"** (same styling, same deep link behavior)
- Instruction text: "If the app does not open, make sure Where2Go is installed and try the button above."

**If email verification (default):**
- Keep current behavior exactly as-is:
- Title: "Email Verified!"
- Body: "Your email has been confirmed. Open the app and sign in to continue."

#### 3. Deep Link Construction

Ensure the deep link URL includes **all** query params from the current URL, including `flow=recovery` if present. The deep link should look like:
```
where2go://login-callback?flow=recovery&code=AUTH_CODE
```

The app's Supabase SDK will read the `code` param from the deep link to exchange the PKCE code, and the `flow` param helps the app's router direct to the correct screen.

#### 4. Do NOT Change

- Loading state (State 1) — keep as-is
- Error state (State 3) — keep as-is
- Brand colors, layout, card design — keep as-is
- noscript fallback — keep as-is
- No external dependencies rule — keep as-is

---

## Brand Colors Reference (unchanged)

- Dark navy background: `#023047`
- Amber accent: `#FFB703`
- Orange gradient end: `#FB8500`
- Success green: `#4ADE80`
- Error red: `#F87171`
- Text primary: `#f1f5f9`
- Text secondary: `#94a3b8`
- Card background: `#02374f`

---

## Verification After Update

1. `https://aeydevstudio.me/auth/callback?code=test123` → should show "Email Verified!" (default)
2. `https://aeydevstudio.me/auth/callback?flow=recovery&code=test123` → should show **"Password Reset Ready!"**
3. `https://aeydevstudio.me/auth/callback?error=access_denied&error_code=otp_expired` → should show error state (unchanged)
4. Deep link button on recovery page should link to `where2go://login-callback?flow=recovery&code=test123`

---

## Constraints

- Do NOT break existing behavior for email verification flow
- Do NOT remove or modify any other files in the repository
- Keep the file self-contained (single HTML file, inline CSS/JS, no external deps)
- Only modify `auth/callback/index.html` — nothing else
