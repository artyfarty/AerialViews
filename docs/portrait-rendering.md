# Portrait rendering — Ken Burns pan & face bias

Portraits (taller-than-screen images) look bad center-cropped on a landscape TV. This fork adds
cinematic handling in `ImagePlayerView` plus optional face-aware biasing.

## Vertical Ken Burns pan

For a portrait whose decoded bitmap is taller than the container, apply a scale + vertical
translate animation that pans the image top-to-bottom (or bottom-to-top) over the slide's dwell
time. This works uniformly for **100%** of portraits and is the primary behaviour — no server-side
face data required.

## Face-aware biasing (refinement on top of the pan)

When a portrait has a detected face, the pan range is narrowed so the subject stays in view
throughout, and the pan direction is chosen to start from the side away from the face. There's
also a face-biased rule-of-thirds center-crop path for the CENTER_CROP scale preference. Portraits
without a detected face simply fall back to the uniform pan.

Face rectangles are fetched per portrait via `GET /api/faces?id={asset}` (see
[immich-provider.md](immich-provider.md) — this moved endpoints in Immich v3). Only a fraction of a
typical library's portraits have an ML-detected face, so the pan-without-bias fallback matters.

### Critical gotcha: face bbox coordinate frame

Immich face bounding boxes are **not** in the original asset's coordinate system. They're in the
frame of the **downscaled preview** the ML ran on, which Immich caps around 1440–2160 px on the
long side regardless of original resolution:

```
asset.width / height                 → original dimensions (e.g. 4000×6000)
face.imageWidth / imageHeight        → the ML-preview frame the bbox lives in (e.g. 1440×2160)
face.boundingBoxX1..Y2               → pixel coords within that ML-preview frame
```

Normalize the bbox against `face.imageWidth/imageHeight` (**not** `asset.width/height`) to get a
0..1 fraction, then project onto whatever image you display. Using the asset dimensions shifts
every box toward the upper-left, because the naive ratio treats the coords as too small a fraction.

When multiple faces exist, bias toward the largest bbox (or the average centre-of-mass) — this
looks fine for small groups.

## Known limitation: a few portraits display sideways

A small percentage of shot-in-portrait photos decode with **landscape** pixel dimensions, so the
aspect-ratio branch treats them as landscape: Ken Burns doesn't activate and they get
center-cropped in the wrong orientation (lying on their side).

This is **not** a Ken Burns bug — the animation only scales/translates, never rotates. The root
cause is upstream in the decode/preview pipeline, most often: the server's preview generation
stripped EXIF orientation from the served JPEG without rotating the pixels first (seen with some
HEIC sources / re-scans / camera makes). The feature merely makes the pre-existing issue *visible*,
because correctly-oriented portraits now pan while the wrong ones sit static and obviously sideways.

**Diagnosing:** open an affected photo in the Immich web UI. If it's sideways there too → a
server-side issue (fix via *Refresh metadata* / *Regenerate thumbnail*, or an upstream Immich bug).
If it's correct in the web UI but sideways here → a decode issue on the client side.

**Deferred mitigation:** the Immich image path currently skips the header EXIF-extraction that
other sources use (Immich provides metadata via its API, so re-reading the stream was considered
wasted work). A fix would re-enable EXIF extraction for Immich too and rotate the bitmap manually
before display — deferred because only a small % of libraries are affected and it needs a careful
benchmark so it doesn't slow the common correctly-oriented case.
