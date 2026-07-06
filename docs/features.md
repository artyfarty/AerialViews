# Immich features added by this fork

The Immich-specific behaviour this fork adds on top of upstream. Portrait rendering has its own
doc ([portrait-rendering.md](portrait-rendering.md)).

## Smart slideshow

**Problem.** Upstream picks albums + "latest" and then randomizes across the whole flat pool. Two
failure modes in practice:

1. A single large album dominates the mix — smaller albums rarely show.
2. Burst sequences (photos taken seconds apart) consume several consecutive slideshow slots
   showing essentially the same scene.

Comparison point: Google's built-in TV screensaver, which spreads across albums more evenly.

**Approach.** Two cooperating mechanisms, in tension by design (variety across events vs. not
starving large collections):

- **Temporal clustering of bursts** (`ImmichClusterer`): photos within a configurable time gap of
  each other are grouped, and a single cluster occupies one slideshow slot (one representative,
  re-randomized among cluster members at render time rather than fixed at fetch time). This
  collapses near-duplicate bursts.
- **Temporal-spread-weighted album sampling**: sample an album first, then a photo within it, and
  dampen the selection weight of albums that are tightly clustered in time (single-event albums)
  while leaving wide-date-spread albums ("collections") less dampened. This evens out exposure so
  a 5-photo event and a 500-photo collection each get reasonable airtime.

**Exempt pattern.** Albums whose names match a user-supplied regex bypass clustering — intended
for scans where capture timestamps are unreliable (e.g. bulk-digitized film). The committed
default is **empty**; any real pattern lives only in the user's on-device settings, never in the
repo.

This sampling requires knowing which album each asset belongs to — see the per-album fetch note in
[immich-provider.md](immich-provider.md).

## Overlay metadata tags

The overlay system shows metadata in screen-corner slots. This fork adds Immich-oriented tags so
several independent fields can be shown at once (each in its own slot — putting multiple tags in
one slot renders only the first non-empty one):

- **SOURCE_POOL** — the asset's Immich album name, or a Favorites / Rated / Random / Recent label
  identifying which pool it came from.
- **LOCATION** — reverse-geocoded city/country from the asset's Immich/EXIF metadata.
- **DATE_TIME_TAKEN** — humanized capture date + time.

Per-asset album attribution for SOURCE_POOL comes from the per-album fetch (see
[immich-provider.md](immich-provider.md)); an asset in multiple selected albums shows its album
names joined.

## Scope

Per the maintainer's use case, these features target the **Immich code path only** and are not
generalized to other AerialViews providers unless it costs nothing to do so.
