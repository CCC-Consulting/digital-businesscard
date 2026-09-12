# The prompt

Copy everything below into your AI coding assistant (Claude Code, ChatGPT,
Cursor, or similar). Attach a screenshot of a reference card — the images in
this repo's [README](../README.md), or a screenshot from the video — for
layout and structure inspiration only.

Before you paste it: read **[Prepare your assets](01_PREPARE-YOUR-ASSETS.md)**
first. The prompt tells your assistant to interview you about your photo,
brand, and content — you'll move a lot faster if you already have answers
ready instead of figuring them out mid-conversation.

---

````markdown
I want to build a personal digital business card: a single static web page,
reachable by NFC tap or QR code, that works like a modern replacement for a
paper business card. I'm attaching a screenshot of a reference card for
LAYOUT AND STRUCTURE inspiration only — do not copy its colors, fonts, or
company branding. This is my own card, for my own identity.

Before writing any code, ask me questions about the following, one topic at
a time, and wait for my answers:

1. **Identity** — my name, job title, company (if any), location, and which
   contact channels I want as tap-to-act buttons (phone call, email, SMS,
   website, a meeting-booking link like Calendly, LinkedIn, other socials).
2. **Brand** — my color palette (or a vibe like "dark and minimal" /
   "bright and playful" if I don't have exact colors), and font preferences
   (or let you suggest a pairing — e.g. a serif display font for my name,
   a clean sans-serif for body text).
3. **Photo** — do I have a square profile photo ready? What format/size?
4. **Sections beyond the core card** — do I want any of these optional
   sections, and if so, what feeds them:
   - A "latest content" feed (blog posts, articles, a newsletter archive)
   - A "latest videos" feed from a YouTube channel
   - A QR code fallback section at the bottom of the page, for cases where
     NFC isn't available and I want someone to scan my screen directly
5. **Hosting** — confirm I'll deploy this to Azure Static Web Apps (free
   tier) via GitHub Actions, and whether I already have a custom domain or
   will use the default `<name>.azurestaticapps.net` URL for now.
6. **Privacy** — confirm this page should be blocked from search engine
   indexing (robots.txt + meta robots tags) since it's meant to be reached
   only via the site's address (an NFC tap, a QR code, or a shared link),
   not discovered and indexed via Google or AI engines.

Once I've answered, build a plain HTML/CSS/JS static site (no build step,
no framework, no backend) with this structure:

- A hero section: photo in a circular frame, name, title, company, location,
  a short one-line tagline in my own words, and a few skill/topic "pills."
- A "Save Contact" button that downloads a `.vcf` (vCard 3.0) file with my
  details, including my photo embedded as inline base64
  (`PHOTO;ENCODING=b;TYPE=JPEG:<base64>`) — NOT as a remote URL. Use the
  Blob + `<a download>` pattern for the download trigger, not
  `window.location.href = 'data:...'`.
- A grid of tap-to-act buttons for the contact channels I chose.
- Any optional sections I asked for (content feed / video feed / QR
  fallback), each fetching its data from a same-origin static JSON file
  (e.g. `articles.json`, `videos.json`) rather than fetching a third-party
  URL directly from the browser.
- A `robots.txt` that disallows all crawlers, and a
  `<meta name="robots" content="noindex, nofollow, noai, noimageai">` tag —
  the `noai`/`noimageai` directives are a newer convention some AI
  crawlers respect for opting out of training/indexing use, worth
  including alongside the standard search-engine directives.
- A `staticwebapp.config.json` for Azure Static Web Apps with:
  - A strict Content-Security-Policy (`default-src 'none'`, then allow-list
    only what's actually used: `self` + Google Fonts for style/font,
    `self` + `https:` for images, `self` for scripts and connect).
  - Security headers: `X-Content-Type-Options`, `X-Frame-Options`,
    `Referrer-Policy`, `Strict-Transport-Security`, `Permissions-Policy`.
  - `navigationFallback.exclude` covering every static asset extension I
    actually use (css, js, png, jpg, svg, vcf, json, txt, ico) — otherwise
    Azure will try to serve `index.html` for requests to those files
    instead of the real file.
- If I asked for a content feed or video feed: a GitHub Actions workflow
  (not client-side JS) that runs on a schedule (cron) plus manual
  `workflow_dispatch`, fetches the external feed **server-side** (Python or
  Node, no extra dependencies needed for RSS/Atom XML), and commits the
  result as a small JSON file back into the repo — because a direct
  browser-side fetch to most third-party feeds will be silently blocked by
  CORS (no `Access-Control-Allow-Origin` header), and this sidesteps that
  entirely with no backend server needed.
- Make the profile `<img>` visible by default (not hidden until a JS
  `onload` fires) with an `onerror` fallback to a text-initials placeholder
  — don't hide the image and reveal it via JS on load, since on a fast
  connection the load can complete before the JS handler even attaches,
  permanently stranding the page on the fallback.

Ask me clarifying questions as you go rather than assuming defaults for
anything identity- or brand-related. Where you do need to make a technical
default (e.g. exact CSS values, breakpoint sizes), pick something sensible
and tell me you did, rather than asking about everything.
````

---

Once you're through the interview and have a working local site, head to
**[the setup guide](03_SETUP-GUIDE.md)** to get it live.
