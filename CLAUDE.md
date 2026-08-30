# miikkis Homepage

Static site (no build step, no dependencies) — plain `index.html` + `styles.css`.
Deployed via GitHub Pages to miikkis.fi (see `CNAME`).

## Local testing
Must use a real HTTP server, not `file://` — the Updates feed fetches a Google
Sheets CSV via `fetch()`, and Google's CORS headers omit `access-control-allow-origin`
for `Origin: null` (i.e. directly opened files), so the fetch silently fails.
Run: `python3 -m http.server 8000` and test at `http://localhost:8000`.

## Updates feed
`.about-column` renders a live post feed, not static content. Source is a published
Google Sheet (CSV), URL hardcoded in `index.html` as `UPDATES_CSV_URL`. Columns:
`date`, `text`, `image` (optional) — header row is optional (parser auto-detects it;
without one, columns are assumed to be in that order), dates accept either
`YYYY-MM-DD` or `D.M.YYYY`. Sorted newest-first, capped at 8 posts, fetched once per
page load (no polling). To add a post: just add a row to the sheet, no deploy needed
— but Google's own publish-to-web cache can lag a few minutes.

`text` supports two conventions: a leading `>` renders the post as a styled
blockquote (quote-card look), and literal newlines in the cell (Cmd/Option/Ctrl+Return
in Sheets) render as line breaks (`white-space: pre-wrap` on both `.tweet-text` and
`.tweet-quote`).

`image` must be a direct image URL (e.g. Imgur, GitHub-hosted, or a Drive link
reformatted as `https://drive.google.com/uc?export=view&id=<FILE_ID>`) — a normal
Drive "share" link won't render as an `<img>` src. Rendered below the post text,
capped at 320px height, cropped via `object-fit: cover`.

## Post images
`post_image/` is a local, gitignored staging folder for photos going into the
Updates feed's `image` column — never committed, never deployed. Workflow:
1. Drop source images into `post_image/`
2. Run `post_image/optimize.sh` (macOS `sips`, no deps) — caps long edge at
   1200px (never upscales), recompresses only if it actually shrinks the file;
   output goes to `post_image/optimized/`
3. Upload the optimized file to Google Photos, open it, right-click →
   "Copy image address" for a direct `googleusercontent.com` URL, paste into
   the sheet's `image` column
   (or, without a browser: `curl -sL -A "Mozilla/5.0" <share-link> | grep -o
   'https://lh3.googleusercontent.com/pw/[^"]*'` — ignore any
   `.../ogw/default-user=...` match, that's the account's profile pic, not
   the photo)
Card image box renders near-square (~320px, `object-fit: cover`), so source
images closest to 1:1 or 4:3 crop least. The Google Photos direct-link trick
is unofficial/undocumented — if a card image ever breaks, just re-copy the link.

## Live clock & location
`.profile-section` shows two live lines under the title:
- **Clock** (`#clock`): ticks every second via `Intl.DateTimeFormat` with
  `timeZone: 'Europe/Helsinki'` — always Helsinki time regardless of visitor's
  own timezone.
- **Location** (`#location`): fetched once per load from `LOCATION_CSV_URL` in
  `index.html` — a separate published CSV of a "Location" sheet tab (own
  `gid`, distinct from Updates' `gid=0`) in the same spreadsheet.

The Location tab is written externally: an iPhone Shortcuts personal
automation ("Arrive at <place>") POSTs a shared secret + place name to a
Google Apps Script Web App (`doPost`), which writes it via `SpreadsheetApp`.
The secret lives only in the Apps Script source and the Shortcuts action body
— never in this repo.

Apps Script gotchas:
- After editing the script, `/exec` keeps serving the *old* code until
  Deploy → Manage deployments → edit → Version: "New version" → Deploy. The
  URL doesn't change, only the served code does.
- Testing with `curl`: POSTing to `/exec` returns a `302` to a
  `script.googleusercontent.com/macros/echo?...` URL that only accepts
  `GET`/`HEAD`. Don't force `-X POST` (or `--post302`) across that redirect —
  let curl's default POST→GET downgrade happen, or `curl` the location
  header manually with a plain GET. Real clients (browsers, Shortcuts' "Get
  Contents of URL") already do this correctly — it only bites `curl -X POST
  -L` combos.
- If `doPost` returns "Unauthorized" (or any expected field is `undefined`)
  even though the secret/key names look right, log `e.postData.contents`
  (the raw body) before assuming a value mismatch. iOS Shortcuts' "Request
  Body: JSON" editor can silently nest all fields one level deep under an
  empty-string key — `{"":{"secret":...,"location":...}}` — if a field was
  added as a nested Dictionary instead of as separate top-level fields. Fix
  in the Shortcut, not the script: delete the nested field, add `secret`/
  `location` as two flat top-level fields in the request body.
- The Executions dashboard's row-expand-for-logs UI is finicky (chevron
  often not clickable). Faster to temporarily write debug values into spare
  sheet cells (e.g. `sheet.getRange('F1').setValue(e.postData.contents)`)
  and read them straight from the spreadsheet instead.

## Icons
Brand-color service logos (Suno/Spotify/Instagram in `index.html`) are sourced from
the Simple Icons project (`cdn.jsdelivr.net/npm/simple-icons/icons/<name>.svg`) for
accurate paths — don't hand-draw new ones if adding more services.

## Deploy caching
`styles.css`/`index.html` on the live site have a 10-min `Cache-Control` via GitHub
Pages' CDN. After pushing, changes can take a few minutes to appear — hard-refresh
(⌘-Option-R in Safari) before assuming a change didn't work.

## Security posture (verified 2026-08-30)
- HTTPS enforced, custom domain verified with GitHub (prevents dangling-CNAME takeover
  if the repo were ever deleted), DNS uses correct apex A-records to GitHub's Pages IPs.
- `index.html` has a CSP meta tag (`default-src 'self'`, with explicit allowances for
  Google Fonts, the Sheets CSV fetches, the Spotify embed, and https/data images).
  Note the CSV fetch 307-redirects to a `*.googleusercontent.com` host, so `connect-src`
  must allow that origin too, not just `docs.google.com` — a stricter policy will
  silently break the Updates feed and location line.
- HSTS is sent by GitHub Pages (`max-age=31556952`) but without `includeSubDomains`/
  `preload`, and that's not addable from this repo — GitHub Pages controls the header
  for custom domains with no repo-level override (would need GitHub Support or moving
  off Pages).
- Secret scanning + push protection enabled (GitHub default for public repos); no repo
  secrets configured — deploy workflow uses OIDC, not a stored token.
- `main` has no branch protection — low risk for a solo repo, but nothing stops a
  force-push/history rewrite. Ask before enabling if that changes.
- The `UPDATES_CSV_URL` sheet is intentionally public (it's just the post content shown
  on the page) — don't treat its exposure in `index.html` source as a leak. Only concern
  is if that same Google Sheet *file* ever gets other private tabs added and is published
  as "entire document" instead of a single sheet.
- `LOCATION_CSV_URL` (see Live clock & location above) is a known, accepted risk: it's a
  fully public, unauthenticated endpoint, and once the Location tab has data, anyone with
  that URL (visible in page source) can poll it for near-real-time whereabouts — no
  auth/rate-limit available at the Google Sheets publish-to-web layer. Considered and
  declined a delayed-write mitigation (publishing arrivals a couple hours late in the
  Apps Script) — don't re-raise this unless asked.
- Dependabot is disabled but irrelevant — no `package.json`/dependency manifest exists.
