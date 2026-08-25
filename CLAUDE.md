# Reforged Studio Website — working agreement

## What this repo is

The company site for **reforged.studio**: hand-written static HTML — no build step, no framework,
no package manifest. A root landing page plus one directory per shipped app.

**Hosting:** Cloudflare Pages, per `streetproof/README.md` and `stopspotter/README.md` ("Cloudflare
Pages builds on commit"). Pages 308-redirects `/privacy.html` → `/privacy`, so internal links are
deliberately extensionless (`href="privacy"`, `href="./"`). Keep it that way.

**Deploy mechanism:** push to `main` at `https://github.com/Reforged-Studio/reforged-web` —
Cloudflare Pages builds on commit, so **push = deploy**. No other config exists in the tree.

## Deploy and commit rules

Sessions **never** deploy, publish, or purge — deploying *is* committing here. Sessions implement
and stage; the owner commits and pushes:

```bash
cd /Users/bobferriola/ReforgedStudioWebsite && git add -A && git commit -m "..." && git push
```

Plain descriptive commit messages. **No milestone numbering in this repo** — that belongs to the
app projects.

## Branch-first workflow — main is production

Push to `main` = production deploy. Therefore **all work** — any size, any file — lands on a
working branch (`draft/<topic>`) by default. Sessions stage on the branch and hand the owner a
branch commit+push block.

Cloudflare builds a preview URL for every non-production branch push; the owner reviews the
rendered preview there.

Merging to `main` happens **only** on explicit owner approval given after the owner has seen the
preview (or waived it in so many words). A session **never** hands a main-push block as the
default next step; the merge block is handed only after the owner says to land it.

There is no small-edit exception. If the owner wants to skip the branch for a trivial change, the
owner says so explicitly, per change; sessions never propose it.

## Structure

- `index.html` — landing page. All CSS in one inline `<style>`, Google Fonts, inline-SVG grain.
  Root `README.md` documents its lockup math; read it before touching the wordmark.
- `streetproof/` · `fitferry/` · `stopspotter/` — each has `index.html`, `privacy.html`,
  `support.html` (the last two are required App Store Connect fields) plus its own images, and its
  own `README.md` (StopSpotter also `CHANGES.md`). Read the folder's README before editing it.
- **Nothing is generated** — every file is hand-edited. Leave alone unless asked:
  `stopspotter/stopscout.css` (pre-rename copy, referenced by no page), `stopspotter/incoming/`,
  `stopspotter/reference/`.

## Styling — inconsistent by history

Root `index.html` uses an inline `<style>` with its own `--surface-*/--ink-*/--ember*` tokens.
`fitferry/` and `stopspotter/` each have one external stylesheet shared by all three pages — the
pattern to follow. `streetproof/` is the wart: **three separate inline `<style>` blocks**, one per
page, each re-declaring `--sp-*` with a slightly different subset. They have already drifted; a
token change there must be made in all three files.

## Design authority (pointer map — read-only, never fork)

Each app's design language is **owned by its app repo**. Pages here read from those sources; never
copy a spec or token sheet into this repo. Cross-repo reads are permitted and read-only — a website
session never edits an app repo.

| Subpage | Repo (`~/XcodeProjects/…`) | Spec | Tokens in source |
|---|---|---|---|
| `streetproof/` | `StreetProof` | `docs/design/streetproof-design-spec.md` + its `## Owner amendments` blocks (**amendments supersede** the sections they name) | `StreetProof/UI/DesignTokens.swift` |
| `fitferry/` | `FitFerry` | `docs/design/fitferry-redesign-spec.md` (v1.2, supersedes v1.1) | `FitFerry/Design/FFDesign.swift` — owner amendments live in its **header comment**, not the spec |
| `stopspotter/` | `StopSpotter` | `docs/design/stopscout-design-spec.md` + its `A…` amendment ledger; `stopscout-selection-model.md` supersedes §5.4/§7; `stopscout-addendum-*.md` | `StopSpotter/UI/DesignTokens.swift`; palette of record `StopSpotterShared/WidgetPalette.swift` |

**Where site CSS conflicts with an app repo's tokens, the app repo wins and the site CSS is the bug.**
Known live drift: `streetproof/*` uses `--sp-blue: #0290FE`, which appears nowhere in StreetProof
(app route accent is `#3FA3F2`; AccentColor asset `#1A85E6`). `fitferry/styles.css` uses
`--green-text: #0B8A70` where the app has `FFColor.tealDeep` `#178A74` (brand `#1EB093` does match).
`stopspotter.css` tracks `WidgetPalette` correctly — keep it that way.

## Company brand — what this repo owns

Only the root landing page, and root `README.md` is its record: palette `--surface-base #14110F`,
`--surface-deep #0E0C0A`, `--surface-lift #1F1A16`, `--ink-primary #EFE8DF`, `--ink-secondary
#8A8076`, `--ink-tertiary #4F4740`, `--ember #D97F4A`, `--ember-muted #B86B3D` (defined, unused);
type Newsreader (display, `opsz 72`) + Inter (UI/meta); logo `favicon.svg` ("R" + ember bar). The
nested-STUDIO lockup is landing-page-specific and positioned in wordmark-em units (`--wm-size`) on
purpose — do not convert to %. **There is no written brand guide beyond this** — don't invent one;
subpages keep their app's identity, not the company palette.

## Scope boundary

Website work happens in sessions against this repo, under this file. App-project sessions do not
touch this repo (StreetProof's own `CLAUDE.md`: the website "is managed separately and does not live
in this repo"), and website sessions do not modify app repos.
