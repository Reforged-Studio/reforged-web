# StopScout — website

Three static pages plus one stylesheet. No build step, no framework, no external
requests. System fonts only. Drop the folder on Cloudflare Pages (or any static
host) and it works.

```
index.html            Marketing page
privacy.html          Privacy policy   — required App Store Connect field
support.html          Support          — required App Store Connect field
stopscout.css         Shared stylesheet
appicon.png           Wordmark glyph, used as a CSS mask
favicon.png           32px
icon-512.png          512px, also the Open Graph image
apple-touch-icon.png  180px
```

Clean URLs work without configuration on Cloudflare Pages: `/stopscout/privacy`
resolves to `privacy.html`. Canonical tags already assume
`https://reforged.studio/stopscout/`.

---

## Before publishing — in order

### 1. Remove the draft watermark
Delete `class="draft"` from `<body>` in all three pages, then delete the block
in `stopscout.css` marked `DRAFT WATERMARK — DELETE BEFORE PUBLISHING`. Nothing
else references it.

### 2. Resolve the location question
`privacy.html` → **Location** currently says location is used to list nearby
stops *and to point the nearest-stop widget at the right stop*. That second use
happens outside the app, on iOS's refresh schedule. If it needs anything beyond
When In Use, this paragraph is wrong and so is the App Store privacy
questionnaire answer. This is a free feature every user gets, so it is not an
edge case. **Settle this before the app or the site ships.**

### 3. Drop in screenshots
Four placeholders, each marked with an HTML comment giving the intended
filename. Export at 2× or 3×, dark appearance, PNG, no device frames — the
frames and corner radii are drawn in CSS, so a bare screen composites better.

| Replace | File | Aspect | Notes |
|---|---|---|---|
| `.shot.wide` in *One place* | `img/search-modes.png` | 16:10 | One query returning a bus stop, a rail station and a PATH station together. The only proof of the coverage claim — worth staging. |
| `.shot.phone` #1 | `img/stop-detail.png` | 9:19.5 | Mixed provenance, so per-row `Live` / `Scheduled` words are visible on the secondary line. |
| `.shot.phone` #2 | `img/search.png` | 9:19.5 | Modes intermixed. |
| `.shot.phone` #3 | `img/nearby.png` | 9:19.5 | |

Swap each `<div class="shot …">…</div>` for:

```html
<img src="img/stop-detail.png" alt="StopScout stop detail, showing live and
     scheduled departures" width="1290" height="2796" loading="lazy"
     class="shot phone">
```

Optional but better than the CSS mocks: a real Home Screen with a StopScout
stack, mid-swipe. It would replace the layered `.stack` in the StopScout+
section and the `.hs` block in the hero.

### 4. Turn on the App Store link
Both CTAs use a non-functional placeholder badge. `index.html` carries a
commented-out live version next to the first one. Apple requires their supplied
badge artwork rather than a CSS approximation, so download the official SVG and
substitute it inside the `<a>`.

---

## Things deliberately built in

**Independence appears twice, by design.** The full §9.3f sentence is the
canonical statement and lives in the footer of every page. The hero deck carries
a short-form echo. Change the footer if the wording ever changes; the deck is a
summary, not a duplicate.

**Mode colour appears on badges only** (spec A16.6). No tinted sections,
buttons, links or backgrounds. Every badge contains a route number or the word
PATH, so colour is never the only thing carrying meaning — the four fills share
a lightness register and collapse in greyscale by construction (A16.7). If you
add a mode-coloured element anywhere, it needs a word or a glyph doing the real
work.

`fill` is the badge background in **both** appearances with `--badge-ink`
(`#1A1512`) on top. The `deep` tokens are declared in `stopscout.css` but
unused, because nothing here renders mode colour as text or as an unfilled icon.
If that changes, `--mode-lrail-deep` (`#846538`) must not go below 15px.

**The icon is a CSS mask, not an image.** The source PNG is a flat `#FFB187`
glyph on transparency, so `appicon.png` ships as an alpha mask filled with
`var(--accent)`. One asset renders correctly in both appearances and tracks the
accent token automatically — the light icon variant isn't needed for the web.
Worth replacing with an SVG eventually: it's simple geometry, it would be a
fraction of the size, and it would stay crisp at any density.

**Legal pages are printable.** The print block flattens to black on white,
expands every collapsed FAQ answer, prints mailto targets in parentheses, hides
navigation and widget mocks, and stamps `DRAFT — NOT FOR PUBLICATION` at the top
while the draft class is present.

**Accessibility.** Skip link, one `<h1>` per page, semantic landmarks, labelled
nav regions, `aria-current` on the active nav item, visible focus rings,
`prefers-reduced-motion` respected, native `<details>` for the FAQ so it's
keyboard-operable without script. There is no JavaScript on any page. Body text
and all secondary inks clear WCAG AA against their surfaces — note that the
spec's `7C716A` / `8A7F78` greys measured 3.96:1 and 3.63:1 at these sizes, so
the site uses `9C8F86` and `6F625A` instead. Worth deciding whether the app
tokens should follow.

---

## Still open

- **Price** is stated as `$2.99 · one time · not a subscription` in four places:
  hero note, divider, the StopScout+ cost cell, and the support FAQ. If it
  changes, grep `2.99`.
- **§9.5 of the app spec** hides the purchase row until purchasable and names it
  `Support StopScout`, which reads as a tip jar. It's a feature unlock now — the
  app and the site should use the same word for it.
- **Terminating arrivals.** If an empty board at a terminus is correct
  behaviour, the app wants an empty state saying so rather than looking broken,
  and the support page probably wants an entry for it.
- **Widget stacks at scale.** The copy says "as many as you like" and states the
  refresh-budget tradeoff. It deliberately does not name a number, since
  WidgetKit budgets refreshes per app rather than per widget and nobody has
  watched ten for a day.
- **Worth a lawyer, not a guess.** The exact attribution wording the NJ TRANSIT
  licence requires on a public site; the community-hosted PATH feed the app
  depends on with no agreement in place; whether a future caching proxy triggers
  New Jersey or CCPA obligations once traffic scales; and whether "Data Not
  Collected" on Apple's questionnaire holds for a bundled-database location
  lookup.

## When the caching proxy ships

`privacy.html` → **Network requests** is written to be replaced. Nothing
elsewhere in the policy depends on its claims — no "as described above", no
global "nothing ever leaves your device". Amend that section and revise the date
above it; no audit of the rest is needed.

It will need to state: that requests now reach a proxy operated by Reforged
Studio before the agency feeds; what the proxy receives (a stop identifier and
an IP address); that it caches agency responses rather than requests; whether
request logs exist and for how long they are retained — advice: retain nothing
and say so, a stated zero-retention policy being worth more than the debugging
convenience; that no identifier for you or your device is stored; and that the
agency feeds now see the proxy's IP rather than yours. It must also drop the
sentence saying requests pass through no Reforged Studio server, which is the
only line in the document that becomes false.
