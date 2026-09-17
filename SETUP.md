# OR Booking Engine — Frontend Template

This is the static-site half of the OR Booking Engine. It's a template: this
repo alone does nothing until it's paired with a GAS backend and pointed at
that backend's `/exec` URL.

Use GitHub's **"Use this template"** button on this repo (not Fork) to make a
clean copy with no shared git history — that's the right move for each new
person or client who wants their own instance.

## What this fixes

Any Google Apps Script web app served from `script.google.com` shows a
permanent, un-removable grey Google banner. This repo is the frontend, hosted
off-Google, that eliminates it. The GAS project behind it does nothing but
serve JSON now (see the paired GAS project's `Code.gs`) — no rendered HTML.

## What you need before you start

1. A GAS project already migrated to JSON-only `doGet`/`doPost` (see
   `Code.gs` in the OR Booking Engine GAS project — copy that pattern if
   you're standing up a new backend from scratch).
2. That GAS project deployed as a web app, and its `/exec` URL copied from
   **Deploy → Manage deployments**.

## Setup (2 files to edit)

1. Open `index.html`. Find this line near the top of the `<script>` block:
   ```js
   var API_BASE = '__REPLACE_WITH_YOUR_GAS_EXEC_URL__';
   ```
   Replace the placeholder with your GAS project's `/exec` URL.
2. Open `setup/index.html`. Find the same `API_BASE` line and make the same
   replacement.
3. Commit.

That's the only required edit. Everything else — copy, colors, the OR
availability logic — lives server-side in the GAS project, not here.

## Deploy to Netlify

**Fastest (no CLI):**
1. [app.netlify.com](https://app.netlify.com) → **Add new site → Deploy manually**.
2. Drag this repo's folder (containing `index.html` and `setup/`) onto the
   drop zone.

**Git-linked (auto-deploys on every push):**
1. [app.netlify.com](https://app.netlify.com) → **Add new site → Import an
   existing project → Deploy with GitHub**.
2. Pick this repo (your own copy, made via "Use this template").
3. Leave the build command blank and the publish directory as `/` — this is
   a plain static site, nothing to build.

Either way, Netlify gives you a `*.netlify.app` URL immediately. Point a
custom subdomain at it from **Domain management** on the site if you want
one (Netlify gives you the CNAME target to add at your DNS host).

## Gotchas

- `apiPost` sends with `Content-Type: text/plain` on purpose — that's what
  lets the browser skip a CORS preflight against the GAS domain. The GAS
  side still parses the body as JSON. Don't change this to
  `application/json` without also handling CORS preflight in `doOptions`.
- If you rotate or redeploy the GAS project and get a new `/exec` URL,
  update `API_BASE` in both HTML files and redeploy the static site — the
  two are not automatically linked.
- Pool member names/emails belong in the GAS project's config (Script
  Properties, via the setup page), never hardcoded into these HTML files.
