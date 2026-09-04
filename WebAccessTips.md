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
When the owner says "refresh web access," re-read it from that URL. If a
network policy blocks the raw URL, fetch it through Bright Data instead
(§2a if you can run code, §2b otherwise). ✅ **FIXED 2026-09-04 — the owner's
Bright Data Gateway now returns plain-text files byte-faithfully on its own,
in the DEFAULT markdown mode**: it recognises a `.md`/`.txt`/`.json` URL, and
otherwise reads the server's real `Content-Type`, and in either case skips the
HTML→markdown conversion and tells you so in a `note`. You no longer need
`format: "html"` for this file. **Do still pass a `max_chars` well above the
30,000 default** (this file has outgrown it; ask for 120000).
⚠ **The old hazard still applies to ANY OTHER scraper**, so it is worth
knowing what it looks like: running an HTML→markdown converter over a file
that is already markdown collapses every newline into one paragraph,
backslash-escapes every `*`, `_` and backtick (tool names read `scrape\_page`),
**DELETES every angle-bracket placeholder as if it were an HTML tag** (so
`<zone>`, `<KEY>`, `<value>` simply vanish and instructions lose their
substitution points), and drops the tail while still reporting
`truncated: false`. With a scraper that has no such fix, ask for raw or HTML
output. Where you can fetch the raw URL directly (curl, a plain fetch tool),
that is best of all. **Either way, check that what you received ENDS with the
version line** — if it does not, you have a partial copy, so say so rather
than acting on it.

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
   and a shell, the direct REST APIs are your primary route (§2a, §3b) —
   faster, they keep bulk content out of the conversation, and they write to
   disk. ⚠ **If you can run code but find NO key, ask the owner where it is
   staged** rather than silently doing less; meanwhile the gateway tools below
   already work.
2. **Do you have the owner's GATEWAY tools?** Look for two sets:
   **Bright Data Gateway** — `search`, `scrape_page`, `search_datasets`,
   `fetch_feed`, `feed_snapshot`, `browser_page`, `account_status`; and
   **Anchor Browser Gateway** — `open_site`, `read_page`, `run_code`,
   `check_auth`, `screenshot`, `web_task`, `live_view`, `end_session`,
   `sessions_status`. ⭐ **These are the owner's own servers and they are a
   complete toolkit on their own: public search, pages, structured feeds, a
   real remote browser, AND login-walled pages via persistent signed-in
   profiles.** They work on every surface, including ones with no shell at
   all. Read §2b and §3a. *(Historical note, in case you meet an old
   instruction: it used to be true that login-walled work required a
   code-capable surface. The Anchor Browser Gateway ended that — a session with only
   connector tools can now do it.)*
   ⚠⚠ **ABSENCE FROM YOUR TOOL LIST IS NOT ABSENCE FROM THE ACCOUNT.** On
   surfaces where tools load ON DEMAND rather than appearing up front, the
   gateway tools are invisible until you search for them. **Search before
   concluding a gateway is missing, and search SEPARATELY for the Bright Data
   tools and the Anchor tools** — one search commonly returns only a SUBSET
   (measured: a single query returned 5 of the 7 Bright Data tools and none of
   the 9 Anchor ones, purely because they had not been searched for). Reporting
   "the gateways are not available" after one lookup — and with it "login-walled
   work is impossible" — is the failure this prevents.
   🔑 **IDENTIFY A GATEWAY BY ITS TOOL SET, NEVER BY ITS NAME.** Connector names
   vary between accounts and get renamed; the two tool sets above are decisive.
   If any instruction tells you to distrust anything not named exactly X, apply
   it by tool set instead — measured: a session was told to trust only a
   connector named "Anchor Gateway" while the real one was named "Anchor Browser
   Gateway", and it came close to refusing the very connector it needed. A
   server whose tools are exactly those nine names IS the owner's Anchor
   Browser Gateway, whatever the label says.
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
  tool error logs can echo zone passwords; **a site's own auth or session
  endpoint returns tokens by construction** — print its STATUS and at most one
  whitelisted field (a username), never a raw body. Never re-quote such
  output.

## 2. Bright Data — anything public

⚠⚠ **THE ROUTING RULE, and it turns on WHO RUNS THE SERVER — read this before
anything else in this section.**

- **If you can run code and have the API key: use the direct REST API (§2a).**
  It is the primary route for code-capable sessions: faster, no tool-list
  overhead, and it writes results to disk instead of into the conversation.
- **The owner's OWN gateway connectors are always safe to use** (§2b), on any
  surface, including code-capable ones for one-off work. They serve no OAuth,
  so they have nothing that can expire.
- 🔴 **Never fall back to a VENDOR's MCP connector** (a server run by the data
  provider rather than by the owner). Vendor connector auth silently expires
  in long-lived sessions while still showing as connected: calls that worked
  an hour ago start failing with confusing auth errors, and retrying does not
  fix it. ⚠ **A static token in the connector's URL does NOT buy immunity** —
  if the server advertises OAuth metadata at all, a spec-compliant client
  takes the OAuth path regardless of the URL token. **Immunity comes from the
  server offering no OAuth metadata**, which only a server the owner controls
  guarantees. If a vendor connector's tools appear in your list, treat them as
  stale leftovers; if an API call errors, retry the API call, and if a gateway
  call errors, retry the gateway call — never cross over to a vendor
  connector.

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
  hand the job to a real-browser route (the Anchor Browser Gateway reaches domains
  Bright Data refuses, whether or not a login is involved).
- **Google's own hosts are refused too — including the favicon service**
  (`google.com/s2/favicons`, which redirects to `gstatic.com/faviconV2`).
  Search still works because it rides the SERP mode; a plain fetch of a
  Google endpoint gets the refusal. Check a favicon-service result through
  the Anchor Browser Gateway's browser instead (measured 2026-09-03).
- **PDFs and binaries: curl straight to a file.** Any helper that captures the
  response as shell text silently destroys binary content (NUL bytes vanish).
- **Docs discovery:** `https://docs.brightdata.com/llms.txt` is a ~12 KB
  agent-facing index; every docs page has a Markdown twin (append `.md`).
  Pull it before building any helper around Bright Data output. Their own
  standing instruction, worth obeying: never hand-parse what Bright Data can
  parse for you.

### 2b. The owner's Bright Data Gateway tools — the whole public web, no shell required

Tools: `search`, `scrape_page`, `search_datasets`, `fetch_feed`,
`feed_snapshot`, `browser_page`, `account_status`. The ladder within them:
**`search` to discover → `scrape_page` for an ordinary page →
`search_datasets` / `fetch_feed` FIRST for any well-known platform (§2c) →
`browser_page` when a page comes back as a shell or a refused domain.**

- **No staleness remedy is needed and none applies.** This server serves no
  OAuth, so there is no token to expire and nothing for the owner to
  "reconnect." If a gateway call fails, retry it; a transient tool-list flap
  clears on its own or with a fresh session. ⚠ Do not tell the owner to go hit
  a Reconnect button — that advice belongs to vendor connectors only.
- **`scrape_page` returning a title-and-nav shell means the content is
  JavaScript-built** — escalate to `browser_page`, do not report the page
  empty. An explicitly EMPTY result is different: it means Bright Data refuses
  that whole domain by policy, and no amount of escalating within Bright Data
  helps. Send those to the Anchor Browser Gateway's browser (§3a).
- ✅ **`scrape_page`'s "looks like a SHELL" note was a raw character count
  and false-positived on genuinely tiny complete pages** (example.com, 183
  characters: heading, sentence, link — measured 2026-09-03). **FIXED
  2026-09-04:** a page carrying a heading AND a real sentence is no longer
  called a shell, so the note now means what it says. The underlying judgment
  is still yours on any other tool — escalate on a near-empty body, not on a
  small one.
- **`browser_page` is one complete, self-contained visit** — open, navigate,
  optionally run in-page JavaScript, read, close. There is no session between
  calls and the remote browser is pinned to one domain. For anything
  multi-step or interactive, use the Anchor Browser Gateway instead. ⭐ **"Pinned to
  one domain" is NOT a reason to avoid in-page fetches — the opposite:
  fetching the site's OWN backend API from inside the js payload is the
  intended way to reach it**, because a same-origin call inherits the page's
  cookies and any bot-defense clearance (§4).
- ⏱ **If `browser_page` text comes back thin, raising `wait_ms` is the
  cheapest thing to try — but do NOT treat it as the diagnosis.** ⚠ Measured
  the hard way on one hospital location-search page: it rendered completely
  for one session at 12000 ms and returned the same 23 characters for another
  session at the same 12000 ms. Timing is variable and proves nothing on its
  own. **The reliable diagnosis is the resource count — see §4.**
- **Oversized results** that exceed the tool-result cap are saved to a file
  path — hand that path onward with explicit slicing instructions rather than
  reading it inline.

### 2c. Structured feeds — check before scraping any well-known site

Purpose-built datasets (LinkedIn profiles/companies/jobs, Indeed, Glassdoor,
Amazon, Walmart, eBay, Zillow, Google Maps, Crunchbase, Reddit, Instagram,
TikTok, X, and YouTube — whose feed returns the FULL video transcript — plus
1,700+ more) return cleaner data and dodge exactly the sites that fight
scrapers hardest. **Reach for a feed whenever a question merely TOUCHES one of
those platforms, even if nobody said the word "scrape."** Discover from the
LIVE catalog (`GET https://api.brightdata.com/datasets/list` with code, or
`search_datasets` through the gateway) — ids are opaque, never guess or
hardcode them. Feeds bill per record (~$0.70/1,000; failed rows bill too —
pass `include_errors=true` and inspect failures). Some feeds are slow (a
YouTube video runs ~2.5 minutes), so expect to go async.

### 2d. Proxy and browser modes — where code executes decides everything

- **Native proxy mode uses port 44445.** The legacy ports 22225/33335 die
  September 25, 2026 (root-certificate expiry). Derive the zone password from
  the API key at run time (`GET /zone/passwords?zone=<zone>`); never freeze a
  composed proxy URL into a secret — frozen credentials die silently at
  rotation.
- **Cloud agent containers cannot reach the superproxy at all.** Proxy mode
  and the direct CDP Browser API work from local machines and CI runners
  (GitHub Actions), not from cloud sandboxes. ⚠ **Judge this by whether the
  connection SUCCEEDS, never by which error it throws** — the same block has
  appeared as a TLS reset mid-handshake and, on another day, as a bare connect
  timeout with nothing answering. No setting fixes it. **In a cloud sandbox
  the real-browser rung is the gateway's `browser_page`** (§2b), which is
  brokered server-side, so the container's own egress is irrelevant.
- **Probing egress correctly:** curl printing `000` does NOT mean blocked.
  The authoritative signal is the proxy's CONNECT response (`curl -v`):
  `403` on CONNECT = policy-blocked; `200 Connection Established` = allowed
  (even if the request then fails for other reasons). A bare unauthenticated
  GET to an API host returning **HTTP 401 is a "path is open" signal**, not a
  failure — useful for probing egress without a key.

## 3. Anchor Browser — pages behind the owner's logins

The last rung: slow, metered browser time. Public data never goes here. Two
doors, and which one you use depends on what you can do.

### 3a. The Anchor Browser Gateway tools — the interactive door, any surface

Tools: `open_site`, `read_page`, `run_code`, `check_auth`, `screenshot`,
`web_task`, `live_view`, `end_session`, `sessions_status`. Flow:
**`open_site` (naming an existing signed-in profile) → `check_auth` →
`read_page` or `run_code` → `end_session`**, with `sessions_status` as the
orphan check.

- 💰 **`read_page` has a context-cost control (new 2026-09-04) — use it.** A
  capture-sized page runs to tens of thousands of characters, and pulling that
  into your conversation just to learn the page loaded is pure waste. Pass
  `mode: "measure"` for the size and title with NO text (the cheapest way to
  confirm a page rendered, and the right way to POLL a slow one), or
  `mode: "preview"` for the opening text only. Both report the page's TRUE
  length, so you can decide before pulling the whole thing; `mode: "full"`
  (the default) is unchanged.
- ⭐ **REUSE a signed-in profile; never log into one that already works.** An
  unnecessary login can end in a password reset that logs the owner out
  everywhere. Verify sign-in from `check_auth` (cookie NAMES), never from how
  the page looks.
- 🔴 **NEVER run a credential login through `run_code`** — the code string
  would put the owner's password into the conversation transcript. When a
  login is genuinely needed, hand the owner a `live_view` link and let them
  type it themselves.
- **`end_session` always, even on failure** — a profile saves its cookies only
  on a clean end.
- **Profile names are DERIVED, not invented:** the site's domain, lowercased,
  dots removed, TLD kept (`consumerreports.org` → `consumerreportsorg`).
  Anchor rejects dotted names (HTTP 400). Because the name is derivable, a
  brand-new session can compose it and find a profile some earlier session
  warmed — so **before ever asking for a login, open the derived name and
  check the cookie jar.**

### 3b. The REST API — the scripted-batch door (shell + key)

- **Session creation:** `POST /v1/sessions` with header `anchor-api-key`.
  ⚠ `captcha_solver` REQUIRES an active proxy configuration — a proxyless
  session with the solver on is rejected (HTTP 400). ⚠ The solver only works
  in HEADFUL sessions; **a named profile cannot be headless at all** (HTTP 400
  "Failed to run browser").
- **Attaching to a running session:** `GET /v1/sessions/{id}` does NOT return
  the CDP URL. Compose `wss://connect.anchorbrowser.io?apiKey=<KEY>&sessionId=<id>`
  in memory at the moment of use. It embeds the key — never print, store, or
  commit it, and never re-quote a connect-failure error (it echoes the URL).
- **`POST /v1/tools/execute-code?sessionId=` runs arbitrary Playwright inside
  a named session over plain HTTPS** — Anchor is fully drivable with no CDP.
  ⚠⚠ **TRAP: `/v1/tools/fetch/webpage` ACCEPTS `?sessionId=` and IGNORES it**,
  spinning its own anonymous browser — so using it to read a logged-in page
  returns the ANONYMOUS page with HTTP 200, a silent auth-stub failure.

### 3c. Login-wall judgment — applies through either door

- **Authenticate on SESSION STATE (the cookie jar), never on rendered
  content.** Sites restyle headers and greetings without notice;
  content-based auth tests break repeatedly.
- **Login-wall trust is all-or-nothing.** While a stored login cookie lives,
  fetches meet no challenge at all. The moment a real login is required, the
  full CAPTCHA wall returns — regardless of how warm or old the profile is.
  On hard-CAPTCHA sites the solver loses the hard interactive classes, so:
  **capture what you need while the cookie is alive; never trigger an
  avoidable login; treat a dead cookie as a human-recovery event.** The one
  proven recovery is a password reset with the emailed link opened INSIDE the
  profile's own browser (paste the URL and navigate the session to it —
  clicking it in the human's own mail app seeds the wrong browser).
- **ONE scripted login attempt, then stop and offer the live view.** More
  attempts cost minutes, risk a lockout, and change nothing.
- ⚠ **Do not be fooled by what a failure LOOKS like.** A CAPTCHA that gets
  SOLVED does not mean access — a site can accept the solve and still refuse.
  And **a "we don't recognize that sign in" message is NOT proof the password
  is wrong**: sites soft-refuse an untrusted browser using the wording of a
  credential error. Before touching a stored credential, ask the owner to try
  it in their own browser.
- **Filling the visible form is often the WRONG way in.** If the login is a
  JavaScript widget, a scripted fill can submit and set no auth cookie at all.
  Look for the login endpoint the page itself calls and POST to it FROM INSIDE
  THE PAGE (`fetch` with credentials included) — an in-page call inherits the
  page's own cookies, including any bot-defense clearance cookie.
- A failed login attempt does NOT damage a stored cookie — retrying a fetch
  afterward is safe.
- Never click, nudge, or "help" a CAPTCHA widget mid-solve — hands off; it
  measurably worsens the solve.
- **Profiles pin their browser fingerprint** across sessions; a sticky IP can
  only be set at profile creation, never added later. Never use Anchor's
  identity/auto-login feature — it fails on CAPTCHA-gated sites (HTTP 422).
- **The live view has no address bar** — a human watching cannot navigate; the
  driving side must do all navigation.
- **Docs:** the ordinary docs pages are JS shells and their `.md` twins
  return the literal string "null". Working routes: `llms-full.txt` (~866 KB
  — grep it) and `openapi.yaml` (~313 KB). Real fields exist in neither —
  the resolved config of a live session (`GET /v1/sessions/{id}`) is the
  ground truth.

## 4. Driving real pages — browser lessons (any browser, any surface)

- **Go under the UI first.** When a page gates, hides, or ignores content the
  session should be able to see, the data is usually already crossing the
  wire. Find the page's own JSON endpoint and call it directly: the page
  context's request facility rides the same cookie jar the page uses.
- ⭐⭐ **DISCOVERING THAT ENDPOINT WITHOUT A NETWORK LISTENER — the trick for
  one-shot renders.** The classic method is to attach network listeners BEFORE
  navigation, then load and **interact** (a no-XHR initial load does NOT mean
  a no-XHR site; many first paints are server-rendered and the endpoints
  reveal themselves on the first interaction). But a one-shot render tool runs
  its JavaScript AFTER load, too late to attach anything. **You do not need
  to: the Performance API retains every URL the page already fetched.** One
  post-load call —
  `performance.getEntriesByType('resource').map(e => e.name)`, filtered to
  drop `.js/.css/.png/.woff2` and friends — hands you the full request list,
  JSON endpoints included. Make this your default first move on any "renders
  in a browser, empty when scraped" listing page.
- ⭐⭐ **AND THE RESOURCE COUNT IS ITSELF THE DIAGNOSIS — this is the tip that
  tells you whether waiting longer can EVER help.** Read it before you reach
  for a bigger `wait_ms`:
  **a near-EMPTY resource list (a beacon or two, no API calls) means the app
  never issued its query at all — more waiting will never help, and the fault
  is in the URL or its parameters.**
  **A FULL resource list with a bare DOM is the genuine timing case** — that
  is when raising the wait is the right move.
  Field-proven: a session stuck on a listing page found exactly ONE non-asset
  entry (an analytics beacon), which reframed the problem from "render too
  slow" to "wrong query parameter" — changing the query string made the app
  run and the endpoints appear immediately. Without this check it had already
  burned a doubled wait on a page that was never going to render.
- 🔑 **Do the token fetch and the API call in the SAME in-page block.** When a
  site's API needs a short-lived token from a companion endpoint, fetch the
  token and call the API inside ONE js payload on a same-origin page. Three
  reasons: the token stays fresh (two round trips can race its expiry), the
  calls inherit the page's cookies, and **the token never enters your
  transcript** — which matters, because tokens are secrets you must not print.
- 📋 **A captured search URL's `facet=` parameters are a free list of the
  filterable field names.** Before guessing filter syntax, read them: a
  request carrying `facet=location_type.name&facet=specialties.specialty`
  is telling you `filter=location_type.name:<value>` is valid. ⚠ And prefer a
  precise filter to a fuzzy text query — a free-text search can sweep in
  loosely-related records (one such search returned 60 rows where the correct
  type filter returned the 46 that actually matched).
- 🎯 **Vendor meta tags name the TECHNOLOGY, not the endpoint.** A page may
  advertise its search vendor, an org id, and a public client-side key in its
  `<head>` — tempting you toward that vendor's public API. The page often
  calls the SITE'S OWN proxy instead, on the site's domain, with a short-lived
  token from a companion token endpoint. Let the resource list settle it
  rather than reasoning from the vendor's name.
- **Read the JSON's real key names before parsing.** Return the top-level keys
  first: fields are often underscore-prefixed (`_locations`,
  `_total_locations`, `_distance`) and the obvious guesses silently return
  empty arrays that look like "no results."
- **Capture once, replay many.** Once an endpoint is captured, parameter
  variants cost one call, not one session each: a `per_page` parameter lifts a
  10-row UI cap to the full set in ONE request, and an `origin`/location
  parameter is usually live (echoed back in the response), so the same
  endpoint answers "nearest to anywhere." **Replay read endpoints only** —
  never replay anything whose name implies mutation (add/save/update/delete)
  with modified parameters.
- ⚠ **Distances from a location API are usually STRAIGHT-LINE.** They can rank
  results badly wherever road geometry dominates — a point 22 straight-line
  miles away over back roads can be a longer drive than one 37 miles away by
  highway. Say which you are reporting; if the answer is about travel, it
  needs drive time, not radius.
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
- **A Cloudflare interstitial is not a failure — wait it out, and prefer a
  profile session.** A profile-backed browser session runs the full stealth
  stack and clears interstitials an anonymous session cannot. Then **poll the
  read WITHOUT re-passing the URL** — re-passing it re-navigates and restarts
  the challenge. Expect ~3–4 polls: 0 chars → a short "performing security
  verification" → correct title with no body → full text. Never conclude
  "empty" before the fourth poll. *(Measured on one Cloudflare-fronted site;
  treat the stealth-stack generalization as promising rather than proven.)*
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
- **Default shell timeouts can be short (~2 minutes on some surfaces), but
  that is usually a DEFAULT, not a cap** — pass an explicit longer timeout
  (ceilings of ~10 minutes have been measured where the default was 2). Design
  cleanup to fire on SIGTERM, because the timeout kill is the likeliest
  failure.
- **Egress varies per surface and per environment.** Do not generalize
  reachability from one container to another; probe (see §2d for reading the
  CONNECT response correctly).
- ⚠ **Help-center sites (Intercom-style and similar) defeat plain scraping in
  a specific way:** a raw markdown scrape can return tens of thousands of
  characters of site NAVIGATION with the article body truncated away. Prefer a
  prompt-directed fetch tool for those, or a real browser, rather than
  trusting the character count as evidence you got the article.

## 6. Maintenance

Master: `https://github.com/HauntGuy/web-access-tips` — maintained by the
owner's web-access project, which folds in new field lessons as they are
proven. Corrections and new tips go to the owner, not into forks. Framed by
capability, kept token-free, one file forever.

*v1.7 — 2026-09-04, a FIX release: two long-standing gateway papercuts are
gone and one new capability exists. The owner's Bright Data Gateway now
returns plain-text files (`.md`, `.txt`, `.json`) byte-faithfully in its
DEFAULT mode — detected from the URL and from the server's real
`Content-Type` — so the `format: "html"` workaround this file used to insist
on is no longer needed for it; its "looks like a SHELL" note no longer fires
on small complete pages; and the Anchor Browser Gateway's `read_page` gained
`mode: "measure"` (size only, no text) and `mode: "preview"` (opening text
only), both reporting the page's true length. The workaround advice is kept
where it still applies — to any OTHER scraper.*

*v1.6 — 2026-09-03 (later the same day), two measured refusal-side lessons
from the owner's connector-icon work: Bright Data refuses Google's own hosts,
the favicon service included — probe those through the Anchor Browser
Gateway's browser instead; and `scrape_page`'s "looks like a SHELL" note is a
character-count heuristic that fires on genuinely tiny complete pages, so
judge by content, not by the note.*

*v1.5 — 2026-09-03, from a session that followed the owner's standing
web-access prompt cold and reported back. The method itself worked; these close
two first-round-trip traps in §0. **Absence from a tool list is not absence from
the account** — where tools load on demand, search for them, separately per
gateway, and expect one search to return a subset. **Identify a gateway by its
TOOL SET, never its name** — that session was told to trust only a connector
named "Anchor Gateway" while the real one is named "Anchor Browser Gateway",
and nearly refused it. Also worth knowing for whoever writes instructions: the
recipe for fetching this file correctly (below) has to be repeated in the
INSTRUCTION, because a first-time reader can otherwise only learn it from an
already-mangled copy. Also in v1.5: the Anchor connector is now called by its
full name, **Anchor Browser Gateway**, everywhere it is named — except where
the short form is deliberately quoted as the WRONG name in the lesson above.*

*v1.4 — 2026-09-02. One correction to the "How to refresh" note, from
testing it directly: a scraping tool's markdown mode does far more than
truncate this file — it collapses newlines, backslash-escapes every markdown
character, DELETES every angle-bracket placeholder, and reports
`truncated: false` regardless. The fix is precise: request raw/HTML output
(`format: "html"` on the gateway's `scrape_page`), which returns the file
byte-faithfully.*

*v1.3 — 2026-09-02, from a live field test of v1.2 by a session that had been
stuck. ⚠ CORRECTS v1.2's wait_ms bullet, which over-generalized one lucky run
into a diagnosis: the same page rendered at 12000 ms for one session and
returned 23 characters for another at the same wait. The reliable test is the
RESOURCE COUNT (§4) — near-empty means the query never fired and waiting
cannot help. Also adds: the token-and-API-in-one-in-page-block rule, `facet=`
parameters as a free list of filterable fields, prefer-a-precise-filter over
fuzzy text search, the same-origin in-page fetch clarification in §2b, and a
warning to verify this file's own version line after fetching it, since a
scraping tool can truncate it in transport.*

*v1.2 — 2026-09-02. Corrected §0, §2 and §3 for the owner's two gateway
connectors (v1.1 predated them and described retired vendor tools); added the
Performance-API endpoint-discovery trick, the underscore-key and
straight-line-distance traps, the Cloudflare poll ladder, the
credential-shaped-refusal warning, and the help-center nav-noise gotcha.*
