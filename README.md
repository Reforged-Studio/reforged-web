# Reforged Studio — Production Build

The organization landing page for `reforged.studio`. Drop these at the domain root.

## Files

- **`index.html`** — the complete page. Self-contained: all CSS inline, semantic HTML, responsive, accessible. Nothing to compile.
- **`favicon.svg`** — the "R" + ember-bar mark.

## Before you go live (two assets to generate)

1. **PNG favicons** — generate `favicon-16.png`, `favicon-32.png`, and `apple-touch-icon.png` (180×180) from `favicon.svg`. Any favicon generator does this in seconds. (`index.html` already links to them.)
2. **`og.png`** (1200×630) — the social-share preview image. `index.html` references `/og.png`. Design: the "Reforged Studio" wordmark on the dark background; no "coming soon."

## The design, briefly

- **Wordmark:** "Reforged" (Newsreader serif) with "STUDIO" (Inter, heavy, full-ember `#D97F4A`) nested into the descender pocket of the "g," justified edge-to-edge. On mobile it falls back to a centered tracked label beneath the stacked wordmark — the nested treatment is a desktop flourish that doesn't hold at small sizes.
- **Tagline:** *Stronger for what's next.* — grouped tight under the wordmark as part of the identity.
- **About:** one sentence, set apart below.
- **Footer:** two-zone — "Our Work / Connect" links left, legal block right.

## Key implementation notes

- **STUDIO positioning is in wordmark-em units** (`--wm-size`), not viewport percentages. This is deliberate — it keeps STUDIO locked to the "g" pocket at every screen width. If you ever retune it, the three coefficients are `width` (letter spread), `font-size` (height in the band), and `right`/`bottom` (position). Don't convert these back to %-of-box or it will drift.
- **Two ember values are intentional:** `--ember` (#D97F4A, full) on STUDIO + footer arrows + favicon; `--ember-muted` (#B86B3D) is defined but currently unused on this page — kept for the future full site.
- **Accessibility:** the `<h1>` carries `aria-label="Reforged Studio"`; the per-letter STUDIO spans are `aria-hidden` so screen readers say "Reforged Studio," not "S-T-U-D-I-O."
- **`prefers-reduced-motion`** is honored — load animation disabled, content renders in final position.
- **Fonts** are Google Fonts (Newsreader + Inter); import is in `<head>`. **Grain texture** is an inline SVG data-URI — no asset to host.
- **StreetProof / privacy / support** live separately under `/streetproof`; this page only links there.

## Design tokens (for the future full site)

Colors: `--surface-base #14110F`, `--surface-deep #0E0C0A`, `--surface-lift #1F1A16`, `--ink-primary #EFE8DF`, `--ink-secondary #8A8076`, `--ink-tertiary #4F4740`, `--ember #D97F4A`, `--ember-muted #B86B3D`.
Type: Newsreader (display, `opsz 72` on the wordmark) + Inter (UI/meta).
All reusable; the nested-STUDIO lockup, the copy, and the single-screen composition are specific to this page.
