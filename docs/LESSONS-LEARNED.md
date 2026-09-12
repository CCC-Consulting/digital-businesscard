# Lessons learned

Real bugs and gotchas hit building the reference card. The prompt in
[PROMPT.md](PROMPT.md) already bakes in the fixes — this explains *why*,
which helps if you customize further or hit something similar.

## "Save Contact" silently did nothing on iPhone

The first version used `window.location.href = 'data:text/vcard...'` to
trigger the download on iOS specifically. Safari has deliberately blocked
script-triggered top-level navigation to `data:` URIs since around
2019/2020 as an anti-phishing measure — and it fails *silently*, no console
error, nothing visibly wrong to a user. The fix: use a `Blob` +
`<a download>` link instead, which works on Safari 13+ (i.e. all real-world
iOS devices) and every other browser too.

## The saved contact had no photo

A vCard `PHOTO;VALUE=URI:https://.../photo.jpg` pointing at a remote image
looks correct, but iOS Contacts does not fetch remote URLs referenced
inside an imported `.vcf` file. The fix: embed the photo directly as
base64 inside the vCard (`PHOTO;ENCODING=b;TYPE=JPEG:<base64 data>`), which
every Contacts app can read with no network request at all.

## A "fetch on page load" image swap intermittently failed

A pattern like "hide the photo by default, reveal it once a JS `onload`
handler fires" can lose a race: if the image finishes loading (and fires
its `onload` event) *before* the browser's parser even reaches the
`<script>` tag that's listening for it — which happens more often than
you'd think on a fast connection or warm CDN cache — the handler never
runs, and the photo stays hidden forever. The fix: show the image by
default via CSS, and only use `onerror` as the fallback path (which fires
reliably regardless of timing) if the image genuinely fails to load.

## The RSS feed always showed stale, hardcoded fallback content

A direct client-side `fetch()` to an external blog's RSS feed was silently
blocked by CORS (see the setup guide's [content-feed
section](SETUP-GUIDE.md#6-how-the-content-feed-automation-is-set-up)) — the
page always fell back to its hardcoded placeholder list, with no error
visible anywhere. Moving the fetch server-side into a scheduled GitHub
Action (committing a same-origin JSON file) fixed it permanently.

## A YouTube channel ID lookup by scraping the channel page failed

Automated tools fetching `youtube.com/@handle` directly can hit an EU/EEA
cookie-consent redirect (`consent.youtube.com`) depending on the
requester's apparent region, and third-party ID-lookup sites can block
automated requests outright. The reliable source is your own YouTube
Studio settings, not scraping.

## Infrastructure config and the live DNS record drifted apart

A custom domain got renamed at one point, and the HTML was updated to
match — but a separate config file (infrastructure-as-code parameters)
referencing the same domain was missed, and kept a stale, misspelled value
with no matching DNS record at all. If the infra pipeline had ever been
re-run from that stale config, it would have failed trying to validate a
domain that doesn't exist. Lesson: when a domain or any config value
changes, grep the whole repo for the old value — don't assume you found
every reference by memory.

## Adding a live embedded video player would have meant loosening the security policy

A strict Content-Security-Policy (`default-src 'none'`, then an explicit
allow-list) is good practice for a page with no backend and no reason to
load arbitrary third-party content. An embedded YouTube `<iframe>` player
would need a `frame-src` exception added to that policy, plus it pulls in
YouTube's own JavaScript and cookies on every page load. Linking out to
thumbnail cards instead (rather than embedding a live player) keeps the
strict policy intact and avoids loading any third-party tracking on page
load at all.

## A paid developer account is a real gate for Apple Wallet, not just Google Wallet

If you want an "Add to Wallet" button: Google Wallet is free (a Google
Cloud service account + a one-time Wallet issuer application), but Apple
Wallet requires signing a `.pkpass` file with a certificate that only comes
from an **Apple Developer Program membership — a real $99/year cost**, not
a one-time setup fee. Worth knowing before you commit to the feature, not
after.

## GitHub's own Actions runners deprecate Node.js versions over time

Pinned action versions (e.g. `actions/checkout@v4`) can silently start
running on a newer forced runtime with just a warning, until the old
runtime is removed entirely on a fixed date. Keep an eye on your workflow
run logs for deprecation notices and bump pinned action versions (`@v4` →
the current major) when you see one, rather than waiting for a hard cutoff
to break your pipeline.
