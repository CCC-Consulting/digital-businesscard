# Setup guide

Your AI assistant can write all the code, but it can't click "Create
account" for you. Here's the rest, in order. You don't need to be a
developer to get through this — about an hour, start to finish.

If GitHub, GitHub Actions, or Azure are new concepts to you, read
**[the architecture diagrams](04_ARCHITECTURE.md)** first — it'll make the
steps below click faster.

## 1. Create a free GitHub account

1. Go to [github.com](https://github.com) and sign up (free tier is all you
   need).
2. Verify your email address.

## 2. Create a personal repository

1. Click **New repository** from your GitHub homepage.
2. Give it a name (e.g. `my-business-card`).
3. **Public** or **private** both work fine with Azure Static Web Apps.
4. Don't add a README/gitignore yet if your AI assistant is going to
   `git init` and push the generated project itself — otherwise you'll need
   to `git pull` first to merge histories.
5. Once your assistant has generated the site locally, get it into this
   repo:
   ```
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```

## 3. Create a free Azure account

1. Go to [azure.microsoft.com](https://azure.microsoft.com) and sign up for
   a free account.
2. The **Static Web Apps Free tier** includes: 100GB bandwidth/month, 2
   custom domains, 3 staging environments — plenty for a personal card, at
   no cost.

## 4. Create the Static Web App and connect it to GitHub — the easy path

This is the fastest way to get live, and what to start with:

1. In the Azure Portal, search for **Static Web Apps** → **Create**.
2. Pick your subscription and resource group (or create a new one, e.g.
   `rg-my-card`).
3. Give the app a name, pick a region close to you, and select the **Free**
   plan.
4. Under **Deployment details**, choose **GitHub**, sign in, and pick your
   repository and the `main` branch.
5. Set **Build presets** to "Custom", `App location` to `/` (or wherever
   your `index.html` lives), and leave `Output location` blank — this is a
   static site, no build step.
6. Click **Create**. Azure automatically:
   - Generates a deployment secret and adds it to your GitHub repo's
     secrets for you.
   - Commits a ready-made GitHub Actions workflow into your repo that
     deploys on every push to `main`.
7. Wait a couple of minutes, check the **Actions** tab in your repo — once
   the workflow run finishes green, your card is live at the
   `*.azurestaticapps.net` URL shown in the Azure Portal.

**A more advanced path exists**, and it's what the reference build actually
uses: instead of a stored deployment secret, set up **OIDC federated
credentials** so GitHub Actions authenticates to Azure with a short-lived
token instead of a long-lived stored secret, and define the Static Web App
as Bicep/infrastructure-as-code instead of clicking through the Portal.
Worth doing once you're comfortable — nothing sensitive is stored in
GitHub at all, and your infrastructure is reproducible from code. See
[how this authentication flow works](04_ARCHITECTURE.md#3-how-github-actions-authenticates-to-azure--the-advanced-path)
for the diagram. If you want this, tell your AI assistant "set this up
with OIDC and Bicep instead of a stored deployment token."

## 5. A custom domain (optional)

If you own a domain:

1. In the Azure Portal, on your Static Web App, go to **Custom domains** →
   **Add**.
2. Add a CNAME record at your DNS provider:
   `card.yourname.com → <your-app>.azurestaticapps.net`.
3. Once the CNAME is live (minutes to a few hours to propagate), validate
   the domain in Azure — it auto-issues a free SSL certificate.

Double-check the exact spelling of your subdomain everywhere it appears
(DNS record, Azure config, any hardcoded links in your HTML) — see
[Lessons learned](05_LESSONS-LEARNED.md) for why this specific mistake is
easier to make than it sounds.

## 6. How the content-feed automation is set up

If you asked for a "latest articles" or "latest videos" section, here's
what your AI assistant should have built, and why (see also the
[diagram of this loop](04_ARCHITECTURE.md#4-how-the-content-refresh-automation-works)):

- **The problem:** if your page's own JavaScript tries to `fetch()` an RSS
  feed or a YouTube feed directly from someone else's domain, the browser
  blocks it unless that domain explicitly allows cross-origin requests
  (CORS) — and most blogs, and YouTube itself, don't allow this for
  arbitrary websites. You'd get a silent failure with no visible error.
- **The fix:** a **GitHub Actions workflow** — a small automated script
  GitHub runs on a schedule — fetches the feed *from GitHub's servers*
  (where CORS doesn't apply, since it's not a browser), parses out the
  latest few items, and commits the result as a small JSON file (e.g.
  `articles.json`) directly into your repository.
- Your page's JavaScript then fetches that JSON file **from your own site**
  (`fetch('articles.json')`) — always same-origin, so it always works.
- The workflow runs on a `schedule:` (cron syntax, e.g. `0 6 * * *` for
  daily at 06:00 UTC) plus `workflow_dispatch:` so you can also trigger it
  manually from the GitHub Actions tab any time, without waiting for the
  schedule.
- Zero new infrastructure, zero cost, zero backend server — just a
  scheduled script and a static file.

## 7. Physical NFC card / QR code (optional, no code involved)

- A QR code pointing at your card's URL can be generated with any free QR
  generator and printed, added to an email signature, or put on a physical
  card.
- NFC cards/tags that can be programmed to open a URL on tap are available
  cheaply from various suppliers — this is a hardware purchase, not a
  coding step, and works with any URL once your card is live.

---

Once you're live, read **[Lessons learned](05_LESSONS-LEARNED.md)** — a short
list of real bugs hit building the reference card, already accounted for in
the prompt, worth understanding if you customize further.
