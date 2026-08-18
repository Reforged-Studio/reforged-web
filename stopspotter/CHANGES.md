# CHANGES.md — StopSpotter site

Newest pass first. This folder is not a git repository, which is why each pass takes a backup
before it edits anything.

| Pass | What it was |
|---|---|
| [30 July 2026 — v4](#30-july-2026--v4-the-website-fix-round) | Hero stops advertising a paid feature; the status cards drop to two; the walkthrough video is cut. |
| [30 July 2026 — v3](#30-july-2026--v3-one-narrative-real-captures-a-walkthrough) | The tour restructured into one narrative, real captures throughout, a walkthrough video. |
| [30 July 2026 — design](#30-july-2026--the-design-pass) | Visual design only. Typography, image scale, the widget flip, spacing. |
| [29 July 2026 — copy](#29-july-2026--the-copy-refresh) | Copy against what the app actually does, plus real screenshots. |

---

# 30 July 2026 — v4: the website fix round

Three owner reactions to v3, each mandatory. **Copy, structure and palette were frozen going in
except where a reaction named them**; nothing outside the hero image, the status-card list and
the walkthrough slot was rewritten, moved or restyled.

Backup of the folder as it stood before this pass — it is the only surviving copy of the video:

```
~/Library/Mobile Documents/com~apple~CloudDocs/Downloads/stopspotter-backup-2026-07-30-152154/
```

## 1 · The hero was advertising a paid feature

**The defect.** v3's hero cropped `69-inch/dark/01-hero.png`, whose medium widget is pointed at
the seeded combo **"Morning to Midtown"**. A nicknamed saved combo is StopSpotter+. The hero
sits *outside* the `#plus` section, three screens above the paywall — so the first thing the
page showed as "the widget" was the thing you have to buy, directly above fine print promising
"The free app puts your nearest stop on the Home Screen."

**The fix.** `img/widget-hero.png` is re-cropped from a fresh capture of the FREE default: an
unconfigured widget resolving the nearest stop, headed by the **station name** rather than a
nickname —

> **Washington St at 12th St**  ·  160 ft  ·  Hoboken
> **5** min · 126 → Hoboken · then 13, 21, 28 min · *updated 1 min ago*

The nicknamed renders (`widget-bus` / `widget-path` / `widget-mixed`) are untouched and stay
where they were, inside the StopSpotter+ section — that is the product being sold there.

**How it was captured.** The repo's SHIP-A tooling, on a scratch output root so the repo's own
`screenshots/` never changed (the tree is clean on `main` and this pass is read-only for git):
`StopSpotter` built Debug, `SS-Shots-69` (iPhone 17 Pro Max, iOS 26.2) erased and booted, the
four combos seeded straight into the App Group, location pinned to Washington & 12th, status bar
overridden to 9:41, then
`StopSpotterUITests/AppStoreScreenshotTests/testCaptureHomeScreenHero`.

**Why the existing staging file could not simply be cropped.** The shipped
`01-hero-nearest-default.png` has both widgets placed but the MEDIUM one still drawing its
**redacted grey-bar placeholder** — no timeline had arrived. The test's wait loop is
`while springboard.staticTexts["… CONTAINS min"].count == 0`, which the *small* widget satisfies
on its own, so the shot fires while the medium is still blank. The re-run reproduced this
exactly: the freshly harvested `shot-01-hero-nearest-default.png` was redacted the same way.

What made the pass work is that the run then **skipped** — `could-not-find-the Edit Widget menu
item` — which meant it never got as far as configuring the widgets. The device was left sitting
on a Home Screen with both widgets on the free default, so a settled capture was taken directly
with `simctl io screenshot` about a minute later, once the medium had rendered. Sampled five
times, twelve seconds apart, and the first frame taken.

**The crop is the v3 crop.** The widget card's bounding box was measured in both the v3 source
and the new capture and is byte-identical in position — `(96,282)–(1223,811)`, 1128 × 530 —
so the new asset is the same box scaled to the same on-disk 1035 × 487.

**One thing changed in the encode.** Pillow's median-cut quantiser turned the orange refresh
glyph **grey**: the glyph is a few hundred pixels in half a megapixel, and with the footer text
now grey rather than orange there is no longer enough orange in the frame to earn a palette
entry. Caught by eye, then confirmed by sampling the glyph box. ImageMagick's quantiser keeps
it, so the asset is `magick … -colors 192 PNG8:` — **45.9 KB, down from 113 KB**, in line with
the other widget crops.

## 2 · The status cards: three became two

The middle card is gone. The owner-approved intro paragraph above the list is **unchanged, word
for word**, as is the closing note below it.

| | v3 | v4 |
|---|---|---|
| 1 | **Fresh** — `updated 1 min ago` | **Live** — `updated 1 min ago`, plus one sentence |
| 2 | **Gone quiet** — `updated 18 min ago — may not reflect real-time data` | *dropped* |
| 3 | **Timetable** — `Schedule data` | **Timetable** — `Schedule data`, unchanged |

The added sentence, which absorbs the dropped card's tap without showcasing staleness:

> A live prediction from the agency, stamped with the moment it arrived. **If that stamp grows
> older, a tap of the refresh button fetches the latest.**

**License lint.** The mandated string *"may not reflect real-time data"* leaves `index.html`
with the middle card. That is fine for marketing copy — the mandated placement is in-app, and
the phrase also still appears on **`support.html`**, in the widget-refresh answer. The new
sentence claims only that the tap **fetches**; it does not say what comes back, and nothing on
the page now claims accuracy or timeliness. Swept for `always up to date` / `always accurate` /
`real-time accuracy`: zero hits. The only match for `guaranteed` is the footer's existing
*"Data availability is not guaranteed"*, which is a disclaimer, not a claim.

`.c-str.stale` — and the `--stale` colour token it reads — are now unused; no element on the
site carries the class. **Left in place deliberately**: the palette and CSS are frozen except
where a reaction named them, and one dead selector plus one dead token is cheaper than a change
nobody asked for. Worth knowing if the stale treatment is ever wanted back.

## 3 · The walkthrough video is cut

Owner verdict: home screen dead ends, spinners, glitchy, hard to follow. Removed:

| Gone | |
|---|---|
| `video/walkthrough.mp4` | 4.2 MB, 330 × 718, H.264, 38 s |
| `video/walkthrough-poster.jpg` | the poster frame |
| `<figure class="walk">` | the video + its figcaption |
| `.walk` / `.walk video` / the reduced-motion `.walk` rule | in `stopspotter.css` |
| the page's only `<script>` | the reduced-motion shim |

The `video/` directory is deleted; the backup above is the only copy. **The page now ships zero
`<script>` elements** — that shim existed only because CSS cannot pause a playing video. The
remaining motion (the widget stack's roll and its page dots) is CSS and is already suppressed
under `prefers-reduced-motion` in the stylesheet, so nothing regressed by removing it.

**The section's layout without the video slot.** `.tour-open` now holds exactly its eyebrow,
headline and lead paragraph, and the five `.step` rows carry the section alone. Verified: the
section's last child is the lead `<p class="body">` at every width — **no empty frame and no
orphaned caption**. The walkthrough was the page's only `<figure>`, so `index.html` now contains
none at all; there is nothing left that could render as a caption without its picture. The
`.tour-open + .step { padding-top: var(--sp-md) }` rule (written for v3, to tuck
the first step under the video) is what keeps step 01 close to the lead now that the video is
gone, rather than a full section apart; it needed no change.

### If a better capture is ever wanted

What was removed and why is above. The v3 notes record how the original was made — one
`simctl io recordVideo` pass with the app relaunched through its DEBUG routes between beats,
exported with AVFoundation because `ffmpeg` is not installed here. The complaints to fix are the
dead ends and the spinners, which are pacing and staging problems in the recording, not encoding
ones. The 4.2 MB / 330 × 718 trade-off is documented in the v3 entry.

## 4 · Not done, deliberately

- **"That's where the walkthrough ends and the Home Screen begins."** (section 06's opening) is
  unchanged. It reads as a reference to the five step rows, which *are* the walkthrough, so it
  still parses — but it was written while a video sat above it. Flagged for the owner, not
  rewritten: copy is frozen and no reaction named this sentence.
- **The free-widget section still shows `widget-nearest.png`** (Journal Square, PATH). The hero
  and section 06 now both show a nearest-stop widget, but at different stops, in different modes
  and at different scales, and they make different points. No reaction asked for this to change.
- **The draft watermark, `noindex`, and `href="#"` on both App Store buttons** are all still in
  place. Owner's call at publish time.

## 5 · Verified in-browser

Served over HTTP, driven through CDP with `Emulation.setDeviceMetricsOverride` at
**320 / 375 / 768 / 1280** (headless Chrome clamps its own window to 500px on macOS — the v3
trap, avoided the same way).

| Check | Result |
|---|---|
| Horizontal overflow | `scrollWidth === clientWidth` at all four; **zero** elements past the right edge |
| Status cards | exactly two: `Live / updated 1 min ago` and `Timetable / Schedule data`, at all four |
| Hero image | `img/widget-hero.png`, 1035 × 487, showing the free nearest-stop default, at all four |
| `<video>` elements | **0** |
| `<script>` elements | **0** |
| Empty/one-child `<figure>` | **0** — the page has no `<figure>` left at all |
| `.step` rows | **5**, alternating at 1280, single column at 768 and below |
| Images | **16/16 decode with zero failures** at all four widths; all 12 unique files answer 200 |
| `may not reflect real-time data` on `index.html` | absent (expected — still present on `support.html`) |

**A measurement trap worth recording.** The hero asset was replaced **in place, at the same
filename**, and the first post-swap verification run passed while still rendering *"Morning to
Midtown"* from the warm cache — the check sailed past the exact thing it existed to catch. Every
run since sets `Network.setCacheDisabled`. A second, related trap: with the cache disabled, a
probe taken shortly after navigation reports every `loading="lazy"` shot as broken, because the
walk down the page only *starts* those fetches. "Broken" there means "slow". The honest check is
to force `loading = 'eager'` and `await img.decode()` on each — which is what the 16/16 above is.

Folder is now **1.0 MB** (was 5.3 MB; the video was 4.3 MB of it).

## 6 · The repo was not touched

`git status` in `~/XcodeProjects/StopSpotter` is clean and `HEAD` is still `main`, before and
after. The build, the derived data, the `.xcresult`, the harvested attachments and the poller
frames all went to the session scratchpad, never to `screenshots/appstore/`, whose files are
tracked. No commit, no push, no publish.

**Carried forward for the repo** (not fixed here — this pass is read-only for git): the SHIP-A
hero test's content-wait loop cannot tell the two widgets apart, so `01-hero-nearest-default`
ships a redacted medium widget whenever the small one renders first. Both the committed capture
and this session's re-run show it. A per-widget wait, or simply a longer settle after the
`min` match, would fix it.

---

# 30 July 2026 — v3: one narrative, real captures, a walkthrough

**The v2 design pass is the base. Palette and design tokens are frozen and untouched.** What
changed is structure, the images, and three cards' wording — each named by the brief.

Backup of the folder as it stood before this pass:

```
~/Library/Mobile Documents/com~apple~CloudDocs/Downloads/stopspotter-backup-2026-07-30-140206/
```

## 1 · "Inside the app" + "saving a stop" are now ONE narrative

The owner's read was that the two sections were jumbled, repetitive and out of order, and that
was right: "Look up any stop. Free." showed three phones in a row with no line connecting them,
then a *separate* section re-explained finding and saving a stop in two cells — saying the same
thing twice, in the wrong order, with the screenshots nowhere near the sentences they proved.

They are replaced by one run of five `.step` rows in the order a rider actually meets the app:

1. **Discover** — one search reaches every mode *(search screen)*
2. **Or don't search at all** — stops near you, no typing *(the Nearest list, with the map)*
3. **Departures** — gates at Port Authority *(the PABT board)*
4. **Save it** — your routes, your direction, your name for it *(the route/destination sheet)*
5. **Saved** — then you never search again *(the Saved tab)*

The free-widget section that follows is now numbered **06** and opens "And then you don't open
the app at all", so the narrative runs through it rather than restarting.

Each row is text beside the visual that proves it, alternating sides — the rhythm from
`reforged.studio/streetproof`, whose `.pfeat` / `.pfeat.flip` two-column grid was read directly
off the live page. Only the layout idea is borrowed; every size and colour is StopSpotter's own
token. Rows collapse to a single column at 860px.

**A trap worth recording:** the reversing modifier is `.rev`, **not** `.flip`. `.flip` is already
the widget stack's 3D cylinder (`width` + `aspect-ratio` + `perspective`), so the first draft's
`.step.flip` inherited a 345px fixed width and collapsed the copy column to **0px**. Caught in
the 1280px verification pass, not by eye.

## 2 · Every image is now a real app render

Sourced from the SHIP-A App Store set that landed on `main` this morning
(`screenshots/appstore/`), captured from the running app on real NJ Transit and PATH data:

| On the page | Source |
|---|---|
| `img/search.png` | `69-inch/dark/03-search.png` |
| `img/nearest.jpg` | `69-inch/**light**/04-nearest.png` |
| `img/stop-detail.png` | `69-inch/dark/02-departures-pabt.png` |
| `img/filter.png` | `69-inch/dark/06-filter.png` |
| `img/saved.png` | `69-inch/dark/05-saved-combos.png` |
| `img/widget-hero.png` | cropped out of `69-inch/dark/01-hero.png` |

**The hero's widget was a CSS reconstruction** — hand-built `<div>`s imitating a widget. It is
now the real widget, cropped straight out of a Home Screen capture ("Morning to Midtown", the
119 and 126 to New York, footer reading *Scheduled · updated 3 min ago*).

**Nearest uses the LIGHT capture, deliberately.** The dark `04-nearest` in the SHIP-A set is not
a usable shot — it is the Search screen under an *"Open in StopSpotter?"* system dialog, with the
Nearest module never revealed. The set's own README recommends light for this shot on legibility
grounds; that recommendation understates it. **Flagged for the repo, not fixed here** (this pass
is read-only for git). Light also happens to be the right call: the dark Apple Maps style renders
Hoboken as near-black.

JPEG for the map shot (photographic), palette-quantised PNG for the flat UI. Total folder 5.3 MB.

## 3 · The walkthrough video

A real screen recording of the app, in the merged section, muted + looping + inline. Every frame
is the running app on a dedicated simulator — search, the Port Authority board, choosing routes
and a direction at Willow Ave at 15th St, and the Saved tab with three saved stops, one showing
**Gate 213-1**.

Segmented capture stitched at cuts, which the brief allows: one continuous `simctl io recordVideo`
pass with the app relaunched through its DEBUG routes between beats, so the cuts are real app
launches rather than an edit. Driving the save flow by synthesized taps was not attempted — a
wedged gesture on this simulator does not fail fast (a `swipeLeft()` was measured at 1664 s during
SHIP-A), and a wedge would have cost the rest of the pass.

Exported with AVFoundation, since `ffmpeg` is not installed here: **330×718, H.264, 38 s, 4.2 MB**,
plus a poster frame. The size is a deliberate trade — the same export at 450×978 is 6.2 MB, which
is too heavy for a landing page, and the medium-quality preset silently caps output at 220×480,
which is too soft. Displayed at 248px, phone-sized rather than as a banner.

`prefers-reduced-motion` shows **the poster only**: a small script drops `autoplay`, clears
`loop`, pauses and rewinds. Verified — the video ends paused at `t=0` with the poster showing.

## 4 · The widget stack now shows its page dots

**Fallback path taken, and here is why.** A real capture would mean driving SpringBoard to drag
one placed widget onto another to form a stack — the flakiest interaction in this project's
harness, on a simulator whose input stack is a standing landmine and whose wedges do not fail
fast. The brief permits the fallback, and it was the right call for a session already carrying
two merges.

So: the CSS cylinder stays, and gains the **three side page-dots** a real Home-Screen stack draws
on its right edge, outside the card where iOS puts them, cycling in step with the roll on the same
`cubic-bezier(.62,.03,.3,1)` the turn already used. Under reduced motion the roll, the dots and
all motion are replaced by the three widgets at rest.

## 5 · The live / scheduled cards now match shipped behaviour

**The owner-approved intro paragraph is unchanged, word for word.** Only the three cards moved.

| Card | Reads |
|---|---|
| Fresh | `updated 1 min ago` — a live prediction, stamped when it arrived |
| Gone quiet | `updated 18 min ago — may not reflect real-time data` |
| Timetable | `Schedule data` |

The stale string is the mandated wording and is reproduced **verbatim**. The third card was the
inaccurate one: it was headed *"Feed unavailable"*, which is only half of when `Schedule data`
appears. It also appears when nothing live runs for that stop and hour at all. The card now covers
both reasons and says plainly that the label does not separate them — *"The label doesn't separate
the two, so neither do we: it means this is the timetable, not a prediction."* No distinction is
invented that the app does not draw.

**live/scheduled cards: final pass pending the provenance design round.**

## 6 · Verified in-browser

Served over HTTP and checked at **320 / 375 / 768 / 1280**:

- **No horizontal overflow at any width** — `scrollWidth === clientWidth`, and a sweep for any
  element extending past the viewport returned **zero** offenders at all four.
- **Steps alternate correctly at 1280** — copy 716px / media 232px, sides swapping row to row;
  single column at 768 and below.
- **All 16 images load** — zero broken, natural sizes as intended.
- **Video plays inline**, muted, looping, `playsinline`.
- **Reduced motion honoured** — poster only, dots and roll suppressed.

---

# 30 July 2026 — the design pass

**Copy, page structure and palette were frozen going in and are unchanged.** Not one word of
running copy was rewritten, no section was added, moved or removed, and no colour token was
touched. Everything below is typography, scale, layout and motion.

Backup of the folder as it stood before this pass:

```
~/Library/Mobile Documents/com~apple~CloudDocs/Downloads/stopspotter-backup-2026-07-30-1002/
```

## 1 · Typography discipline

The opening viewport had no focal point because seven different size/weight/face combinations
were competing in it — and the loudest thing on the page was the wordmark, at 48px/800, set
**larger than the headline it sat above**. That is the whole problem in one measurement.

There is now one type system, declared at the top of `stopspotter.css` and enforced through
custom properties rather than by hand at each call site.

**One text/display face** — the system sans — at exactly **five sizes**:

| Token | Size | Used for |
|---|---|---|
| `--t-display` | `clamp(32px, 4.1vw, 46px)` | the hero `h1` and the StopSpotter+ tier banner. Nothing else. |
| `--t-title` | `clamp(25px, 2.9vw, 33px)` | every `h2`, and the support/privacy page titles |
| `--t-lead` | `clamp(17px, 1.5vw, 19px)` | the one subhead under a headline; page leads; `.doc h2` |
| `--t-body` | `17px` | running copy — and `h3`, which is separated by *weight*, not size |
| `--t-small` | `14px` | fine print, captions, card copy, footer |

**One utility face** — the mono — at exactly **two sizes**, never in prose:

| Token | Size | Used for |
|---|---|---|
| `--t-label` | `11px`, uppercase, `.14em` | every eyebrow, kicker, caption, CTA note, cell label, price, stamp, jump link, draft flag |
| `--t-data` | `13px` | the transit strings the app itself prints (`.c-str`) |

**Three weights: 400, 600, 700.** The old sheet used 400, 500, 600, 650, 700 and 800 — including
`650`, a half-step the system font synthesises. All of it collapsed: `650 → 600`, `800 → 700`,
`500 → 400` or `600`.

The label sizes were the messiest part and are worth stating separately: the eleven uppercase
mono labels on the site were previously set at 9, 10, 10.5, 11, 11.5, 12, 12.5 and 14px with
five different tracking values. They are now one rule, one size, one tracking.

**The deliberate exception, so it doesn't read as a miss.** `.wg` — the CSS widget mock — and
`.wg-badge`, which the mode row borrows, keep the app's own internal sizes (9–18px). They are a
*drawing of the app*, not page typography; a mode badge shrunk to 11px to satisfy the page scale
stops reading as the control it is a picture of. This is commented in the stylesheet.

### What that did to the hero

The hero now reads in the order it was supposed to: **headline → one subhead → one visual → one
CTA**, with nothing else competing above the fold.

| Element | Was | Is |
|---|---|---|
| Wordmark lockup | 66px icon + 48px/800 wordmark — louder than the `h1` | 32px icon + 19px/700 wordmark; a signature, not a title |
| `h1` | 38px max | 46px max, and now the largest thing in the viewport |
| Deck | 19px **/650**, a second bold block | 19px/400 in `--ink2` — one quiet subhead |
| The "save the stops you use…" paragraph | 19px, four lines, sitting *between* the subhead and the button | moved below the CTA at 14px/`--ink3` as fine print |
| Device note | 13.5px, separate treatment | same 14px fine-print block |

Only the *placement and weight* of that paragraph changed — its words are untouched. The hero
markup is now a single grid with named areas (`brand / head / deck / art / cta / fine`) so that
one source order gives the visual-between-subhead-and-CTA order on a phone and the
art-in-a-right-column layout on a desktop. The two-column breakpoint moved 900px → 1000px,
which is where the left column stops being too narrow for the headline to hold two lines.

One typographic fix in the markup: `NJ&nbsp;Transit` in the hero deck, because the line was
breaking between "NJ" and "Transit". The site already sets `Home&nbsp;Screen` and
`App&nbsp;Store` the same way.

## 2 · Image restraint

| Image | Was | Is |
|---|---|---|
| Three app screenshots | three bare full-bleed slabs, 290px each, lined up as one wall | **190px each in slim device frames**, stepped 0 / 44 / 88px down the page, left-aligned to the text column, with 44px between them. About 41% of the old footprint per shot. |
| `saved-modes.png` (the wide one) | 992px — the full content width | **540px**, aligned to the text column it explains rather than centred in the page |
| `widget-nearest.png` (free widget) | floating alone at 360px with 630px of dead space beside it | inside a **Home-Screen frame** at 322px — the real capture with a row of app tiles under it, so the size on the page *is* the size on a phone |
| The StopSpotter+ widget | 360px | 345px — 1:1 with the real widget's 345pt width |

The device frame is a 5px bezel with a 26px outer / 21px inner radius. The Home-Screen frame in
the free-widget section carries **one** row of four tiles, against the hero's two rows of four,
so the two frames don't read as the same picture twice.

## 3 · The flip

The crossfade is gone. In its place is the vertical 3D roll a Smart Stack actually does.

**The geometry is a real cylinder, not a faked one.** Each of the three faces sits at radius
`--r` — exactly half the card's height — and the stage is pushed back by the same amount, so a
face at rest lands on the `z = 0` plane at its natural size with no perspective magnification.
Rotating a face `-90° → 0°` brings it up from underneath; `0° → +90°` takes it over the top. The
outgoing and incoming faces turn together, on the same cylinder, which is what makes it read as
one object turning rather than two images swapping.

- **Timing:** 12s loop, three faces offset 4s apart — **3.4s at rest, 0.6s of turn**, eased
  `cubic-bezier(.62,.03,.3,1)` on the turn intervals only.
- **Depth:** a gradient overlay on each face, dark at both edges and clear through the middle,
  fading in to 0.9 across the turn and out again — so the edge tipping away from the light
  darkens, going out over the top and coming in from below. Plus a restrained drop shadow that
  turns with the card.
- **Radius:** 24px, the Home-Screen widget radius at this card size.
- **No JavaScript.** Negative `animation-delay`s do the sequencing; `--d` is a custom property
  so the shading overlay inherits the same offset as the face it belongs to.
- **`prefers-reduced-motion: reduce`** hides the card entirely and shows `.wgtrio` — the same
  three real widgets, side by side, at rest. Verified as *zero* running animations, not slowed
  ones.

Two things found by measuring the running animation rather than by looking at it:

1. **A one-frame flash.** The face snaps from the top of the cylinder back to the bottom
   (`+90° → -90°`) in a 0.12ms keyframe window. Hiding the face one keyframe *after* that snap
   left a window where a sampled frame caught it face-on at ~30° — and a frame seek landed in
   it. The face now goes invisible at exactly `33.333%`, the instant it reaches `+90°` and is
   edge-on, so the snap happens behind an already-hidden face and nothing changes on screen.
2. **Clearance.** Mid-turn a corner swings about 26px past the card's resting box, top and
   bottom — the way a rotating cube shows its diagonal. At the spacing first used, the incoming
   card's corner reached into the caption's line box. The margins above and below the card are
   now `44px` and are clearance, not decoration. (A `.flip + .wg-cap` rule was tried and does
   nothing: the reduced-motion `.wgtrio` sits between them in the markup.)

## 4 · Spacing, alignment and polish

- **A spacing scale**, `--sp-xs` 8 through `--sp-xl` 44, plus `--sp-sec` (64px / 104px at
  ≥760px). Every vertical gap on the site is now one of those values.
- **Every inline `style=` attribute is gone** — eight of them across the three pages
  (`margin-top:16px` ×5, two `max-width` overrides, and the `font-size:clamp(28px,5vw,44px)`
  on the support and privacy `h1`s, which were fighting the scale). They became
  `section h2 + p`, `.note` and `h1.page-title`.
- **One radius family:** `--r-card` 14, `--r-widget` 24, `--r-btn` 12.
- **Restrained shadows.** The widget mock's `0 18px 40px -22px rgba(0,0,0,.7)` came down to
  `0 16px 36px -26px rgba(0,0,0,.6)`; the new device frames and flip faces are lighter still.
  Light appearance gets its own warm, weaker set — a shadow tuned for a near-black surface is
  much too heavy over warm paper.
- **Grids aligned to the text column.** The screenshot trio is left-aligned at ≥760px instead of
  centred; the wide capture and the free widget start at the same left edge as the copy.
- **`text-wrap: balance`** on `h2` and the hero deck.
- **Caption measure in px, not `ch`.** `ch` ignores the `.14em` tracking these labels carry,
  which is what was breaking the longest caption onto a two-word second line.
- **Dead CSS removed:** `.wgcycle` and its `@keyframes`, orphaned by the flip. (`.stack` /
  `.stack-l` are still unused and still left alone — they were unused before this pass too.)

## 5 · Horizontal overflow at 320px — fixed, and it pre-existed

Checked at 1280 / 1024 / 900 / 768 / 414 / 375 / 320 on all three pages. Everything is clean
now. At 320px the overview page had a **37px horizontal scroll before this pass** (`367px` of
content in a `320px` viewport) from the header nav and the hero lockup. Three cheap fixes:

- the header row and its nav list may now wrap instead of crowding the right padding — with
  the gaps tuned (14 → 10px between mark and nav, 20 → 18px between links) so the header still
  holds **one** row at 375px and only breaks to two below about 360px;
- `.hs` is `max-width: min(340px, 100%)` rather than a bare `340px` cap;
- `.hero-art` is stretched rather than `justify-self: start`, which was sizing it to the art's
  max-content — a few pixels wider than the column.

## 6 · What was verified, and how

Rendered in Chrome and measured, not eyeballed:

| Check | Result |
|---|---|
| Horizontal overflow, 3 pages × 7 widths (320–1280) | `scrollWidth === innerWidth` everywhere; no element extends past the viewport |
| Hero reading order, 1280 and 375 | headline → subhead → visual → CTA at both |
| The flip, sampled frame by frame through a turn in the live page | rolls; no flash at the snap; caption clears the card at every frame |
| `prefers-reduced-motion: reduce` | `.flip` `display:none`, `.wgtrio` `display:grid`, **0** animations running |
| Light appearance | palette and hierarchy hold; shadows retuned |
| Type inventory — every size/weight/face on the page | 5 sans sizes, 3 weights, mono at 11 and 13 only, plus the documented `.wg` / `.wg-badge` exception |

Two measurement traps worth writing down, because both produced false results first:

- **Headless Chrome clamps its own viewport to 500px wide on macOS.** A
  `--window-size=375` screenshot is a *500px layout cropped to 375px*, which looks exactly like
  a text-overflow bug. Every mobile capture was redone through a CDP session with
  `Emulation.setDeviceMetricsOverride`, which gives a true 375px viewport.
- **`--virtual-time-budget` does not advance CSS animations**, so sampling the flip that way
  returns the same first frame at every budget. The frames above come from seeking the
  animations through the Web Animations API over CDP.

## Not done, deliberately

- **The draft watermark and `noindex` are still in place** on all three pages, and both App
  Store buttons still point at `href="#"`. Unchanged from the last pass; still the owner's call
  at publish time.
- **The hero's Home-Screen vignette is still the CSS mock**, not a real capture. It is the one
  widget on the site that adapts to light appearance and stays sharp at any size, which is worth
  more in the hero than photographic fidelity — and the real free-widget capture now appears in
  its own section two screens down, at Home-Screen scale.
- **No copy was touched.** The only markup edits to running text are one `&nbsp;` and the class
  names on elements whose words are identical.

---

# 29 July 2026 — the copy refresh

Everything edited in that pass, old → new. The site was already in good shape; this was a
refresh of the copy against what the app actually does today, plus the screenshots and the
widget animation the README had been waiting for. **The design system, the layout and the page
structure were unchanged by that pass.**

A backup of the folder as it stood before it is at:

```
~/Library/Mobile Documents/com~apple~CloudDocs/Downloads/stopspotter-backup-2026-07-29-2242/
```

---

## The three things that were actually wrong

Everything else here is polish. These were false:

1. **The privacy policy said there is no server.** There is one — `stopspotter-api.reforged.studio`,
   a Cloudflare Worker the developer runs — and every live departure the app shows comes through
   it. The old *Network requests* section said requests go from your device straight to NJ Transit
   and to "a community-hosted PATH real-time feed", and that they "do not pass through any server
   operated by Reforged Studio". None of that is true any more, and the README anticipated exactly
   this ("When the caching proxy ships"). Rewritten from the code.
2. **The privacy policy said your coordinates "are not stored between sessions."** They are: the
   app keeps your most recent coordinate on the device so the widget can still name a stop when it
   can't get a fresh fix. It expires after 24 hours and is deleted the moment you revoke location
   or drop to approximate. Corrected to say so — the fact is still good news, but it has to be the
   fact.
3. **The support page said NJ Transit real-time was "pending".** It has been live since the M16
   milestone — all four services now have live departures through the proxy, with gate numbers at
   Port Authority. That answer was the first thing a visitor opened.

---

## index.html

| Where | Old | New |
|---|---|---|
| Meta + OG description, hero deck, footer ×2 | `NJ TRANSIT` | `NJ Transit` — see *One change that touches every page* below |
| Hero note | "iPhone, iOS 18 or later" | "iPhone, iOS 26.2 or later" — the project's actual deployment target |
| Hero note | "StopSpotter+ adds widgets for the stops you save" | "StopSpotter+ points a widget at a stop you saved" — what the purchase actually buys |
| Closing CTA note | "Free · iPhone · iOS 18" | "Free · iPhone · iOS 26.2" |
| *One place* section | `<div class="shot wide">` placeholder | `img/saved-modes.png` — a real capture of a bus stop at Port Authority (with its gate) and a PATH station sitting next to each other in the saved list, plus a one-line caption |
| *Inside the app* | three dashed `.shot.phone` placeholders | `img/stop-detail.png`, `img/search.png`, `img/saved.png` — real captures |
| *Inside the app* → Terminals cell | "Terminal gate assignments are best-effort and may change. Check station signage before boarding." | "Where NJ Transit reports the gate a bus is boarding from, it sits on the row — so you read it here instead of hunting the concourse board. Gate assignments can change; check the signage before boarding." |
| *The free widget* | a CSS mock of a widget | `img/widget-nearest.png` — the real free widget, rendered by the app |
| *StopSpotter+* | three CSS-mock widgets in a scrolling row + page dots | one widget-sized frame crossfading three real widget captures (see below) |
| *StopSpotter+* caption | "One stack, three stops — swipe to reach the next" | "One widget per saved stop — stack them and swipe" |

### The widget animation

The three-across row is now a single frame at systemMedium proportions (360 × 169 CSS px,
aspect ratio 1035:487) holding three real captured widget renders stacked on top of each other.
They crossfade in place — 3.0 s hold, 0.5 s fade, 10.5 s round trip, looping — so it reads like a
widget on a Home Screen changing rather than a carousel moving.

- **No JavaScript.** Three `<img>`s, one `@keyframes`, staggered negative animation delays.
- The frame carries the card colour behind the stack, so the instant where two frames are both
  part-transparent dissolves between two identical surfaces rather than showing the page through.
- **`prefers-reduced-motion: reduce` hides the cycler entirely and shows `.wgtrio`** — the same
  three widgets, side by side, at rest. Not a slowed-down animation: no animation.
- The three faces are a bus stop on schedule data, a PATH station updated 30 seconds ago, and
  Hoboken Terminal showing the 22, the 126 and PATH on one card. They were chosen so the loop
  carries the coverage claim and the live-vs-scheduled labelling as well as the "your stops" one.

---

## support.html

Meta description now mentions gates at Port Authority. `NJ TRANSIT` → `NJ Transit` in four
places. The FAQ went from 8 entries to 13; here is the whole new list, in order.

| # | Question | Status |
|---|---|---|
| 1 | What's the difference between "updated 2 min ago" and "Schedule data"? | **Replaces** the old "Why do my times say Scheduled instead of Live?", whose answer said NJ Transit real-time was pending. It isn't. Now explains the two footer states and the per-row `Live` / `Scheduled` words. |
| 2 | My widget says "Schedule data" even though the app shows live times. | **New.** A widget built during the first-run window comes up on timetable data and asks to be rebuilt within about two minutes rather than waiting for its usual slot, so it heals itself. Tells people to wait a minute or two, and what it means if it doesn't. |
| 3 | Why does my widget show times that look old? | Kept; the stale wording now matches the app's own ("may not reflect real-time data" past fifteen minutes). |
| 4 | I tapped Edit Widget and the list of stops just spins. | **New.** Restart the phone. Says plainly that the list is built on the device with no network involved, that the stuck thing is iOS's own record, that it was seen once after repeated delete-and-reinstall cycles, and that a restart cleared it. Asks people to write in if a restart doesn't fix it. |
| 5 | Why did adding a widget ask for my location? | **New.** The gentle version: there is one widget, its default setting is Nearest stop, and iOS asks once for the whole app — so the prompt appears even for someone who only ever wants a saved stop. Says you can decline and the saved-stop widget still works. |
| 6 | What is free, and what costs money? | Kept, retuned: the paid tier now reads as "pointing that widget at a stop *you* saved" rather than "adds widgets". |
| 7 | What do I choose when I save a stop? | Unchanged. |
| 8 | How do I put a saved stop on the Home Screen? | Retitled from "How do I add a widget and choose which stop it shows?"; absorbed the widget-refresh-budget note that used to sit loose at the bottom of the marketing page. |
| 9 | I bought StopSpotter+ on another phone / I reinstalled the app. | **New.** Restore Purchases, where to find it, and refunds → reportaproblem.apple.com, because they are Apple's to give and we never see payment details. |
| 10 | Which gate is my bus at Port Authority? | Retitled from "How accurate is the gate information…" — the old title invited the one word this site must not use about data. Same caveats, now leading with what the feature is. |
| 11 | A station is showing no departures at all. | **New.** Trips that terminate at a station produce no departures there, so an empty board at the end of a line is the correct answer. The README flagged this as wanting an entry. |
| 12 | Which agencies and modes are supported? | Now says all four services have live departures, not just PATH. |
| 13 | Where does the data come from? | **New.** Carries the sanctioned attributions and the independence disclaimer inside the FAQ itself, not only in the footer. |
| — | Is there an Apple Watch app? | Unchanged. |

---

## privacy.html

Re-verified line by line against the app's source, its `Info.plist` usage strings, its
entitlements, the Worker's request handling and its StoreKit use. **Every correction below makes
a claim narrower or more exact; none adds a favourable claim.**

| Section | Old | New |
|---|---|---|
| Title | "It doesn't collect anything." | "There's nobody to be." — the old headline is a stronger claim than the page can support now that a server is in the picture. The new one says the true thing: the app never learns who you are. |
| Lead + meta + OG | "No accounts, no analytics, no server." | "No accounts, no analytics, no tracking." |
| Fact strip | "No server" | "No profile of you" |
| One-sentence callout | "…no server of its own; …location used on-device only" | Now four clauses: no accounts, device-only storage, location not sent anywhere, and "the one server the app talks to is asked which stop, never who is asking." |
| Location | "…not transmitted … and they are not stored between sessions." | Coordinates still not transmitted — that part is verified and now set in bold. The "not stored" claim is replaced with the truth: the last coordinate is kept on the device for the widget's benefit, expires after 24 hours, and is deleted on revoke or on a drop to approximate location. |
| Network requests | Requests go direct to NJ Transit's feeds and a community-hosted PATH feed; "those requests do not pass through any server operated by Reforged Studio" | Complete rewrite. One address, `stopspotter-api.reforged.studio`, run by Reforged Studio; the device never contacts the agencies itself; NJ Transit's licence requires that shape. The request carries a stop identifier and nothing else. The IP any request carries is acknowledged rather than glossed, along with Cloudflare's role as host. |
| Schedule data | "Timetables … update when the app updates. Reading them involves no network request at all." | Reading them still needs no network — true and worth keeping. But the app does check the same server, at most once a day and on Wi-Fi, for a newer copy of the database, so a timetable change doesn't wait on an App Store release. That check was undisclosed. |
| Purchases | Apple handles it, we never see payment details | Same, plus: there is no purchase server of ours, and the entitlement is recorded on the device. |
| Children | "StopSpotter collects no data from anyone, including children." | "StopSpotter asks nobody who they are, and that includes children." — the absolute version no longer holds once a request reaches a server carrying an IP. |
| Date | 25 July 2026 | 29 July 2026 |

### Verified as already correct, and left alone

- **No accounts.** Nothing anywhere in the app authenticates a user.
- **No analytics, no tracking, no third-party SDKs.** The Xcode project has no package
  dependencies at all — there is no analytics or crash-reporting library to disclose.
- **Saved stops, names and preferences never leave the device.** No sync, no account, no copy.
- **Location is resolved on the device.** The nearest-stop lookup runs against the transit
  database the app carries; the only outbound request carries a stop identifier.
- **Purchases are entirely Apple's.** StoreKit only; no receipt server, no payment details.
- **One host.** The whole shipping app contains exactly one production URL.

### Flagged rather than resolved

- **How long Cloudflare retains its own request logs** is a fact about the hosting account, not
  about the code, so the page says only that short-lived operational logs may exist. If a
  zero-retention position is wanted, that is a Cloudflare setting to make and then state.
- **"Data Not Collected"** on Apple's privacy questionnaire is still the answer the code
  supports, but the README already lists it as a lawyer question and this pass does not settle
  it.

---

## One change that touches every page

**`NJ TRANSIT` → `NJ Transit`, 11 occurrences.** The app settled this deliberately: the licence
mandates *statements*, not typography, and setting a vendor's name in caps inside running prose
is itself a faint trade-dress gesture, which the paywall spec tells the product to avoid. The
app's shipping code holds the agency name as `NJ Transit` in one constant, and composes the
independence disclaimer and both attributions from it. The site's own README says the
independence disclaimer is the one statement that most needs to match everywhere — so the site
now matches the app, word for word:

> StopSpotter is an independent app. It is not endorsed by, affiliated with, or supported by
> NJ Transit or the Port Authority of NY & NJ.

> Predictions from NJ Transit data. Derived from publicly available PATH data. Data
> availability is not guaranteed.

**If you would rather the site keep the agency's own all-caps setting, this is the one change to
reverse** — it is a single find-and-replace across the three pages, and nothing else depends on
it.

---

## stopspotter.css

Every change here is required by a change above.

| Change | Why |
|---|---|
| `.shot` split into `div.shot` (the dashed placeholder) and `img.shot` (a real capture) | The placeholder forced `aspect-ratio: 9/19.5` and `16/10`. A real screenshot has its own proportions and would have been squeezed. Images now keep their intrinsic ratio and lose the dashed box. |
| `.wgcycle`, `@keyframes wgcycle`, `.wgtrio`, and a `prefers-reduced-motion` block | The widget animation. |
| `.wgshot` | The single real widget capture in the free-widget section, sized to match the CSS mocks it sits among. |
| `.doc code` | The privacy page now names the API hostname; there was no `code` style. |
| `.stackset` and `.dots` deleted | Their only markup was the three-across widget row the animation replaced. A comment marks the spot. |
| Print block hides `.wgcycle`, `.wgtrio`, `.wgshot` | It already hid the widget mocks these replaced. |

`.stack` / `.stack-l` are also unused, but they were unused *before* this pass too — no markup
in the folder has ever referenced them. Left alone deliberately rather than swept up.

---

## Files added and removed

**Added — `img/`, all captured from the app running in the simulator against live data on
2026-07-29, dark appearance, no device frames:**

| File | What it is |
|---|---|
| `stop-detail.png` | Port Authority Bus Terminal departures — every row labelled `Live` and carrying its gate |
| `search.png` | The search screen, with bus / rail / light rail / PATH as one row |
| `saved.png` | Four saved stops in one list, bus and PATH together, live and scheduled mixed |
| `saved-modes.png` | The wide one: a bus stop at Port Authority with its gate, and a PATH station, side by side |
| `widget-nearest.png` | The free widget — Journal Square, 720 ft, PATH departures |
| `widget-bus.png` | Crossfade frame 1 — a 126 to New York, `Schedule data` |
| `widget-path.png` | Crossfade frame 2 — PATH to 33rd Street, `updated 30s ago` |
| `widget-mixed.png` | Crossfade frame 3 — Hoboken Terminal, the 22 + the 126 + PATH on one card |

The widget frames are the app's own widget views, rendered by the app's widget-state harness and
cropped to the exact card bounds — not mockups, and not redrawn.

**Added at the root:** `favicon.png` (32), `apple-touch-icon.png` (180), `icon-512.png` (512).
All three were referenced by every page's `<head>` and none of them existed — the README's
"icons not appearing" section was describing a real gap. Generated from the app's own dark app
icon, composited on the site's `#0E0B09` surface with the same rounded tile treatment and the
same 66.7% mark scale as `icon-tile.png`, so all four icons match.

**Removed:** `stopscout.css` — byte-for-byte identical to `stopspotter.css` apart from its
header comment, and referenced by nothing. A leftover from the rename.

---

## Deliberately NOT done

- **The draft watermark and `noindex` are still in place** on all three pages. Removing them is
  README step 1 and it is the owner's call at publish time, not part of a copy refresh.
- **Both App Store buttons still point at `href="#"`.** They keep their `data-appstore` hook.
  Apple's supplied badge artwork goes in at the same time as the real URL.
- **The hero's Home-Screen vignette is still a CSS mock.** It is a decorative composition — a
  widget on a grid of blank app tiles — rather than a claim about what the widget looks like, and
  replacing it means rebuilding the `.hs` layout. The two places that *do* claim to show the
  widget now show real captures.
- **`img/nearby.png` was not produced.** The Nearest module never finished loading in the
  simulator across several attempts, with location granted and set. Rather than ship a
  spinner, the third phone slot shows the saved-stops list instead, and the section's copy and
  alt text were written to match. Worth checking on a real device — if the nearby list is slow
  or stuck there too, that is a bug and not a screenshot problem.

---

## Ready to upload

The folder is complete and self-contained: no build step, no framework, no external requests, no
JavaScript. To preview it locally, serve it rather than opening `index.html` off disk — the
internal links are deliberately extensionless and need a server that resolves them:

```
cd stopspotter && python3 -m http.server 8000
```

Under `python3 -m http.server` the `privacy` and `support` links 404, because it has no
extensionless routing; Cloudflare Pages does. Every stylesheet, icon and screenshot resolves.
