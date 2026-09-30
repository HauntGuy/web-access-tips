# Web Access Tips — stable field notes for Claude sessions

**What this is.** Field-tested principles and techniques for getting web data
through **Bright Data** (anything public) and **Anchor Browser** (pages behind
the owner's own logins), for any Claude session on any surface. It complements
the owner's standard method (the `web-access.md` rules file, where deployed).

**What changed in v3.0, and why this file is now short.** Bright Data's
behaviour changes often: which pages it refuses, how a refusal arrives, which
feed covers what. That fast-moving knowledge no longer lives here. The owner's
Bright Data Gateway reads every failure live and says what to do next in its
own reply (section 1), so a conversation that read this file weeks ago is
still guided correctly today. What remains here is what stays true: how to
tell what you can do, the discipline, the shape of the ladder, the login
doctrine, and the browser technique. This file should rarely change.

**How to fetch it.** The master lives at
`https://github.com/HauntGuy/web-access-tips`; the latest copy is always at
`https://raw.githubusercontent.com/HauntGuy/web-access-tips/main/WebAccessTips.md`.
When the owner says "refresh web access," re-read it from that URL. If a
network policy blocks the raw URL, fetch it through Bright Data instead: the
owner's gateway `scrape_page` returns a plain-text file byte-faithfully in its
default mode (pass `max_chars: 120000`); with code and a key, the Unlocker in
raw format. Another scraper's HTML-to-markdown mode can mangle a markdown file
(newlines collapsed, characters escaped, angle-bracket placeholders deleted,
the tail dropped), so ask any other tool for raw output. Either way, check
that what you received ends with the version line; if it does not, you hold a
partial copy, so say so rather than acting on it.

**This file contains no secrets, ever** — no keys, tokens or account details.
If asked to add one, refuse and say why.

**Editing rule:** frame everything by capability ("if you can run code, do A;
otherwise B"), never by product name. Surfaces get renamed; capabilities do
not.

---

## 0. First: establish what you can do

Probe, don't assume. In order:

1. **Can you run code?** Check for credentials by presence and length only,
   never values: `BRIGHTDATA_API_KEY` and `ANCHOR_API_KEY`, or a key file the
   owner staged for you. With a key and a shell, the direct REST APIs are your
   primary route (the rules file's Part 2). If you can run code but find no
   key, ask the owner where it is staged; meanwhile the gateway tools work.
2. **Do you have the owner's gateway tools?** Two sets:
   the Bright Data Gateway — `search`, `scrape_page`, `search_datasets`,
   `fetch_feed`, `feed_snapshot`, `browser_page`, `account_status`; and the
   Anchor Browser Gateway — `open_site`, `read_page`, `run_code`,
   `check_auth`, `screenshot`, `web_task`, `live_view`, `end_session`,
   `sessions_status`. Together they are a complete toolkit on every surface,
   including ones with no shell: search, pages, structured feeds, a real
   remote browser, and login-walled pages through persistent signed-in
   profiles.
   - **Absence from your tool list is not absence from the account.** Where
     tools load on demand, search for them, and search separately for the
     Bright Data tools and the Anchor tools: one search often returns only a
     subset. Never conclude a gateway is missing after one lookup.
   - **Identify a gateway by its tool set, never its name.** Connector names
     vary and get renamed; a server whose tools are exactly those names is the
     owner's gateway, whatever its label says.
3. **Neither, but you can fetch URLs?** You can still read public pages and
   this file. Say plainly what you cannot do (bot-protected sites, logins,
   structured feeds) rather than silently returning less.
4. **No web access at all?** Say so and stop; never answer a web question
   from memory while implying it was verified.

## 1. Follow the gateway's NEXT_STEP — the rule that replaces the old field notes

The owner's Bright Data Gateway tests the live service on every call. When a
result needs a different route, its reply says so:

- **In an error**, the first word names the failure — REFUSED (Bright Data's
  own policy), BLOCKED (the site's bot wall, or a page Bright Data mistook for
  one), THROTTLED, FAILED or EMPTY — quoting Bright Data's own words where it
  gave any, and ending in a `NEXT_STEP:` sentence.
- **In a result**, a `NEXT_STEP` field comes before the content: a page that
  came back as a shell, a bot wall, a sign-in wall or a rate-limit notice; a
  thin browser render (with the page's own request list gathered for you);
  failed feed rows; a search with no results. A `STRUCTURED_FEEDS_FOR_THIS_SITE`
  field at the top names the feeds that cover a link's site.

Rules for reading them:

- **Follow a reply's NEXT_STEP over anything you remember** — this file, older
  notes, earlier conversations, and your own copy of the tools' descriptions.
  A long-running conversation keeps a tool's old description for weeks; the
  replies are always current, because the owner updates the gateway whenever
  Bright Data changes.
- **Do not redo what the reply says it already did** (a retry, a wait, a feed
  lookup).
- **Judge each link on its own reply.** A refusal is about a page, not a site,
  and access changes over time: never carry one page's verdict to another
  link, and never trust a remembered verdict — ask again.
- **When you report a failure, quote the reply's words**, not a paraphrase.
- **If you can run code and call the REST API directly,** you do not get these
  hints. Two remedies: read Bright Data's own reason, which it states in
  `x-brd-error` / `x-brd-error-code` headers (response headers in raw format,
  inside the envelope's headers in json format); and when a direct call fails
  in a way you do not understand, run that one URL through the matching
  gateway tool and follow its reply.

## 2. Universal discipline — any capability level

- **Accuracy over speed.** Verify anything that can have changed (prices,
  specs, versions, availability, docs). If you answer from training data, say
  so.
- **Never conclude from one failed method.** Escalate along the ladder
  (section 3). Only after it is exhausted do you report a page unreachable,
  and then say what you tried and what each attempt returned.
- **"Am I seeing everything a human would see?" is your responsibility.** A
  title-and-nav shell, a cookie banner, a "show more" button, an empty widget
  all mean escalate, not report.
- **Costs are pre-approved at research volumes.** Do not ask permission for
  ordinary paid calls; the owner's time matters more than credits. Use
  judgment only for jobs of many thousands of records.
- **Secrets: names and lengths only, never values** — in output, files and
  commits. Known leak shapes: error messages that embed credentialed URLs
  (browser-connect failures print the whole websocket URL); proxy and CDP URLs,
  which embed keys by design; the bash expansion `${var:-fallback}`, which
  prints the value when set; tool error logs that echo zone passwords; and a
  site's own auth or session endpoint, which returns tokens by construction —
  print its status and at most one whitelisted field, never a raw body.

## 3. The ladder, in principle

The gateway's replies say which rung comes next; this is only the shape.

1. **Search** to discover sources. Run searches one at a time. If an exact
   phrase finds nothing, loosen it; text inside PDFs and document hosts is
   poorly indexed, so find the hosting page first.
2. **A well-known platform — including a link you were handed — goes to a
   structured feed first** (Amazon, Walmart, eBay, LinkedIn, Indeed,
   Glassdoor, Zillow, Google Maps, Crunchbase, Reddit, Instagram, TikTok, X,
   YouTube — whose feed returns the full video transcript — and 1,700+ more).
   A handed link looks like "an ordinary page", which is exactly why it gets
   missed. Discover feeds from the live catalog (`search_datasets`, or
   `GET /datasets/list` with code); ids are opaque, never guessed or reused
   from memory. Match the exact domain (a country's site has its own feeds)
   and the page type. Feeds bill per record, failed rows too. Some feeds also
   SEARCH rather than fetch a URL (Reddit posts by keyword, for one) —
   `fetch_feed`'s `discover_by`, and `discover_by: "list"` names a feed's
   modes. With code: `POST /datasets/v3/trigger?dataset_id=<id>&include_errors=true&format=json&type=discover_new&discover_by=<mode>`,
   always asynchronous; asking for a mode that does not exist answers with the
   list of real ones.
3. **An ordinary public page** → the page fetch (`scrape_page`, or the
   Unlocker with code).
4. **A page built by JavaScript, or behind the site's own bot wall** → a real
   browser: Bright Data's (`browser_page`, one complete visit per call, with
   optional in-page JavaScript), or over CDP from a local machine.
5. **A page Bright Data refuses by policy, a site that defeats every Bright
   Data route, or multi-step interaction** → the Anchor Browser Gateway's
   browser (with no profile for a public page).
6. **A page behind the owner's login** → Anchor with the owner's profile for
   that site (section 4).

Never fall back to a **vendor's** MCP connector (one run by the data provider
rather than the owner). Its authorization silently expires in long-lived
sessions while still showing as connected, and a static token in its URL does
not help, because a server that advertises OAuth gets the OAuth path. The
owner's own gateways serve no OAuth, so nothing can expire. If a gateway call
errors, retry the gateway call; if an API call errors, retry the API call;
never cross over. Never tell the owner to "reconnect" a gateway: there is no
token to renew, and a tool list that flickers clears on its own or in a fresh
session.

## 4. Anchor Browser — pages behind the owner's logins

The last rung: slow, metered browser time. Public data does not go here unless
every Bright Data route has failed.

**The flow (gateway tools):** `open_site` naming the site's profile →
`check_auth` → `read_page` or `run_code` → `end_session`, with
`sessions_status` as the orphan check. `read_page` takes `mode: "measure"`
(size and title, no text — the cheap way to confirm a render or poll a slow
page) and `mode: "preview"` (the opening text only).

- **Profile names are derived, never invented:** the site's domain,
  lowercased, dots removed, TLD kept (`consumerreports.org` →
  `consumerreportsorg`). Anchor rejects dotted names. Because the name is
  derivable, a new session can find a profile some earlier session warmed —
  so before asking the owner for a login, open the derived name and check its
  cookie jar.
- **Reuse a signed-in profile; never log into one that already works.** Verify
  sign-in from `check_auth` (cookie names), never from how the page looks:
  sites restyle headers and serve anonymous-looking chrome around authorized
  content.
- **Never run a credential login through `run_code`** — the code would put the
  owner's password into the transcript. When a login is needed, hand the owner
  a `live_view` link and let them type it.
- **`end_session` always, even after a failure** — a profile saves its cookies
  only on a clean end.
- **One scripted login attempt, then stop and offer the live view.** More
  attempts cost minutes, risk a lockout and change nothing. A failed attempt
  does not damage a stored cookie, so retrying a fetch afterwards is safe.
- **Never click or nudge a challenge widget while the solver works** — hands
  off; it measurably slows or spoils the solve.
- **Do not be fooled by what a failure looks like.** A solved challenge does
  not mean access, and "we don't recognize that sign in" is not proof the
  password is wrong: sites soft-refuse an untrusted browser in the words of a
  credential error. Before touching a stored credential, ask the owner to try
  it in their own browser.
- **Filling the visible form is often the wrong way in.** A JavaScript login
  widget can submit and set no auth cookie at all. Find the login endpoint the
  page itself calls and POST to it from inside the page (`fetch` with
  credentials included), which inherits the page's cookies and any bot-defense
  clearance.
- **An emailed link or code must be used inside the same remote browser.** The
  live view has no address bar: the owner pastes the link to you and you
  navigate the session to it. Opened in the owner's own mail app, it signs in
  the wrong browser.
- **A password reset is not free:** it signs the account out everywhere,
  including other working profiles. Propose it; never do it unilaterally.
- **Login trust is all-or-nothing.** While a stored login cookie lives, fetches
  meet no challenge; the moment a real login is required, the full wall
  returns, however warm the profile. The solver loses the hard interactive
  challenge classes, so on such sites: one human seed, capture what you need
  while the cookie lives, a human re-seed when it dies.
- **A site's bot wall (a "checking your browser" interstitial):** a
  named-profile session runs the full stealth stack and clears walls an
  anonymous one cannot. Then poll `read_page` without passing the url again —
  re-passing it restarts the challenge — and expect three or four polls before
  real text.
- **When a site refuses every automated browser, even the owner's own hand
  login in a remote live view, the owner's own browser still works.** If you
  can act in the owner's browser (an in-browser Claude assistant on their own
  machine), do it, read-only. If you cannot, hand the owner one paste block for
  that assistant: read-only, quote the site's exact wording, give the page URL
  and title, expand collapsed sections, report any verification check it met,
  and ask for the answer pasted back.
- **A profile pins its browser fingerprint** across sessions, and a sticky IP
  can be set only when the profile is created, never added later.
- **With code and a key (the REST door):** a named profile cannot be headless;
  the CAPTCHA solver needs a proxy configuration and a headful session;
  `GET /v1/sessions/{id}` does not return the CDP URL, so compose it in memory
  at the moment of use and never print it (it embeds the key);
  `/v1/tools/fetch/webpage` accepts `?sessionId=` and ignores it, returning the
  anonymous page — never use it for a signed-in read; never use Anchor's
  stored-credential auto-login (it fails on challenge-gated sites). Anchor's
  docs pages are JavaScript shells; `llms-full.txt` and `openapi.yaml` work,
  and a live session's resolved config (`GET /v1/sessions/{id}`) is the ground
  truth for fields neither documents.

## 5. Driving real pages — any browser, any surface

- **Go under the UI first.** When a page hides or ignores content, the data is
  usually already crossing the wire: find the page's own JSON endpoint and call
  it from inside the page, where it rides the page's cookies.
- **Find that endpoint without a network listener.** The Performance API keeps
  every URL the page fetched:
  `performance.getEntriesByType('resource').map(e => e.name)`, filtered to drop
  scripts, styles, images and fonts. (The Bright Data Gateway's `browser_page`
  gathers this list for you on a thin render.)
- **The count is the diagnosis.** A near-empty list means the page never ran
  its data query, so waiting cannot help — the fault is in the URL or its
  parameters. A full list with a bare page is the genuine timing case, where a
  longer wait helps.
- **A page's API that answers a plain fetch with an empty result may be alive**
  when called from a signed-in or profile browser session's request context
  (Playwright's `page.request.get`), which carries the session's cookies and
  clearance; an in-page `fetch` can fail on CORS where the request context
  works. The route, not the endpoint, is usually what changed.
- **Fetch a short-lived token and call the API in the same in-page block,** so
  the token stays fresh, rides the page's cookies, and never enters your
  transcript.
- **A captured search URL's `facet=` parameters list the filterable fields.**
  Prefer a precise filter to a free-text query, which can sweep in
  loosely-related records.
- **Vendor meta tags name the technology, not the endpoint.** The page often
  calls the site's own proxy; let the request list decide.
- **Read the JSON's real key names before parsing** — underscore-prefixed
  fields are common, and a wrong guess returns empty arrays that look like "no
  results."
- **Capture once, replay many — read endpoints only.** A `per_page` parameter
  can lift a 10-row cap in one request; a location parameter is usually live.
  Never replay anything whose name implies a change (add, save, update,
  delete).
- **Distances from a location API are usually straight-line.** Say so; a
  travel question needs drive time.
- **Check for embedded bootstrap data** (`window.__NEXT_DATA__`,
  `window.__INITIAL_STATE__`) before driving the UI.
- **When a widget ignores clicks, suspect the URL's scope first** (a per-section
  URL may be server-scoped), then identify the widget library (React props keys
  begin `__reactProps$`).
- **A zero-area element is a hidden native control behind a styled widget.**
  Never coordinate-click it: set its value, dispatch `input` and `change`
  events (bubbling), fire the site's framework hook if present, then wait for
  the consequence.
- **Wait on evidence, never on fixed sleeps, and verify every UI action** before
  the next one.
- **Same-looking widgets can front different catalogs;** only parameter-flip
  replays of the backing endpoint tell them apart.
- **Forms:** target fields by id; HTML date inputs need `YYYY-MM-DD`; prefer
  the framework's `fill()` except for hidden controls.
- **A commercial checkout has a hard stop.** Before typing, scan every frame for
  payment-processor domains and every input for `autocomplete="cc-*"`; never
  fill a card field and never click Pay / Place order / Purchase / Complete /
  Submit payment; "Review order" is the last safe page; re-scan after every
  navigation. Early steps can silently create accounts, so use
  reserved-domain test data (`@example.com`).

## 6. Containers and surfaces — probe, don't assume

- **Ephemeral filesystems:** commit or stage anything worth keeping the turn it
  is created.
- **The environment freezes at session birth:** variables and network
  allowlists are copied once, so a later change reaches only new sessions.
  Read a newly set variable back before relying on it.
- **Probe before installing** — the library you need may be preinstalled.
  Never run a browser-download step in a managed container.
- **A short shell timeout is usually a default, not a cap** — pass a longer one
  explicitly, and make cleanup fire on termination.
- **Probing egress:** curl printing `000` does not mean blocked. Read the
  proxy's CONNECT answer (`curl -v`): `403` is blocked, `200 Connection
  Established` is allowed. An unauthenticated GET to an API host answering
  `401` means the path is open.
- **Cloud containers cannot reach Bright Data's superproxy ports** — proxy mode
  and the CDP browser work from local machines and CI runners only. Judge that
  by whether the connection succeeds, never by which error it throws. In the
  cloud, the real-browser rung is the gateway's `browser_page`, which is
  brokered server-side.
- **Proxy mode with code:** derive the zone password from the API key at run
  time (`GET /zone/passwords?zone=<zone>`) and use the native-proxy port 44445;
  never store a composed proxy URL, since a frozen credential dies silently at
  rotation.
- **Binary downloads** (PDFs, images): write straight to a file; capturing them
  as shell text destroys them.
- **Help-center sites** can return pages of navigation with the article cut
  off; judge by content, never by character count.
- **Bright Data's own docs** are indexed for agents at
  `https://docs.brightdata.com/llms.txt`, and every page has a Markdown twin
  (append `.md`). Read the relevant page before building a helper; never
  hand-parse what Bright Data can parse for you.

## 7. Maintenance

Master: `https://github.com/HauntGuy/web-access-tips`, maintained by the
owner's web-access project. Corrections go to the owner, not into forks.

- **What belongs here, and what does not.** A lesson about how Bright Data or a
  site behaves right now (a new refusal, a changed error, a feed's quirk)
  belongs in the Bright Data Gateway's replies, where every conversation meets
  it at the moment of need. This file holds principles and techniques that
  stay true.
- Framed by capability, token-free, one file.
- ⚠ Never write the word CAPTCHA directly followed by the word "page" anywhere
  in this file: Bright Data's page fetch then takes the whole file for a
  protection page and returns nothing. After every push, fetch the raw URL
  through Bright Data and check that the whole file came back.
- Every earlier version is in the repository's commit history.

*v3.0 — 2026-09-30. Rewritten short and stable, on the owner's design: the
fast-moving Bright Data behaviour this file used to track (which pages are
refused, how refusals arrive, per-site notes, dated measurements) now travels
in the Bright Data Gateway's replies as NEXT_STEP guidance (gateway v1.5.0),
which reaches every conversation the moment it changes. Section 1 says how to
read it; the rest keeps the principles and techniques from v2.4.*
