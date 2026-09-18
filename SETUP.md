# OR Booking Engine — Frontend Template

This repo is the static-site half of the OR Booking Engine. It's a template:
by itself it does nothing until it's paired with a GAS backend and pointed
at that backend's `/exec` URL.

Click **Fork** on this repo to make your own copy for each new person or
client who wants their own instance. A fork keeps a visible link back to
this original repo — that's normal GitHub behavior, not a problem.

## What this fixes

Any Google Apps Script web app served from `script.google.com` shows a
permanent, un-removable grey Google banner. This repo is the frontend,
hosted off-Google, that eliminates it for the page a prospect actually
sees. The GAS project behind it does nothing but serve JSON to this page —
no rendered HTML for the booking flow.

## Two URLs, two audiences

| | Booking page | Setup page |
|---|---|---|
| Lives in | this repo → Netlify | the GAS project only |
| Talks to GAS via | `fetch()`, `?action=...` → JSON | `google.script.run` (unchanged, old-style) |
| Reachable at | your Netlify URL / custom domain | the GAS `/exec` URL directly — no query string |
| Has the Google banner | No | Yes — harmless, only opened by whoever configures the instance, never a prospect |

**This repo contains only the booking page** (`index.html`). There is no
`setup/` folder — the setup page is never hosted here, never on Netlify,
and never has a public link anywhere except the GAS project's own `/exec`
URL. Don't add a static setup page back to this repo; that defeats the
point of keeping it GAS-only.

## Before you start (per new instance)

Do the GAS side first, all the way through. Only come back to this repo
once you have a working `/exec` URL.

1. Open the source GAS project. On the left-hand menu (hover the icons if
   it's collapsed, to reveal the labels), click **Overview**.
2. On that page, in the **top right corner**, click the **copy button** —
   this makes a whole new copy of the project under your own account. This
   is the new client's own project.
3. In the copy: **Editor → Services (+) → add Calendar API.** Fresh copies
   sometimes 403 on calendar access without this; do it before anything
   else, not after hitting the error.
4. **Deploy → New deployment → Web app.** Copy the `/exec` URL — you need
   it in the next step.
5. Run `runInitSetup` (function dropdown at the top of the editor → Run).
   No password to set — this just auto-detects `SCRIPT_OWNER_EMAIL` and
   records `SETUP_URL`.
6. Send the client the `/exec` URL — that's the whole setup link, no
   password. A plain visit (no query string) renders the setup page. They
   share their calendars with the `SCRIPT_OWNER_EMAIL` shown on that page,
   and fill in host/pool/hours themselves.
7. Once they've shared calendars, run `diagnoseCalendarAccess()` to confirm
   every calendar is actually readable before calling it done.

## Setup this repo (1 file to edit)

1. Open `index.html`. Find this line near the top of the `<script>` block:
   ```js
   var API_BASE = '__REPLACE_WITH_YOUR_GAS_EXEC_URL__';
   ```
   Replace the placeholder with the `/exec` URL from Step 3 above.
2. Commit.

That's the only required edit. Everything else — copy, colors, the OR
availability logic — lives server-side in the GAS project, not here.

**If you forget this step:** the page won't silently call a broken or wrong
URL. It checks for the placeholder on load and replaces the whole page with
a visible red error telling you to come back here — so a half-configured
fork fails loudly instead of quietly pointing at the wrong calendar backend.

## Deploy the booking page to Netlify

**Fastest (no CLI):**
1. [app.netlify.com](https://app.netlify.com) → **Add new site → Deploy
   manually**.
2. Drag `index.html` (or a folder containing just it) onto the drop zone.

**Git-linked (auto-deploys on every push):**
1. [app.netlify.com](https://app.netlify.com) → **Add new site → Import an
   existing project → Deploy with GitHub**.
2. Pick this repo (your own fork). First time through, this may prompt a
   one-time GitHub connection for that Netlify account.
3. Leave the build command blank and the publish directory as `/` — this
   is a plain static site, nothing to build.

Either way, Netlify gives you a `*.netlify.app` URL immediately. Point a
custom subdomain at it from **Domain management** on the site if the
client needs one (Netlify gives you the CNAME target to add at your DNS
host).

## Finding the setup link later

Just the GAS project's `/exec` URL itself, no query string — a plain visit
renders the setup page. It's also auto-recorded as the `SETUP_URL` Script
Property (Project Settings → Script Properties in that project) so it's
never something to hunt for in Deploy dialogs. See the comments in
`Setup.gs` for how that gets refreshed after a redeploy.

## Gotchas

- `apiPost` sends with `Content-Type: text/plain` on purpose — that's what
  lets the browser skip a CORS preflight against the GAS domain. The GAS
  side still parses the body as JSON. Don't change this to
  `application/json` without also handling a CORS preflight in `doOptions`.
- If you rotate or redeploy the GAS project and get a new `/exec` URL,
  update `API_BASE` in `index.html` and redeploy the static site — the two
  are not automatically linked. There's no shared ID between a GitHub repo
  and a GAS project; the only connection is that one hardcoded URL, so
  nothing catches a stale one except the placeholder guard above.
- Pool member names/emails belong in the GAS project's config (Script
  Properties, via the setup page), never hardcoded into `index.html`.
