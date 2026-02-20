# AI Agent Prompt — GitHub Pages Deep Link Setup for Where2Go

## Context

I have a Flutter mobile app called **Where2Go** (package: `com.where2go.app`). I just implemented a route sharing feature that generates deep links in this format:

```
https://aeydevstudio.me/route?id=<uuid>
```

My portfolio site **aeydevstudio.me** is hosted on **GitHub Pages**. I need you to add the necessary files so that:

1. **Android App Links verification** works (so the app opens directly without a disambiguation dialog).
2. **A browser fallback page** is shown when someone clicks the link but doesn't have the app installed.

---

## Task 1: Android App Links — `assetlinks.json`

Create the file `/.well-known/assetlinks.json` at the root of the repository with this content:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.where2go.app",
      "sha256_cert_fingerprints": [
        "SHA256_PLACEHOLDER"
      ]
    }
  }
]
```

> **IMPORTANT:** Replace `SHA256_PLACEHOLDER` with the actual SHA-256 fingerprint I will provide. If I haven't provided it yet, leave the placeholder and add a comment reminding me.

### GitHub Pages `.well-known` Serving

GitHub Pages does NOT serve dotfiles/dot-folders by default. To fix this:

- Make sure there is a `_config.yml` (Jekyll config) at the repo root that includes:

```yaml
include:
  - .well-known
```

- If the site already uses a `_config.yml`, just append the `include` directive. Don't overwrite existing config.

---

## Task 2: iOS Universal Links — `apple-app-site-association` (Optional/Future)

Create `/.well-known/apple-app-site-association` (no file extension) with:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.where2go.app",
        "paths": ["/route*"]
      }
    ]
  }
}
```

> Replace `TEAMID` with the actual Apple Developer Team ID when available. For now, use `TEAMID_PLACEHOLDER` as a placeholder.

This file must be served with `Content-Type: application/json`. GitHub Pages handles this automatically for files without extensions if configured correctly.

---

## Task 3: Browser Fallback Page — `/route/index.html`

When someone opens `https://aeydevstudio.me/route?id=xxx` in a browser (not in the app), they should see a nice landing page. Create `/route/index.html`:

### Requirements:
- **Design:** Clean, modern, mobile-first. Use the Where2Go brand colors:
  - Dark navy: `#023047`
  - Light blue: `#8ecae6`
  - Teal: `#219ebc`
  - Amber accent: `#ffb703`
  - Orange: `#fb8500`
- **Content (top to bottom):**
  1. Where2Go logo area (just the text "Where2Go" in a nice heading, or an SVG placeholder)
  2. Message: "Someone shared a travel route with you!"
  3. A brief explanation: "Download Where2Go to view this route and discover amazing places."
  4. **"Open in App" button** — links to the deep link itself (`intent://` scheme for Android):
     ```
     intent://aeydevstudio.me/route?id=ROUTE_ID#Intent;scheme=https;package=com.where2go.app;end
     ```
     Extract `ROUTE_ID` from the current URL's `id` query parameter using JavaScript.
  5. **"Get it on Google Play" button** — links to:
     ```
     https://play.google.com/store/apps/details?id=com.where2go.app
     ```
  6. Footer: "© 2026 Where2Go" or similar.

- **JavaScript logic:**
  - On page load, read `id` from `window.location.search`.
  - If no `id` param, show a generic "Invalid link" message.
  - Attempt to open the app via the intent URL automatically after a short delay (1-2 seconds).
  - If the app doesn't open (user stays on page), the buttons remain visible as fallback.

- **Mobile detection:** Basic `navigator.userAgent` check. If desktop, show a different message like "Open this link on your phone to view the route in Where2Go."

- **No external dependencies.** Pure HTML/CSS/JS. No frameworks, no CDN links.

---

## Task 4: Verify Directory Structure

After all changes, the repo structure related to deep links should look like:

```
repo-root/
├── _config.yml           (with include: [.well-known])
├── .well-known/
│   ├── assetlinks.json
│   └── apple-app-site-association
├── route/
│   └── index.html
└── ... (existing portfolio files)
```

---

## Constraints

- Do NOT break existing portfolio site pages/styles.
- Do NOT remove or modify any existing files unless adding the `include` directive to `_config.yml`.
- Keep all new files self-contained.
- The fallback page must work without JavaScript (buttons should still be visible, just auto-redirect won't work).
- Ensure all JSON files are valid JSON (no trailing commas, proper formatting).

---

## Verification After Deployment

Once pushed to GitHub Pages, verify:

1. `https://aeydevstudio.me/.well-known/assetlinks.json` returns valid JSON (HTTP 200, not 404).
2. `https://aeydevstudio.me/route?id=test` shows the fallback page.
3. Use Google's verification tool: `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://aeydevstudio.me&relation=delegate_permission/common.handle_all_urls`
