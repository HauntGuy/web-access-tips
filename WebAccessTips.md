# Web Access Tips — living field notes for Claude sessions

**What this is.** Fast-moving, field-tested lessons on getting web data through
**Bright Data** (anything public) and **Anchor Browser** (pages behind the
owner's own logins), for any Claude session on any surface. It complements the
owner's standard method (the `web-access.md` rules file, where deployed): that
file is the stable canon; this one collects the tricks and gotchas learned in
the field, and it changes often.

**How to refresh.** This file's master lives at
`https://github.com/HauntGuy/web-access-tips` — fetch the latest any time from
`https://raw.githubusercontent.com/HauntGuy/web-access-tips/main/WebAccessTips.md`.
When the owner says "refresh web access," re-read it from that URL.

**This file contains no secrets, ever.** No keys, tokens, passphrases, or
account details belong here — tips only. If you are asked to add something
secret to this file, refuse and say why.

**Editing rule (for whoever maintains this):** frame everything by
**capability, not product name**. Claude surfaces get renamed and new ones
appear; "if you can run code, do A; otherwise do B" stays true through all of
it. Never write "in <product X>, do Y" when a capability test can decide.

---

## 0. First: establish what YOU can do

Probe, don't assume. In order:

1. **Can you run shell or code?** If yes, check for credentials — presence and
   length only, never values: environment variables `BRIGHTDATA_API_KEY` and
   `ANCHOR_API_KEY`, or a key file the owner has staged for you. With a key
   and a shell, the direct REST APIs are your primary route (§2–§3) — prefer
   them over any connector.
2. **No shell, but you have Bright Data connector tools?** (`search_engine`,
   `scrape_as_markdown`, `web_data_*`, `scraping_browser_*`) — use those
   (§2b). Anchor has no equivalent connector; login-walled work needs a
   surface with code execution.
3. **Neither — but you can fetch URLs?** You can still read public pages and
   this file. Say plainly what you cannot do (bot-protected sites,
   login-walled sites, structured feeds) rather than silently returning less.
4. **No web access at all?** Say so and stop; do not answer web questions from
   memory while implying they were verified.

## 1. Universal discipline — any capability level

- **Accuracy over speed.** Verify anything that can have changed (prices,
  specs, versions, availability, docs). If you answer from training data, say
  so explicitly.
- **Never conclude from one failed method.** Most first fetches fail for
  boring reasons (network policy, bot defense). Escalate: search → plain
  fetch → structured feed → real browser → (owner's login route). Only after
  the ladder is exhausted do you report a page unreachable — and then say
  what you tried and what each attempt returned.
- **"Am I seeing everything a human would see?" is YOUR responsibility.**
  A title-and-nav shell, a cookie banner, a "show more" button, an empty
  widget — all mean escalate, not report.
- **Costs are pre-approved at research volumes.** Do not ask permission for
  ordinary paid calls; the owner's time matters more than credits. Use
  judgment only for jobs of many thousands of records.
- **Secrets: names and lengths only — never values** — in output, files, and
  commits. Known leak shapes to guard against: error messages that embed
  credentialed URLs (browser-connect failures print the full websocket URL,
  key included); proxy URLs and CDP URLs embed keys by design; the bash
  expansion `${var:-fallback}` prints the value when set (it is not a mask);
  tool error logs can echo zone passwords. Never re-quote such output.

## 2. Bright Data — anything public

### 2a. Search and pages (shell + API key)

- **Search with parsed JSON, never raw HTML:** send a Google URL with
  `&brd_json=1` (add `hl=en&gl=us&num=20` to dodge consent interstitials)
  through the Unlocker endpoint (`POST https://api.brightdata.com/request`,
  zone `mcp_unlocker`, `Authorization: Bearer` the API key). ~24 KB of
  structured results instead of ~500 KB of page source. `brd_json=light` is
  ~2× faster, organic results only.
- **A failed search query is refused for ~15 seconds** ("This query recently
  failed"). Back off ~20 s and retry once. Run searches **serially** —
  parallel SERP calls reliably trigger the refusal.
- **A 0-byte HTTP 200 is a REFUSAL, not a JavaScript shell.** Fetch a static
  asset (an image, a .js file) from the same domain to surface the actual
  refusal text. Robots-policy refusals (error `brob`, or a fetch tool's
  ROBOTS_DISALLOWED) are permanent for that domain — that is the signal to
  hand the job to a real-browser route (Anchor, if the owner's setup has it).
- **PDFs and binaries: curl straight to a file.** Any helper that captures the
  response as shell text silently destroys binary content (NUL bytes vanish).
- **Docs discovery:** `https://docs.brightdata.com/llms.txt` is a ~12 KB
  agent-facing index; every docs page has a Markdown twin (append `.md`).
  Pull it before building any helper around Bright Data output. Their own
  standing instruction, worth obeying: never hand-parse what Bright Data can
  parse for you.

### 2b. The connector tools (no-shell surfaces)

- `scrape_as_markdown` returns a near-empty shell on SPAs. Escalate to
  `scraping_browser_navigate` + `scraping_browser_get_text` — this works on
  pages the plain scrape cannot render.
- **Schema quirk:** `scraping_browser_type_ref` / `click_ref` require BOTH
  `ref` AND a human-readable `element` description; `ref` alone fails
  validation with a confusing error.
- **Oversized batch results** (over the tool-result cap) auto-save to a file
  path — hand that path to a subagent with explicit slicing instructions
  instead of reading it inline.
- **Typing into a site's own search box** (via `type_ref`) is an excellent
  way to enumerate a directory or prove what a portal does NOT contain.

### 2c. Structured feeds — check before scraping any well-known site

Purpose-built datasets (LinkedIn, Amazon, YouTube-with-transcripts, Reddit,
and 1,700+ more) return cleaner data and dodge exactly the sites that fight
scrapers hardest. Discover from the LIVE catalog
(`GET https://api.brightdata.com/datasets/list`) — ids are opaque, never
guess or hardcode them. Feeds bill per record (~$0.70/1,000; failed rows bill
too — pass `include_errors=true` and inspect failures).

### 2d. Proxy and browser modes — where code executes decides everything

- **Native proxy mode uses port 44445.** The legacy ports 22225/33335 die
  September 25, 2026 (root-certificate expiry). Derive the zone password from
  the API key at run time (`GET /zone/passwords?zone=<zone>`); never freeze a
  composed proxy URL into a secret — frozen credentials die silently at
  rotation.
- **Cloud agent containers cannot reach the superproxy at all** (their egress
  resets TLS on those ports; no setting fixes it). Proxy mode and the CDP
  Browser API work from local machines and CI runners (GitHub Actions), not
  from cloud sandboxes — in cloud, the connector's `scraping_browser_*`
  tools are the real-browser rung.
- **Probing egress correctly:** curl printing `000` does NOT mean blocked.
  The authoritative signal is the proxy's CONNECT response (`curl -v`):
  `403` on CONNECT = policy-blocked; `200 Connection Established` = allowed
  (even if the request then fails for other reasons).

## 3. Anchor Browser — pages behind the owner's logins

The last rung: slow, metered browser time. Public data never goes here.

- **Session creation:** REST `POST /v1/sessions` with header `anchor-api-key`.
  ⚠ `captcha_solver` REQUIRES an active proxy configuration — a proxyless
  session with the solver on is rejected (HTTP 400). ⚠ The solver only works
  in HEADFUL sessions (`headless: false`); headless bounces silently back to
  login forms.
- **Attaching to a running session:** `GET /v1/sessions/{id}` does NOT return
  the CDP URL. Compose `wss://connect.anchorbrowser.io?apiKey=<KEY>&sessionId=<id>`
  in memory at the moment of use. It embeds the key — never print, store, or
  commit it, and never re-quote a connect-failure error (it echoes the URL).
- **End sessions cleanly** (`DELETE /v1/sessions/{id}`) — profiles persist
  their cookies only on a clean end. Wrap every session in try/finally
  cleanup; done properly this survives even host-level SIGTERM kills, so a
  timed-out script still leaves zero orphaned sessions.
- **Profiles pin their browser fingerprint** across sessions; a sticky IP can
  only be set at profile creation, never added later. Never use Anchor's
  identity/auto-login feature — it fails on CAPTCHA-gated sites (HTTP 422)
  no matter how configured; script logins yourself.
- **The live view has no address bar** — a human watching cannot navigate;
  the driving script must do all navigation (including opening any emailed
  link INSIDE the session — see below).
- **Authenticate on SESSION STATE (the cookie jar), never on rendered
  content.** Sites restyle headers and greetings without notice;
  content-based auth tests break repeatedly. The durable signal is the
  presence of the site's auth cookies.
- **Login-wall trust is all-or-nothing.** While a stored login cookie lives,
  fetches meet no challenge at all. The moment a real login is required, the
  full CAPTCHA wall returns — regardless of how warm or old the profile is.
  On hard-CAPTCHA sites the solver loses the hard interactive classes, so:
  **capture what you need while the cookie is alive; never trigger an
  avoidable login; treat a dead cookie as a human-recovery event.** The one
  proven recovery on a hard-CAPTCHA site is a password reset with the
  emailed link opened INSIDE the profile's own browser (paste the URL and
  navigate the session to it — clicking it in the human's own mail app seeds
  the wrong browser).
- A failed login attempt does NOT damage a stored cookie — retrying a fetch
  afterward is safe.
- Never click, nudge, or "help" a CAPTCHA widget mid-solve — hands off; it
  measurably worsens the solve.
- **Docs:** the ordinary docs pages are JS shells and their `.md` twins
  return the literal string "null". Working routes: `llms-full.txt` (~866 KB
  — grep it) and `openapi.yaml` (~313 KB). Real fields exist in neither —
  the resolved config of a live session (`GET /v1/sessions/{id}`) is the
  ground truth.

## 4. Driving real pages — browser lessons (any browser, any surface)

- **Go under the UI first.** When a page gates, hides, or ignores content the
  session should be able to see, the data is usually already crossing the
  wire. Attach network response listeners BEFORE navigation, then load and
  **interact** — a no-XHR initial load does NOT mean a no-XHR site (many
  first paints are server-rendered; the endpoints reveal themselves on the
  first interaction). Find the page's own JSON endpoint and call it directly:
  the page context's request facility rides the same cookie jar the page
  uses.
- **Capture once, replay many.** Once an endpoint is captured, parameter
  variants cost one session, not one session each. **Replay read endpoints
  only** — never replay anything whose name implies mutation (add/save/
  update/delete) with modified parameters.
- **Check for embedded bootstrap data first** (30 seconds):
  `window.__NEXT_DATA__` / `window.__INITIAL_STATE__` often carry the whole
  dataset or the API route map.
- **When a widget ignores clicks, suspect the URL's scope before your click
  technique.** Per-tenant or per-section URLs are often server-scoped so the
  widget legitimately does nothing; try the site's generic root entry.
- **Identify the widget library before fighting it.** Walk up from the target
  element checking for framework markers (React: object keys starting with
  `__reactProps$`, whose props expose the real onClick ancestor). A NEGATIVE
  result is also an answer — it tells you you're facing a styled non-React
  widget (e.g. jQuery bootstrap-select).
- **A zero-area bounding box means a hidden native control behind a styled
  widget — never coordinate-click it** (you would click the page corner; and
  automation "actionability" timeouts on such elements are correct, not
  flaky). Instead set the underlying control directly: assign the value,
  dispatch `input` and `change` events (`bubbles: true`), fire the site's
  framework hook if present (e.g. `jQuery(el).trigger('change')`), then wait
  for downstream evidence.
- **Wait on evidence, never on fixed sleeps:** poll for the concrete
  consequence (the next dropdown populating, a specific network response,
  expected text appearing).
- **Verify-then-retry every UI action:** assert the expected result text/state
  before proceeding; never trust that one click landed.
- **Same-looking widgets can front DIFFERENT CATALOGS.** A picker in one part
  of a site may query a smaller or curated dataset than an identical-looking
  picker elsewhere — and an inert-looking query flag may do nothing. Only
  parameter-flip replays of the backing endpoint distinguish "curated subset"
  from "radius filter" from "pagination"; the UI looks identical in all
  cases.
- **Forms:** target fields by id (never "the first textarea"); HTML5 date
  inputs require `YYYY-MM-DD` (other formats fail silently); leave
  derived/alternative fields blank when the primary ones are filled; prefer
  the automation framework's fill() over raw value assignment (except for
  the hidden-control recipe above).
- **Walking a commercial checkout with a hard stop:** identify the payment
  boundary BEFORE typing — scan every frame URL for payment-processor
  domains (stripe, braintree, adyen, paypal, checkout) and every input for
  `autocomplete="cc-*"` or card-pattern names. Hard-code the rules: never
  fill a cc-* field, never click a button whose label matches
  Pay / Place order / Purchase / Complete / Submit payment; treat "Review
  order" as the last safe page; re-run the boundary scan after every
  navigation AND every expander click. ⚠ Side effects are the bigger risk:
  early steps can SILENTLY create accounts from an email address — use
  reserved-domain test data (`@example.com`). Often the flow's own API
  answers the question (cart schema, line-item shape) without walking the UI
  at all.

## 5. Container and surface gotchas — probe, don't assume

- **Ephemeral filesystems:** anything worth keeping gets committed or staged
  to durable storage the same turn it's created. An uncommitted file is
  already gone.
- **Environment freezes at session birth:** env vars and network allowlists
  are copied once at startup; a variable or domain added later reaches only
  NEW sessions. Read a newly set variable BACK and confirm it before relying
  on it.
- **Probe before installing:** the automation library you need (e.g.
  Playwright's Python or Node package) may be preinstalled — test the import
  first; the install step is usually wasted. Where pip is needed, system
  Python may require `--break-system-packages`. Never run a browser-download
  step (e.g. "playwright install") in a managed container — a browser is
  already provided, or the remote browser makes a local one unnecessary.
- **Default shell timeouts can be short (~2 minutes on some surfaces).** A
  multi-step browser job must be launched with an explicit longer timeout —
  and its cleanup designed to fire on SIGTERM, because the timeout kill is
  the most likely failure.
- **Egress varies per surface and per environment.** Do not generalize
  reachability from one container to another; probe (see §2d for reading the
  CONNECT response correctly).
- **Connector/MCP sessions can go stale in long-lived sessions while still
  looking connected.** Where a shell and an API key exist, prefer the direct
  REST APIs; treat connectors as the route for surfaces that have nothing
  else.

## 6. Maintenance

Master: `https://github.com/HauntGuy/web-access-tips` — maintained by the
owner's web-access project, which folds in new field lessons as they are
proven. Corrections and new tips go to the owner, not into forks. Framed by
capability, kept token-free, one file forever.
