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
index.html           landing: hero with a working watch scorer (engine-faithful
                     padel: golden point or advantage, tiebreak, serve rotation,
                     viewer-perspective serve ball), how it works, sports,
                     live sharing (the phone Board runs the same match), trust,
                     free and Pro (features only, no prices)
privacy/index.html   privacy policy, EN with the DE version beneath
terms/index.html     terms of use, EN with the DE version beneath
support/index.html   contact, what the app is, common questions
impressum/index.html legal notice (address, register and VAT ID: TODO Denis)
board/index.html     live scoreboard for a share code (self-contained)
```

## How it is published

Public GitHub Pages PROJECT repo under the `denisfenger` account, Pages
served from `main` / root. Kit naming rule (playbook 3b): the public
site repo takes the bare app name, `denisfenger/rally`, so the base URL
is `https://denisfenger.github.io/rally`; the private app source is
`denisfenger/rally-app`. That base is the ONE constant `SITE_BASE` in
`src/links.ts`, and every URL the app emits derives from it:

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

- `<meta name="robots" content="noindex, nofollow">` on ALL pages until
  release. A `robots.txt` in this repo does nothing for a project page
  (crawlers read it only at the domain root, which is the user-site
  repo). The pages stay reachable by URL, which App Review requires.
  Remove the tag on every page at release.
- NO external requests: system font stack, no Google Fonts, no CDN, no
  analytics, no remote images. The only network calls on the whole
  site are the board's two edge-function POSTs to the Supabase project
  (`share-join`, `share-state`). Verify with
  `grep -rn "http[s]*://" site/` before every push: the supabase.co
  calls in `board/index.html` and `mailto:` links are the only hits
  allowed (plus the SVG namespace in `icon.svg`).
- NO pricing on the landing (the store is the source of truth).
- CONTENT CURRENCY: the landing describes features, so any round that
  adds or removes a feature updates `index.html` in the same round.
- House voice: sentence case, no emoji, no exclamation marks, no em
  dashes in visible copy.

## The board

`board/index.html` is self-contained. It joins through `share-join`
(POST `{code}`), then polls `share-state` (POST `{shareId}`) every
2.5 s with one request in flight, backs off to 10 s after three
consecutive failures, stops on `ended: true` or a 404 (the row is
deleted at match end) and shows the ended banner, pauses while the tab
is hidden and resumes on `visibilitychange`. The dot turns amber after
20 s without a successful poll. No `?code=` shows the entry form, which
submits to `board/?code=` so the URL shape the app shares stays stable.

Why polling and not Realtime: the earlier board pulled `supabase-js`
from a CDN for the broadcast channel, which the no-external-requests
rule forbids; `share-state` exists for the watch follow poll anyway and
carries no per-IP rate limit by design. The page embedded in
`supabase/functions/board/index.ts` is the legacy Realtime twin that
`*.supabase.co` serves as text/plain; this static page supersedes it,
and the edge function can be removed at the next backend pass.

## Verifying before a push

`node scripts/verify-site.mjs` drives the landing in headless Edge
over the DevTools protocol (no dependencies) and exits 1 on any
failure: it plays a scripted three-set padel match (golden point and
advantage) against an independent reference model and asserts the
serve ball's tile, edge and court side after every point on both
mockups (the rule: our serve at the near/bottom edge below the number
on the serve court side, right when the game's point total is even and
left when odd; their serve at the far/top edge above the number with
the sides mirrored); checks that no sport card is selected on load;
lists every element with a click handler or interactive role and
requires a computed `cursor: pointer` plus a visible hover and pressed
state under real mouse events; renders 390, 768 and 1280 full-page
PNGs with overflow probes; asserts no external request, no console
error and no running animation under `prefers-reduced-motion:
reduce`. PNGs land in `%TEMP%/rally-site-verify/` (set
`RALLY_SITE_OUT` to change); eyeball them, sliced if tall.

Why the protocol and not `msedge --screenshot`: headless Edge lays out
no narrower than ~476 CSS px from the command line (kit `lessons.md`),
so a 390 capture would show a cropped 476-wide layout; the protocol's
device-metrics emulation renders the true width and can compute
styles, which `--dump-dom` cannot.
