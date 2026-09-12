# Prepare your assets

Gather these before you start the prompt in [PROMPT.md](02_PROMPT.md) — the AI
assistant will ask you about most of this, and you'll move faster with
answers ready rather than figuring them out mid-conversation.

## 1. Profile photo

- **Shape:** square (1:1). It gets cropped into a circle on the page, so
  frame yourself centered with some breathing room — a tight crop that
  works as a rectangle can end up clipping your head or shoulders once
  it's forced into a circle.
- **Resolution:** at least **512×512px**, ideally **1024×1024px**. The
  photo gets displayed small on the page (roughly 120–150px), but it's also
  embedded directly inside the downloadable contact card, where it may be
  viewed larger in someone's Contacts app — starting too small (e.g. the
  300×300px used in the reference build) leaves no headroom if you ever
  want to reuse the photo somewhere bigger.
- **Format:** JPEG. It gets base64-encoded and embedded directly into the
  vCard file, so file size matters — a JPEG at reasonable quality settings
  (not maximum) keeps the final page size sane. Aim for **under ~150–250KB**
  after export; a lossless PNG at the same resolution can be several times
  larger for a photograph with no real benefit.
- **Color profile:** standard sRGB. Avoid embedding an unusual color
  profile if your editing tool offers a choice.

## 2. Logo (optional)

If you want a small logo centered inside your QR code (like the lighthouse
icon in this repo's reference image):

- **Shape:** square, with a **transparent background** (PNG).
- **Size:** simple enough to stay legible small — avoid a logo with fine
  detail or thin lines. Most QR generators that support a center logo
  handle sizing and error-correction automatically, but a cleaner, simpler
  mark will always scan more reliably than a busy one.

## 3. Brand colors

Have 2–4 hex codes ready:

- A **background** color (e.g. a dark navy or off-white).
- A **primary/accent** color for headings and highlights (e.g. a gold, a
  brand blue).
- A **text** color or two (primary text, muted/secondary text).

If you don't have exact hex codes, a rough description works too ("dark and
minimal," "warm and editorial," "bright and playful") — your AI assistant
can propose a palette for you to react to instead.

## 4. Fonts

Optional, but nice to have an opinion on:

- A **display font** for your name/headings (often a serif, for a more
  editorial feel — e.g. Playfair Display).
- A **UI font** for body text and buttons (a clean sans-serif — e.g. Syne,
  Inter, or similar).

If you don't have a preference, ask your AI assistant to suggest a pairing.

## 5. Content sources (only if you want these sections)

- **Blog/articles feed:** the RSS or Atom feed URL for your blog (often
  `yoursite.com/feed`, `yoursite.com/rss.xml`, or similar — check your
  blogging platform's documentation).
- **YouTube feed:** your channel's ID (starts with `UC...`). Get this
  directly from **YouTube Studio → Settings → Channel → Advanced
  settings** — don't try to scrape it from your channel page (see
  [Lessons learned](05_LESSONS-LEARNED.md) for why that's unreliable).

## 6. A reference screenshot

A screenshot of a card whose *structure* you like — this repo's own
[README](../README.md) images, a frame from the video, or anything else —
to attach alongside the prompt. This is for layout inspiration only; the
prompt explicitly tells your assistant not to copy the colors or branding
from it.

## 7. Domain (optional)

If you own a domain and want a custom subdomain (e.g. `card.yourname.com`)
rather than the default `<name>.azurestaticapps.net` URL, make sure you
have access to add a DNS record for it. This isn't required to get started
— you can add it later.

---

Ready? Head to **[the prompt](02_PROMPT.md)**.
