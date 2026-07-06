# Fork & branch layout

## Relationship to upstream

This repo is a fork of **[theothernt/AerialViews](https://github.com/theothernt/AerialViews)**
(GPL-3.0). It tracks upstream via the `upstream` remote and keeps the upstream `LICENSE`,
attribution, and README intact. The fork's own additions are Immich-focused (see
[index.md](index.md)); everything else is upstream's.

## Branches

- **`master` — the fork's main line / daily driver.** The combined stack of this fork's Immich
  features plus small personal deploy tweaks (a distinct debug app-name, and disabling dev-only
  tooling so it doesn't intrude when used as a real screensaver). This is what builds and runs on
  the maintainer's TV. Because it carries personal deploy tweaks, `master` itself is **not** a
  clean PR target — upstream contributions come from the feature branches below.

- **Feature branches (PR candidates).** Each isolates one upstream-mergeable feature with neutral
  defaults (empty regexes, no personal values), so a PR stays clean:
  - **smart-slideshow** — Immich temporal clustering + album-weighted sampling.
  - **immich-improvements** — periodic provider refresh, DATE_TIME_TAKEN / SOURCE_POOL overlay
    tags, and the modern Immich search-API usage.
  - **portrait-handling-improvements** — vertical Ken Burns pan + face-aware crop/bias for
    portraits (roots on smart-slideshow because face-carrying alternates need its clustering).
  - Immich **v3 compatibility** — album assets via `search/metadata`, faces via `/api/faces`
    (see [immich-provider.md](immich-provider.md)).

## Working rules

- Before rebuilding the APK for the TV: build from `master` (the deploy line).
- Adding an upstream-worthy feature: commit it on the appropriate clean feature branch (neutral
  defaults, no personal data), then merge/rebase `master` on top.
- Adding a personal-only deploy tweak: keep it on `master`, never on a feature branch.
- Keep personal configuration (server address, API key, album selection, private regex patterns)
  in on-device settings only — never in committed defaults. See the upstream-hygiene note in
  [../CLAUDE.md](../CLAUDE.md).

## Build variants

The flavor/buildType matrix that decides applicationId and signing is documented in
[build-and-deploy.md](build-and-deploy.md).
