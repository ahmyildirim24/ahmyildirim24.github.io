# AI Agent Prompt — Auth Callback Page for Where2Go (GitHub Pages)

## Context

I have a Flutter mobile app called **Where2Go** (package: `com.where2go.app`). I've set up Supabase email verification and password reset with PKCE flow. The auth emails (verify email, reset password) redirect to:

```
https://aeydevstudio.me/auth/callback?code=...&type=signup
```

My portfolio site **aeydevstudio.me** is already hosted on **GitHub Pages**. I need you to create a callback landing page that:

1. **Tries to open the app** on the user's device via a custom URL scheme (`where2go://login-callback`)
2. **Shows a friendly fallback** if the app can't be opened (different device, desktop browser, app not installed)
3. **Handles error states** gracefully (expired links, invalid tokens)

---

## Task: Create `/auth/callback/index.html`

Create the file at `auth/callback/index.html` in the repository root.

### Requirements

#### Design
- **Clean, modern, mobile-first.** Use the Where2Go brand colors:
  - Dark navy background: `#023047`
  - Light blue: `#8ecae6`
  - Teal: `#219ebc`
  - Amber accent: `#FFB703`
  - Orange gradient end: `#FB8500`
  - Error red: `#F87171`
  - Success green: `#4ADE80`
  - Text primary: `#f1f5f9`
  - Text secondary: `#94a3b8`
  - Text tertiary: `#64748b`
  - Card background: `#02374f`
  - Card border: `rgba(142, 202, 230, 0.2)`
- **Card-based centered layout**, rounded corners (24px), shadow
- **Brand name** at the top: "WHERE2GO" in uppercase, amber `#FFB703`, small letter-spaced label

#### Three States (show one at a time)

**State 1: Loading (shown immediately on page load)**
- Animated spinner (border-top amber)
- Text: "Opening the app…"
- This state is visible only briefly while JS attempts to open the app

**State 2: Success / Fallback (shown after ~1.5s or if the app didn't open)**
- Green checkmark icon (✓) inside a circle with `rgba(74, 222, 128, 0.15)` background
- Title: "Email Verified!" (for signup) or "Link Received!" (for password reset)
  - Determine type from URL params: `type=signup` → verified, `type=recovery` → password reset
- Body text: "Your email has been confirmed. Open the app and sign in to continue." (signup) or "Open the app to set your new password." (recovery)
- **"Open Where2Go" button** — amber gradient (`#FFB703` → `#FB8500`), dark navy text, rounded (14px), full-width
  - Links to the deep link URL (see JavaScript logic below)
- Smaller instruction text below: "If the app does not open, make sure Where2Go is installed and try the button above."

**State 3: Error (shown when URL contains error params)**
- Red X icon (✕) inside a circle with `rgba(248, 113, 113, 0.15)` background
- Title: "Link Expired"
- Body: "This link is invalid or has already been used. Open the app to request a new one."
- **"Open Where2Go" button** — same styling, links to `where2go://login-callback`

#### JavaScript Logic

```
On page load:
1. Parse URL search params AND hash fragment params
   - const params = new URLSearchParams(window.location.search)
   - const hash = new URLSearchParams(window.location.hash.replace(/^#/, ''))
   
2. Check for errors:
   - If params or hash contains 'error' or 'error_code' → show Error state, stop
   
3. Build deep link URL:
   - Start with: 'where2go://login-callback'
   - Append all query params from the current URL (so the Supabase SDK on the device can exchange the PKCE code)
   - Include both search params and hash fragment params
   
4. Set the "Open Where2Go" button href to the deep link URL

5. Determine type from params to customize success message:
   - params.get('type') === 'recovery' → password reset messaging
   - Otherwise → email verification messaging
   
6. Attempt to open the app:
   - window.location.href = deepLinkUrl
   - Record timestamp before attempt
   
7. After 1500ms timeout:
   - If still on page → show Success/Fallback state (user can tap button manually)
```

#### Constraints
- **No external dependencies.** Pure HTML/CSS/JS. No frameworks, no CDNs.
- Page must be functional even without JavaScript (buttons visible via noscript or default visible state)
- Keep the file self-contained (single HTML file with inline CSS and JS)
- Responsive — looks good on mobile and desktop

---

## Verification After Deployment

1. `https://aeydevstudio.me/auth/callback` should render the page (HTTP 200)
2. `https://aeydevstudio.me/auth/callback?error=access_denied&error_code=otp_expired` should show the error state
3. `https://aeydevstudio.me/auth/callback?code=test123&type=signup` should show loading → success state
4. On a device with the app installed, the deep link should attempt to open the app

---

## Constraints

- Do NOT break existing portfolio site pages/styles or the existing `/route` fallback page.
- Do NOT remove or modify any existing files.
- Keep the new file self-contained.
