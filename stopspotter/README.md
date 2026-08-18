# StopSpotter — website

> **Renamed from StopScout.** The name change is not confined to this folder —
> see *Rename checklist* at the end before treating it as done.
>
> **Refreshed 2026-07-29** (copy re-checked against the shipping app, screenshots
> dropped in, privacy policy re-verified from the source) and **redesigned
> 2026-07-30** (one type system, restrained image scale, the widget crossfade
> replaced by a Smart-Stack-style 3D flip — visual design only, no copy changes).
> Everything that changed is in [CHANGES.md](CHANGES.md); read that first.

Three static pages plus one stylesheet. No build step, no framework, no external
requests, no JavaScript. System fonts only.

```
index.html            Marketing page
privacy.html          Privacy policy   — required App Store Connect field
support.html          Support          — required App Store Connect field
stopspotter.css       Shared stylesheet
CHANGES.md            What each pass changed, and why — newest first
icon-tile.png         192px app-icon tile — nav and hero lockup
favicon.png           32px
icon-512.png          512px, also the Open Graph image
apple-touch-icon.png  180px
img/                  Screenshots and widget captures (see CHANGES.md)
```

### Deploying

Drop the folder into the repo alongside `streetproof/` and `fitferry/` and push.
Cloudflare Pages builds on commit; a static folder needs no configuration
change, no build command, and no cache purge. The folder name must be exactly
`stopspotter` — the canonical and Open Graph URLs assume
`https://reforged.studio/stopspotter/`.

Internal links are deliberately extensionless (`href="privacy"`, `href="./"`).
Pages 308-redirects `/privacy.html` to `/privacy`, so linking with the extension
would add a redirect hop to every click, and internal redirects are a known
cause of Search Console reporting URLs as not indexed. Canonical tags match the
extensionless form.

One consequence: opening `index.html` straight off disk means the nav links
won't resolve, because `file://` has no directory-index behaviour. To preview
locally, serve the folder:

```
cd stopspotter-site && python3 -m http.server 8000
# then open http://localhost:8000/
```

You'll also want to add StopSpotter to the app list on the main `reforged.studio`
index page — nothing here does that for you.

---

## Before publishing — in order

### 1. Remove the draft watermark

Two edits per page:

1. Delete `class="draft"` from `<body>`.
2. Delete the `<meta name="robots" content="noindex">` line above it.

Then delete the block in `stopspotter.css` marked
`DRAFT WATERMARK — DELETE BEFORE PUBLISHING`. Nothing else references it.

The watermark is a fixed banner at the bottom of the viewport only — there is no
diagonal tile over the content, so screenshots and reviews stay readable. In
print the banner is hidden and `DRAFT — NOT FOR PUBLICATION` is stamped at the
top of the legal pages instead.

The `noindex` exists so that pushing this folder to test the deploy can't get a
DRAFT page full of placeholder screenshots into the index.

### 2. ~~Resolve the location question~~ — SETTLED 2026-07-29

The widget's nearest-stop lookup needs no authorization beyond When In Use plus
the separate widget-location grant iOS asks for when any StopSpotter widget is
added. The coordinate is resolved on the device against the transit database the
app carries, and is never transmitted. `privacy.html` → **Location** now also
discloses the one thing the old text got wrong: the most recent coordinate *is*
kept on the device (24-hour expiry, deleted on revoke or on a drop to approximate
location) so the widget can still name a stop without a fresh fix.

The App Store questionnaire answer that follows: precise location, used for app
functionality, **not linked to identity and not used for tracking**.

### 3. ~~Drop in screenshots~~ — DONE 2026-07-29

All four placeholders are replaced with real captures, in `img/`. See CHANGES.md
for what each one shows and how they were staged. The free-widget section and the
StopSpotter+ section now use real widget renders rather than CSS mocks.

One capture was not obtained: `img/nearby.png`. The Nearest module never finished
loading in the simulator, so the third phone slot shows the saved-stops list
instead. If the nearby list is also slow on a real device, that is a bug worth
chasing rather than a screenshot to retake.

Still optional, still better than what is there: a photograph or capture of a real
Home Screen with a StopSpotter stack mid-swipe, which would replace the `.hs`
block in the hero — the last CSS mock on the site.

### 4. Turn on the App Store link

Both CTAs are real buttons with `href="#"` and a `data-appstore` hook. Replace
the `href` with the App Store URL. Apple requires their supplied badge artwork
for the "Download on the App Store" lockup, so at the same time drop the
official SVG inside the `<a>` in place of the text label.

---

## Things deliberately built in

**Independence appears twice, by design.** The full §9.3f sentence is the
canonical statement and sits in the footer of every page. The hero deck carries
a short-form echo. Change the footer if the wording ever changes; the deck is a
summary, not a duplicate.

**Mode colour appears on badges only** (spec A16.6). No tinted sections,
buttons, links or backgrounds. Every badge contains a route number or the word
PATH, so colour is never the only thing carrying meaning — the four fills share
a lightness register and collapse in greyscale by construction (A16.7). Any new
mode-coloured element needs a word or glyph doing the real work.

`fill` is the badge background in **both** appearances with `--badge-ink`
(`#1A1512`) on top. The `deep` tokens are declared in `stopspotter.css` but
unused, because nothing here renders mode colour as text or as an unfilled icon.
If that changes, `--mode-lrail-deep` (`#846538`) must not go below 15px.

**No pure white anywhere.** The light set is built on `#FAF7F4`, the app's own
light surface — the value the mode spec cites when it says a sage badge with dark
text works on `#0E0B09` and `#FAF7F4` alike. Cards sit just above it at
`#FDF9F4`, a warm off-white rather than `#FFFFFF`, with recessed bands at
`#F0E9E1`. The only `#fff` left in the stylesheet is inside the print block,
where paper is white. All light-mode text clears WCAG AA: ink 15.8:1, secondary
8.3:1, tertiary 5.5:1, accent 5.4:1, and the provenance colours 4.5:1+.

**One thing to decide about badges in light mode.** Mode fills are the same in
both appearances by design (A16.2), so against the light card they measure only
1.6–1.8:1 as *shapes* — the badge reads as tinted paper rather than as a chip.
The text inside is unaffected at 9.5–10.7:1, and this is inherent to the spec's
shared-lightness register rather than anything the site introduced; the app has
the same property. If you want the badges more defined on light, the least
invasive fix is a hairline border in the light block only:

```css
@media (prefers-color-scheme:light){
  .wg-badge.m-bus,.wg-badge.m-rail,.wg-badge.m-lrail,.wg-badge.m-path{
    border-color:rgba(34,28,23,.14)}
}
```

That deviates from how the app renders them, which is why it isn't on by
default.

**Light and dark both follow the OS.** Dark is the default and the light set
lives in the one `prefers-color-scheme: light` block near the top of
`stopspotter.css`. If you ever want to force dark, comment that block out and
change `color-scheme` to `dark` plus drop the second `theme-color` tag in each
page's `<head>`. Print is separate and always renders black on white.

### Troubleshooting: icons not appearing

If the nav icon, hero icon and browser-tab icon are all missing at once, the
cause is almost always that the PNGs never reached the server rather than
anything in the CSS. Check by opening the file directly:

```
https://reforged.studio/stopspotter/icon-tile.png
```

A 404 means the binaries aren't in the repo. The usual culprit is a `.gitignore`
entry like `*.png` or `img/` somewhere up the tree — Cloudflare Pages only serves
what's committed. `git check-ignore -v stopspotter/icon-tile.png` names the rule
and the file it came from. `git add -f` gets past it once; fixing the ignore rule
is better.

The icons are `<img>` elements rather than CSS backgrounds precisely so this
fails loudly: a missing file shows a broken-image marker instead of empty space.
The 32px favicon is additionally inlined as a base64 data URI in each page's
`<head>`, so the tab icon works even if no image file deploys at all.

---

**The icon ships as a tile** (`icon-tile.png`) — the peach glyph on the app's
dark surface with rounded corners, which is how the icon actually appears on a
Home Screen. An earlier version filled the glyph's alpha channel with
`var(--accent)`, which broke in light mode: the accent becomes `#965535` there,
turning the glyph muddy brown on cream. A tile has its own background, so it
needs no accent token and sits correctly on any page colour. `icon-tile.png`
(192px) serves the nav and hero lockup; `icon-512.png` doubles as the Open Graph
image.

**Legal pages are printable.** The print block flattens to black on white,
expands every collapsed FAQ answer, prints mailto targets in parentheses, hides
navigation and widget mocks, and stamps `DRAFT — NOT FOR PUBLICATION` at the top
while the draft class is present.

**Accessibility.** Skip link, one `<h1>` per page, semantic landmarks, labelled
nav regions, `aria-current` on the active item, visible focus rings,
`prefers-reduced-motion` respected, native `<details>` for the FAQ so it's
keyboard-operable without script. Body text and all secondary inks clear WCAG AA
against their surfaces — note that the spec's `7C716A` / `8A7F78` greys measured
3.96:1 and 3.63:1 at these sizes, so the site uses `9C8F86` and `6F625A`
instead. Worth deciding whether the app tokens should follow.

---

## Still open

- **Price** appears as `$2.99` in four places: the hero note, the free/paid
  divider, the StopSpotter+ cost cell, and the support FAQ. If it changes, grep
  `2.99`.
- ~~**§9.5 of the app spec** names the purchase row `Support StopSpotter`~~ —
  checked 2026-07-29 and no longer true: the shipping Settings row is
  `StopSpotter+`, and the app never uses tip-jar language. App and site agree.
- **Terminating arrivals.** The support page now has an entry for it ("A station
  is showing no departures at all"). The app side is still open: if an empty board
  at a terminus is correct behaviour, it wants an empty state saying so rather
  than looking broken.
- **Widget stacks at scale.** The copy says "as many as you like" and states the
  refresh-budget tradeoff. It deliberately names no number, since WidgetKit
  budgets refreshes per app rather than per widget and nobody has watched ten
  for a day.
- **Worth a lawyer, not a guess.** The exact attribution wording the NJ Transit
  licence requires on a public site; whether the caching proxy — which has now
  shipped and is the only path to live data — triggers New Jersey or CCPA
  obligations once traffic scales; and whether "Data Not Collected" on Apple's
  questionnaire holds for a bundled-database location lookup. *(The
  community-hosted PATH feed that used to be on this list is gone: PATH now comes
  from the Port Authority's own published data, through the same proxy.)*
- **A 404 page.** Pages walks up the directory tree looking for `404.html`, so
  `/stopspotter/404.html` would catch bad StopSpotter URLs specifically. Not
  required, and the root site may already cover it.

## ~~When the caching proxy ships~~ — IT SHIPPED, and the policy is rewritten

`privacy.html` → **Network requests** now describes the real architecture: one
address (`stopspotter-api.reforged.studio`), operated by Reforged Studio, which
the device talks to instead of the agencies; a request carrying a stop identifier
and nothing else; the IP that any web request carries, acknowledged rather than
glossed; and the agencies seeing the server rather than you. The sentence saying
requests pass through no Reforged Studio server — the one line that became false
— is gone. **Schedule data** gained the once-a-day, Wi-Fi-only check for a newer
copy of the database, which the old text did not mention.

**The one thing left open, and it is a Cloudflare setting rather than a copy
question.** The page currently says the server keeps no record tying a request to
a person or device, and that Cloudflare "may hold short-lived operational logs of
its own" — which is true and deliberately unspecific, because retention is a
property of the hosting account. The old advice here still stands: retain nothing
and say so. A stated zero-retention position is worth more than the debugging
convenience, and it would let that clause be replaced with something definite.

---

## Rename checklist (StopScout → StopSpotter)

The website is fully renamed. These live elsewhere and are not:

- ~~**App spec verbatim strings.**~~ Done, and checked on 2026-07-29: the app
  composes the independence disclaimer and both attributions from one constant
  that already reads `StopSpotter` and `NJ Transit`. The site was still setting
  the agency in caps; it now matches the app word for word. See CHANGES.md →
  *One change that touches every page* if you'd rather go back to `NJ TRANSIT`.
- **The NJ TRANSIT developer application**, if it names the app. Worth checking
  whether an approved application is tied to the submitted name.
- **App Store Connect**: bundle ID, app record, and the Support URL / Privacy
  Policy URL fields, which must point at `/stopspotter/…`.
- **The old path.** `/stopscout/` was pushed live at least briefly. It carried
  `noindex`, so search exposure should be nil, but if you want any inbound link
  to survive, add a `_redirects` file at the repo root:
  ```
  /stopscout/*  /stopspotter/:splat  301
  ```
  Otherwise simply delete the old folder.
- **Verify the new name is clear** on App Store Connect and as a trademark
  before committing further — the reason for the change applies equally to its
  replacement.
