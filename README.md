# site/: Rally web presence (static, no build step)

The landing page, the legal pages and the live scoreboard, as plain
HTML and one shared stylesheet. Layout follows the kit convention
(app-starter `docs/launch-playbook.md` section 3b, reference layout
`denisfenger/pomini`): folder + `index.html` style so every URL ends
in `/`.

```
.nojekyll            GitHub Pages: serve as-is, no Jekyll pass
README.md            this file
icon.svg             the court-sides mark on the court-night ground (favicon)
style.css            shared tokens and components (STYLE_GUIDE.md palette)
app-store-badge.svg  Apple's "Download on the App Store" badge, en-us black,
                     Apple's file byte for byte (see "Apple's artwork")
devices/             Apple's product images for the landing: Apple Watch
                     Series 11 46 mm Jet Black (Apple's PNG untouched) and
                     iPhone 18 Pro Max Black (a uniform 13/30 copy)
index.html           landing: hero with the App Store badge and a working
                     watch scorer (engine-faithful padel: golden point or
                     advantage, tiebreak, serve rotation, viewer-perspective
                     serve ball, the round 28 deciding-point card at 40:40)
                     under Apple's watch image, how it works, sports, live
                     sharing (the iPhone Board under Apple's iPhone image runs
                     the same match as the watch's mirror), trust, free and
                     Pro (features only, no prices), Apple's credit line in
                     the footer
privacy/index.html   privacy policy, EN with the DE version beneath
terms/index.html     terms of use, EN with the DE version beneath
support/index.html   contact, what the app is, common questions
impressum/index.html legal notice (address, register and VAT ID: TODO Denis)
board/index.html     live scoreboard for a share code (self-contained;
                     verified by scripts/verify-board.mjs)
```

## How it is published

Public GitHub Pages PROJECT repo under the `denisfenger` account, Pages
served from `main` / root. Kit naming rule (playbook 3b): the public
site repo takes the bare app name, `denisfenger/rally`; the private app
source is `denisfenger/rally-app`. Since 2026-10-03 the site lives at
the custom domain `https://rally.softailed.com` (Cloudflare CNAME `rally`
-> `denisfenger.github.io`, DNS only; GitHub Pages custom domain with
HTTPS enforced; `CNAME` in this folder keeps it across mirrors), and
every old `https://denisfenger.github.io/rally/...` address answers 301
to the same path on the new domain. The App Store listing links to the
new domain since the 1.0.0 resubmission (2026-10-12). The base the app
emits is the ONE constant `SITE_BASE` in `src/links.ts`, still the old
address in 1.0 (it forwards); it moves to the new domain in the first
update (feedback round 33, item 2). Every URL the app emits derives from
it:

| helper          | URL shape                        | used by                          |
|-----------------|----------------------------------|----------------------------------|
| `privacyUrl()`  | `SITE_BASE/privacy/`             | paywall, Settings, ASC privacy URL (docs/privacy-labels.md) |
| `termsUrl()`    | `SITE_BASE/terms/`               | paywall, Settings                |
| `supportUrl()`  | `SITE_BASE/support/`             | ASC support URL                  |
| `boardUrl(code)`| `SITE_BASE/board/?code=123456`   | share sheet message              |

Should the host ever move (a bought domain once the app has proven
itself), change `SITE_BASE` once; nothing else in the app spells the
host.

Steps (the PAT usually 403s on repo creation and on the Pages flip, so
these are Denis's):

1. Create the public repo, push the CONTENTS of this folder to its root
   (not the folder itself).
2. Settings > Pages > Deploy from a branch > `main` / `/ (root)`.
3. GET every URL once the build is green: `/`, `/privacy/`, `/terms/`,
   `/support/`, `/impressum/`, `/board/`, `/board/?code=123456`.

Later content pushes work with the headless git recipe.

## Rules that hold on every page

- INDEXED since release day: no page carries a robots meta tag. Until
  Rally was on the App Store every page carried `<meta name="robots"
  content="noindex, nofollow">` (a `robots.txt` in this repo does nothing
  for a project page: crawlers read it only at the domain root); the
  release-day push removed it from all six pages, and verify-site and
  verify-board fail if it comes back.
- NO external requests: system font stack, no Google Fonts, no CDN, no
  analytics, no remote images. Apple's badge and product images are
  files in this folder. The only network calls on the whole
  site are the board's, all to the ONE Supabase project that is the
  relay: its two edge-function POSTs (`share-join`, `share-state`)
  and, since feedback round 27, one WebSocket to the project's Realtime
  endpoint (the channel the iPhone follow screen uses). The board
  stores ONE viewer preference in localStorage (Swap sides, per
  browser) and nothing about any match; the round 14 side choice is
  gone from the link and its key is removed on load.
  Verify with
  `grep -rn "http[s]*://" site/` before every push: the supabase.co
  calls in `board/index.html` and `mailto:` links are the only hits
  allowed, plus the App Store listing `https://apps.apple.com/app/id6807145117`
  as two LINKS (the landing's badge and the board's "Scored with Rally";
  a link loads nothing until it is followed) and the SVG namespace in
  `icon.svg` and `app-store-badge.svg`.
- NO pricing on the landing (the store is the source of truth).
- CONTENT CURRENCY: the landing describes features, so any round that
  adds or removes a feature updates `index.html` in the same round.
  (Round 28: the watch mock asks the deciding point's side with the
  app's card, and the workout card says the workout pauses when nobody
  scores for ten minutes. Round 30: the sharing section says the watch
  sends the score over its own Wi-Fi or cellular, so the phone can stay
  at home; the privacy policy names the device that keeps the score as
  the sender, with the iPhone as a backup, and the rate limit's IP as
  that device's; support answers "Can I share without my iPhone?".
  Round 30 verify: terms section 4 says followers of a shared match see
  the player names you added (the "no field for names" line was stale
  since round 7), and the privacy policy's rate-limit line names the
  iPad beside the iPhone.)
- Every control reads as one: `cursor: pointer`, a visible hover and a
  visible pressed state. The one element that listens for clicks and is
  NOT a control is the landing's deciding-point card: it catches clicks
  only to refuse them, so it keeps the default cursor (the verifier
  checks both sides of that). The App Store badge is a control whose
  states never touch the badge (Apple forbids modifying or animating
  it): they are a ring drawn just outside its clear space.
- House voice: sentence case, no emoji, no exclamation marks, no em
  dashes in visible copy.

## Apple's artwork (release day)

Apple's artwork licence covers only an app that is on the App Store, so
the landing drew its own watch and iPhone and showed a "Coming to the App
Store" pill until Rally 1.0 was approved (Denis, 2026-10-01); the
release-day push swapped both, with noindex dropped, in one commit.
Sources: App Store Marketing Guidelines (badges, product images, legal
requirements) and the slides' notes on the device images
(`../Rally-scratch/2026-10-01/rally-slides/assets/rules.md` and
`devices.json`, outside this repo).

- THE BADGE (`app-store-badge.svg`): Apple's "Download on the App Store"
  badge, US-UK black (the landing is English), exactly as
  `toolbox.marketingtools.apple.com/api/v2/badges/download-on-the-app-store/black/en-us`
  and the guidelines page serve it (10,804 bytes, LF line ends; sha256
  `a26fc5b3...`, which verify-site pins). Never redraw, recolour, crop,
  angle, animate or re-export it; a new badge is a new download. Rules
  kept: ONE badge on the page, in a subordinate position under the
  headline, 56 px high (Apple's minimum onscreen is 40 px), a clear space
  of a quarter of its height (14 px) around it with nothing in it, its
  link the listing `https://apps.apple.com/app/id6807145117`. Its hover,
  pressed and focus states are a 2 px ring at `outline-offset: 14px`, so
  the badge itself never changes. Apple's credit line for the badge and
  the devices stands once, in the landing's footer (international form,
  no symbols).
- THE DEVICES (`devices/`): the App Store slides' files from Apple Design
  Resources (Product Bezels). Apple Watch Series 11, 46 mm, Jet Black
  aluminium with the Black Sport Band, is Apple's own 560 x 880 PNG,
  untouched, shown at 280 x 440 (2x). iPhone 18 Pro Max, Black, is a copy
  of Apple's 1470 x 3000 PNG at exactly 13/30 on both axes (637 x 1300,
  area-averaged on premultiplied alpha, nothing else changed), shown at
  318.5 x 650 (2x). Never upscale past Apple's pixel size. Each device is
  shown whole, upright, uniformly scaled, with no shadow, glow,
  reflection or filter and nothing on top of it or overlapping it. The
  live screen lies UNDER the image's transparent display: a box at
  devices.json `ui.box`, clipped to `ui.clipPath`, the screen at
  `ui.contentOffset` (it fills Apple's display rectangle exactly), and the
  image passes the pointer through (`pointer-events: none`), so the
  demo's tiles take the taps. The watch's score page is laid out at the
  46 mm display's own 208 x 248 pt, as the slides lay it out; the iPhone
  draws no island of its own (the image has the real one) and centres
  its status bar on it with full signal, Wi-Fi and battery (Apple's
  status bar rule). Every length on a device's screen is a multiple of
  `--u` (one CSS pixel at full size, from container query units), so
  below its full width the image and its screen scale as one.
- RELATIVE SIZE: the two devices are NOT at true relative size (at true
  scale the watch's display would be about 128 px wide beside a 650 px
  iPhone, too small to play the demo). Apple asks for the correct
  relative scale whenever several products are shown, so they stay in
  separate bands, never in one ordinary viewport: verify-site requires at
  least 1,000 px between them at every width (about 1,850 at 1280, 2,600
  at 390; only a screen taller than that, a portrait 4K monitor, could
  show both at once). A layout that brings them together must put them at
  true relative size (devices.json `relativeScale`).
- The device files may not be redistributed (Design Resources licence):
  they are served only as the page's images, with no download link, and
  only while Rally is on the App Store and Fenger Digital GmbH is in the
  Apple Developer Program.

## The board

`board/index.html` is self-contained. It joins through `share-join`
(POST `{code}`), then (feedback round 27) LISTENS: it joins the relay's
Realtime channel `share:<shareId>` over one WebSocket (Phoenix protocol
v1, the anon key, a heartbeat every 25 s, reconnecting after 1, 2, 5
and 10 s), so a score lands about 0.36 s after the host's upload
(`node tools/board-latency.mjs` measures it end to end on the live
relay; the 2.5 s poller before it took 1.3 to 1.8 s). While the channel
is joined, `share-state` (POST `{shareId}`) is asked every 10 s as the
safety net; whenever it is not (before it joins, after it drops, a
browser without WebSocket) the page asks every 2.5 s with one request
in flight, backs off to 10 s after three consecutive failures, pauses
while the tab is hidden and resumes on `visibilitychange`. The dot
turns amber after 20 s without a fresh answer. No `?code=` shows the
entry form, which submits to `board/?code=` so the URL shape the app
shares stays stable. An X in the header leaves the match for the entry
form. The page speaks every launch language, English, German,
Spanish, French, Italian and Portuguese (round 31 verify; it spoke
English and German until then), by the browser's language list (the
primary subtag: pt-BR reads Portuguese), else English; `?lang=` forces
one. The footer's "Scored with Rally" (in the page's language) opens the
App Store listing in a new tab since release day, so a follower reaches
the app without losing the live score; it is a link, not a request.
THE LINK IS NEUTRAL (round 27): no side
choice, the result reads "<names> won"; Swap sides stays. It draws the
app's Board: tinted tiles in initials, the optic serve ball placed from
the viewer's side with the serving player's name (the host uploads
`serverName` inside the snapshot, never an index: the name lists are
compacted), the full names in a 1 v 1 only on the host's `singles:
true`, the sets ALWAYS below the tiles, a saved sport by its own name
(`sportName` inside the snapshot).

A SHARE IS A MATCH (feedback round 14, decision 6, reversing round 7
decision 12), so the page is one match long:

- the host's final uplink is the ENDED snapshot, rendered as the
  result: the winner tag on the winning tile, no serve dot, the
  status and the banner worded as a result. It is TERMINAL: polling
  stops there and the result stays on screen (feedback round 24: the
  relay keeps a scored end for 24 hours so players can save it, so
  `share-state` goes on answering and no 404 follows for a day);
- a 404 WITHOUT an ended snapshot (the host stopped sharing
  mid-match, or the share idled past the relay's 2 h TTL) keeps the
  last score with "sharing ended" and one explaining line;
- nothing rolls over: a new `matchId` under the same code cannot
  happen on a round 14 host, and the page has no session strip and no
  "waiting for the next match" state any more;
- YOUR TEAM (decision 7): a Team A / Neutral / Team B control under
  the score, Neutral by default. A side marks its tile with "you" and
  words the result from that side ("you won", "you lost"); Neutral
  words it as "Team A won" (or the names). The choice belongs to ONE
  SHARE (round 14 verify: a side picked last week says nothing about
  this match): it is remembered in localStorage under
  `rally.board.team` as `{"share": <shareId>, "team": "A"|"B"}`, so a
  reload of the same code keeps it and any other share starts Neutral
  (the entry is dropped), the way the app's follow screens do. The
  tile COLOURS never change with it: blue is the host's team and clay
  the other on every Rally screen;
- SWAP SIDES (decision 7): the arrows button beside the chooser, the
  same per-device VIEW setting the Board and the watch have. On, team
  B is drawn left and A right (on a desktop the sets table stays
  between them; on a phone it stays beneath), the chooser's team
  buttons change places with their tiles, and nothing else moves:
  colours, serve dot, names and the sets table's rows stay with their
  teams. Off is the round-1 default (the host's team left).
  Remembered per browser under `rally.board.swap` = "1", for every
  share. Those two keys are ALL the page stores;
- participants' NAMES travel with a share (decision 10). When
  `participants` is present the tiles and the chooser read them,
  joined with " & " and shortened to the shortest unambiguous form
  within the match (decision 11: first name, then first name plus
  surname initial, then the full name). Without them they read Team A
  and Team B. Ids never reach this page, and a name change mid-match
  relabels the tiles on the next poll, without a point.

Round 7's session page froze on the first finished match of a session
(the `seq <= curSeq` guard dropped the next match's seq 1, and the
ended flag cleared the poll timer for good), which a live smoke test
caught on 2026-09-18; the per-match page has one seq space for its
whole life.

Why polling and not Realtime: the earlier board pulled `supabase-js`
from a CDN for the broadcast channel, which the no-external-requests
rule forbids; `share-state` exists for the watch follow poll anyway and
carries no per-IP rate limit by design. The page embedded in
`supabase/functions/board/index.ts` is the legacy Realtime twin that
`*.supabase.co` serves as text/plain; this static page supersedes it,
and the edge function can be removed at the next backend pass.

## The landing's watch mock at the deciding point (feedback round 28)

The hero watch runs the padel preset. At 40:40 with golden point the
receivers pick the side, so, as on the watch, an opaque card lies over
the two tiles (never over the undo row) until it is answered: the lead
line ("You set the side." / "Your opponents set the side."), the
question from the wearer's position ("Which side do you receive on?"
when they serve, "Which side do you serve from?" when we do) and two
big neutral buttons, Left on the left and Right on the right, never
mirrored, never preselected, each with the court drawing (our half at
the bottom, the receiving box in the receivers' colour, the optic
server dot and a dashed serve line into the box). The tiles take no
point while it shows; a click on the card outside the buttons, or on a
button in the first 0.7 s, is refused with a small nudge (the watch's
failure haptic). The status band reads "Golden point" from the open
side until the point is played, "Padel" otherwise. While the side is
open the ball is centred (engine `serveCourtSide` none), which only
the phone shows; after the answer the ball sits where the server
stands. Undo after an answer brings the card back; undo of the
deciding point keeps the answer. The phone mock is the MIRROR of the
watch's match: it shows no card, and a tap on it still scores (the
engine never refuses a point from the other device), which closes the
watch's card with the side unanswered. `window.rallyDemo` exposes
`cardUp()`, `cardAgeMs()` and `armMs` for the verifier.

## Verifying before a push

`node scripts/verify-site.mjs` drives the landing in headless Edge
over the DevTools protocol (no dependencies) and exits 1 on any
failure: it plays a scripted three-set padel match (golden point and
advantage) against an independent reference model and asserts the
serve ball's tile, edge and court side after every point on both
mockups (the rule: our serve at the near/bottom edge below the number
on the serve court side, right when the game's point total is even and
left when odd; their serve at the far/top edge above the number with
the sides mirrored; CENTRED while the receivers' side is open, then
the answer until the game ends); checks the deciding-point card end to
end (shown exactly while the side is open; laid exactly over the tiles
and clear of the undo row; opaque; question and lead from our position
in both roles; Left left, Right right, neither preselected; each court
drawing by its geometry and colours; real mouse clicks on the covered
tiles land on the card and score nothing, programmatic clicks and
Enter are refused, a button click inside 0.7 s is refused; answers by
real clicks move the ball to the server's corner on both mockups; undo
after the answer brings the card back, undo of the deciding point keeps
the answer; the phone still scores; the status band names the point;
the card fits the watch at 390 in both roles); checks that the
retired "Receive / Left / Right" capsule is gone and that the workout
card mentions the pause; checks that no sport card is selected on
load; lists every element with a click handler or interactive role and
requires a computed `cursor: pointer` (the card's two buttons
included; the card itself must keep the default cursor) plus a visible
hover and pressed state under real mouse events (the card buttons
too, their release answering); renders 320, 390, 768 and 1280 full-page
PNGs with overflow probes; asserts no external request, no console
error and no running animation under `prefers-reduced-motion:
reduce`. RELEASE DAY (Apple's artwork, above): no page carries a robots
meta or pre-release wording ("coming soon", "TestFlight", "at
launch"...); the badge is Apple's file (sha256 over LF), local, one on
the page, linked to the listing, at least 40 px high at every width
with its clear space empty (no element's box enters it), and under real
pointer events its hover and pressed rings stand outside the clear space
while the badge itself does not change (the press is released with
navigation blocked, so nothing leaves for Apple); the device files are
the reviewed ones (sha256, natural size, never above Apple's), and at
each of the four widths each device loads from the site, keeps its
natural aspect, is about 2x its displayed size, stands at least 200 px
high, lies wholly inside the page and every clipping box, carries no
transform, filter, opacity, shadow, clip or mask anywhere up its tree,
paints over its screen with the pointer passing through, has nothing on
top of it (187 sample points over its silhouette hit the image) and
nothing overlapping it (no element's box meets its silhouette), and its
live screen fills Apple's display rectangle to within 0.75 px; the
iPhone's status bar shows full signal, Wi-Fi and battery and the time on
either side of the real Dynamic Island, on its centre line, with no text
under it; the watch and the iPhone stay at least 1,000 px apart. PNGs land in
`%TEMP%/rally-site-verify/` (set
`RALLY_SITE_OUT` to change); eyeball them, sliced if tall. The card
has its own: `watch-card-serve.png`, `watch-card-receive.png`,
`watch-card-answered.png` (2x crops of the hero watch),
`hero-card-1280.png`, `share-card-1280.png` (the phone's centred ball)
and `hero-card-serve-390.png` / `hero-card-receive-390.png`. Last run
2026-10-03 (release day branch): 5170 checks, all green; a mutation run
(a noindex back on one page, "Coming soon" on another, one byte of the
badge, the watch screen 3 px off, the iPhone stretched, a shadow on it,
the hint over it, the badge 36 px high, the link 8 px from it) failed
exactly the matching checks.

Why the protocol and not `msedge --screenshot`: headless Edge lays out
no narrower than ~476 CSS px from the command line (kit `lessons.md`),
so a 390 capture would show a cropped 476-wide layout; the protocol's
device-metrics emulation renders the true width and can compute
styles, which `--dump-dom` cannot.

The harness covers the LANDING. `node scripts/verify-board.mjs`
covers the BOARD: the same headless Edge, with the DevTools Fetch
domain answering the page's two edge-function POSTs from a scripted
relay and a scripted WebSocket standing in for the Realtime channel,
so every state is reached deterministically and read back out of the
DOM: the channel join and a pushed score with no ask, the 10 s safety
net and the 2.5 s fallback without WebSocket, the serve ball per team
and court side with the serving name, padel 40:40 with the receivers'
side still open (round 28: the host uploads `serveCourtSide` none, the
link centres the ball on the serving tile, with the serving name beside
it the pair is centred as one and no corner is guessed; without a name
the ball alone sits exactly in the middle; the answer then moves it to
its corner), colliding initials, the neutral
result, Swap sides, a saved sport by name, a best of nine below the
tiles at 390, 844 and 1280, the ended snapshot as the result (the
channel closes, asking stops), the mid-match stop, a discarded end,
German, the entry form's answers and the X back to it, no request
beyond the relay, no console error, no overflow, and (second pass and
review) the X top right, 1 v 1 full names on the host's word, the
wide sizes, a server name not in the lists showing nothing, landscape
fits, and (release day) no robots meta and the footer link to the App
Store listing in a new tab, in English and German. Last run 2026-10-03
(release day branch): 258 checks, all green. Run it after ANY change to
`board/index.html`. A live smoke test against the deployed relay
(share-create, share-uplink with an ended snapshot, share-end, the
page on a local static server) is still worth one run before a
release; it costs one `share-create` against the per-IP limit (30
per 10 minutes since round 30, 10 before).
