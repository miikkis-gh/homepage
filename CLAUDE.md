# miikkis Homepage

Static site (no build step, no dependencies) — plain `index.html` + `styles.css`.
Deployed via GitHub Pages to miikkis.fi (see `CNAME`).

## Design system: Etch
The look follows the owner's **Etch** design system (a claude.ai Design System artifact:
https://claude.ai/artifact/16p7erD6h3dQ3uGAmwy7GF — its `project/README.md` is the brand book).
Etch ships React components; this site does not use them. Instead `styles.css` re-implements
them in plain CSS on Etch's token values (`--etch-*` on `:root`, copied from its `tokens.json`,
Dark theme only), and `index.html` carries a vanilla-JS port of its engraving drawer (`Engrave`,
fills every `[data-motif]` box: `nest`, `lanes`, `ladder`, `bus`, `comb`).
Rules that matter when adding things:
- Layout is a floor plan: a `.die` (metal frame + rails) holding `.cell`s. `.die--thin` for the
  column panels; `.rails` is a transparent sub-grid whose gaps show the same metal.
- Monochrome. No brand colours (service icons are `currentColor`), no rounded corners, no
  shadows/glows/gradients except the metal, no entrance animations. Colour only for warnings.
- Labels are `.micro` (Geist Mono, uppercase). Links are `a.cell` with a chevron (`#chevron` symbol).
- Fonts: `fonts/Geist-Variable.woff2` and `fonts/GeistMono-Variable.woff2` (self-hosted, OFL).

**Workflow:** commit and push changes straight to `main` — no feature branches or PRs
(owner's preference). Every push to `main` deploys live, so test locally first.

## Local testing
Must use a real HTTP server, not `file://` — the Updates feed fetches a Google
Sheets CSV via `fetch()`, and Google's CORS headers omit `access-control-allow-origin`
for `Origin: null` (i.e. directly opened files), so the fetch silently fails.
Run: `python3 -m http.server 8000` and test at `http://localhost:8000`.

## Updates feed
`#tweet-feed` (in `.col-updates`) renders a live post feed, not static content; each post is a
`.cell`, headed by its date (no author line — the masthead names the page). Source is a published
Google Sheet (CSV), URL hardcoded in `index.html` as `UPDATES_CSV_URL`. Columns:
`date`, `text`, `image` (optional) — header row is optional (parser auto-detects it;
without one, columns are assumed to be in that order), dates accept either
`YYYY-MM-DD` or `D.M.YYYY`. Sorted newest-first, capped at 8 posts, fetched once per
page load (no polling). To add a post: just add a row to the sheet, no deploy needed
— but Google's own publish-to-web cache can lag a few minutes.

`text` supports two conventions: a leading `>` renders the post as a blockquote
(muted text under a "Lainaus" label), and literal newlines in the cell (Cmd/Option/Ctrl+Return
in Sheets) render as line breaks (`white-space: pre-wrap` on both `.tweet-text` and
`.tweet-quote`).

`image` must be a direct image URL (e.g. Imgur, GitHub-hosted, or a Drive link
reformatted as `https://drive.google.com/uc?export=view&id=<FILE_ID>`) — a normal
Drive "share" link won't render as an `<img>` src. Rendered below the post text,
capped at 320px height, cropped via `object-fit: cover`.

**Clicking a photo opens it large** in a lightbox (one shared native `<dialog id="lightbox">`;
click anywhere / the × / Esc closes; the photo card is a `<button>`, so Enter/Space works).
Only photos do this — YouTube video cards stay plain links to YouTube. For
`*.googleusercontent.com/pw/…` links the lightbox swaps in the same photo at `=w1600`
(`largeImageUrl()`), since the copied link often carries a small size. Observed: a bare
Google Photos link (no `=…` suffix) can return a downsized ~512px copy, not the original,
so a photo can look soft when enlarged unless the link was copied at a decent size.

The `image` cell can also hold a **YouTube link** (`youtube.com/watch?v=`, `youtu.be/`,
`/shorts/`, `/embed/`, `/live/`): it renders as a 16:9 video card — the thumbnail
(`i.ytimg.com/vi/<id>/hqdefault.jpg`, black bars cropped) with a play button, opening
the video on YouTube in a new tab. No embedded player, so nothing but the thumbnail
request goes to YouTube before a click. A post may have an `image` and no `text` (the
card then shows just the header and media). Any other `image` value that fails to load
(e.g. a normal page URL) is removed instead of leaving a broken-image box.

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
The hero die (`.die--hero`, under the "MIIKKIS" masthead) shows two live readouts beside the links:
- **Clock** (`#clock-time` + `#clock-date`): ticks every second via `Intl.DateTimeFormat` with
  `timeZone: 'Europe/Helsinki'` — always Helsinki time regardless of visitor's
  own timezone.
- **Location** (`#location`): fetched once per load from `LOCATION_CSV_URL` in
  `index.html` — a separate published CSV of a "Location" sheet tab (own
  `gid`, distinct from Updates' `gid=0`) in the same spreadsheet. Row 1 is
  `place,date` (date as `D.M.YYYY`); the page shows the place and "Saapunut D.M." and leaves
  the cell as pure engraving if the date is missing or older than `LOCATION_MAX_AGE_MS`
  (7 days).

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

## Electricity price card
Left column (`.col-price`, heading "Pörssisähkö"): a stepped line/area
chart of Finnish spot prices for today + tomorrow, in c/kWh incl. VAT. Source is
`https://api.spot-hinta.fi/TodayAndDayForward?region=FI&priceResolution=15`
(`SPOT_PRICE_URL` in `index.html`) — free, no key, sends `Access-Control-Allow-Origin: *`.
Fields used: `DateTime` (has a `+02:00`/`+03:00` offset) and `PriceWithTax` (€/kWh, ×100 for c/kWh).
Tomorrow's prices appear once published (usually early afternoon), so the chart is
48h or 24h wide depending on the time of day.
- Fetched once per load; every 30 s the page only checks whether the 15-min slot
  changed and, if so, redraws the "now" marker/headline from the cached data.
- Chart is hand-built inline SVG (no library). Hover/touch shows a crosshair +
  tooltip; the SVG is focusable and Left/Right arrows step through slots. A
  "Table view" `<details>` lists every slot, so nothing depends on hover.
- All times are formatted in `Europe/Helsinki`, like the clock.
- Other Finnish price APIs (porssisahko.net, sahkohinta-api.fi) have no CORS headers, so
  they can't be called from the browser; spot-hinta.fi has no history endpoint.
- **UI language:** all text in the card is Finnish (`lang="fi"` on `.price-section`), with
  decimal commas and Finnish weekday names via the `fi-FI` formatter.
- **Pörssi / Kokonaishinta (Spot / All-in) switch** (default Pörssi; choice + company persist in `localStorage`,
  wrapped in try/catch). All-in = spot + the chosen network company's general-transfer
  (yleissiirto) c/kWh + electricity tax, all incl. 25.5 % VAT. Data lives in the `GRID_FEES`
  constant in `index.html`: per company `rate` (c/kWh incl. VAT), `basicFee` (€/month,
  shown only as a note, never in the c/kWh chart), `fuse`, `from` (price list effective
  date) and `source` (the company's own price list). Parts are rounded to 2 decimals
  *before* summing so tooltip/table parts always add up to the total.
- **Electricity tax** (`GRID_FEES.tax`): class 1, 2.325 c/kWh excl. VAT × 1.255 = 2.917875.
  It already includes the security-of-supply (huoltovarmuus) fee — don't add that again.
  Valid from 1.4.2026; update if the state changes it.
- **Maintaining the fee list:** companies change prices once or twice a year (Helen's
  changed 1.10.2026). Re-read each `source`, update `rate`/`basicFee`/`from`, bump
  `checked`. Only figures from a company's own price list go in — comparison sites
  disagreed with each other. Only the general product is covered (no night/seasonal).
  Included: Caruna, Caruna Espoo, Elenia, Helen, Kuopio, Oulu, Tampere, Turku.
  Vantaan Energia Sähköverkot is *not* included: its price list PDF is bot-blocked
  (403) and its pages don't state the prices. Add it if you can read the list.
  The Energy Authority's all-company table is behind a Cloudflare challenge too.
  Caruna's and Turku's lists are dated 1.9.2024 on their own pages (still the
  published ones when checked 2026-10-10).
- Planned, not built: a "vs. 30-day median" comparison. It needs history, so the
  plan is a scheduled deploy-workflow step that computes the median server-side from
  sahkohinta-api.fi (hourly, ≤60 calls/h, ENTSO-E data) into a same-origin
  `baseline.json` — which would then have to be added to the deploy allowlist.

## Icons
Service logos (Spotify/Suno/Instagram links, plus the YouTube heading logo in `index.html`) are sourced from
the Simple Icons project (`cdn.jsdelivr.net/npm/simple-icons/icons/<name>.svg`) for
accurate paths — don't hand-draw new ones if adding more services. Per Etch they're drawn in
`currentColor`, not their brand colours.

## Notes link
The last link cell ("Notes") points to the Finnish-language notes site at `https://i.miikkis.fi` —
a separate repo, `miikkis-gh/notes` (MkDocs Material, deployed by its own Pages workflow).
Nothing from it lives in this repo. Its icon is the handbook's own logo
(MkDocs Material's default `material/library`), so the two match.

## YouTube channels
Third column, below Playlists, headed "Katson" ("Watching"): a hardcoded list of link cells
(`.channel-links`). To add one, copy an `<a class="cell">` in `index.html`
and change the `@handle` URL and name, under the right `.channel-group` category
label (News & politics / Investigations / Science & education). The YouTube logo
is from Simple Icons.

## Deploy allowlist
`.github/workflows/deploy.yml` copies an explicit file list into `_site/` and
deploys only that — README, CLAUDE.md, `Derbyshire Bold.otf`, etc. are not published
(the `fonts/` folder is). **Adding
a new public file (image, font, page) means adding it to the `cp` line there**,
or it'll 404 on the live site while working fine locally.

## Deploy caching
`styles.css`/`index.html` on the live site have a 10-min `Cache-Control` via GitHub
Pages' CDN. After pushing, changes can take a few minutes to appear — hard-refresh
(⌘-Option-R in Safari) before assuming a change didn't work.

## Security posture (verified 2026-08-30)
- HTTPS enforced, custom domain verified with GitHub (prevents dangling-CNAME takeover
  if the repo were ever deleted), DNS uses correct apex A-records to GitHub's Pages IPs.
- `index.html` has a CSP meta tag (`default-src 'self'`, `font-src 'self'` since fonts are
  self-hosted, with explicit allowances for the Sheets CSV fetches, `api.spot-hinta.fi` (price card), the Spotify embed, and https/data images).
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
