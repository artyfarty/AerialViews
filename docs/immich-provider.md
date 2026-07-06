# Immich provider — API integration

How the fork pulls media from an [Immich](https://immich.app) server, and the API quirks worth
knowing — especially the **Immich v3** breaking changes that this fork migrated to.

Source lives in `app/src/main/java/com/neilturner/aerialviews/providers/immich/`
(`ImmichApi.kt`, `ImmichRepository.kt`, `ImmichMediaProvider.kt`, `ImmichClusterer.kt`,
`ImmichAssetMapper.kt`).

## Auth & connection

- Base URL and API key come from app settings (never hardcoded). API key is sent as the
  `x-api-key` header on every request. Shared-link mode uses a `key`/`slug` query param instead.
- Retrofit + kotlinx-serialization; the JSON converter ignores unknown keys, so response DTOs
  declare only the fields the app needs (Immich returns many more).

## Source pools

The provider builds its playback pool from one or more independently-configured sources, then the
slideshow layer samples across them:

- **Selected albums** — a set of album IDs chosen in settings.
- **Favorites** — up to N favorited assets.
- **Rated** — assets at configured star ratings.
- **Random** — N random assets.
- **Recent** — the N most-recently-added assets.

## Immich v3 breaking changes (and how they're handled)

Immich v3 changed several endpoints. This fork targets v3 while staying compatible with the
latest 2.x line where practical. The important ones for this provider:

### 1. Album assets are no longer inlined in `GET /api/albums/{id}`

In v2, fetching an album by id returned its `assets` array inline. In **v3 that array is always
empty** (the endpoint still returns album metadata + `assetCount`). Relying on it silently yields
zero assets — the failure looks like "No assets found in any of the selected albums".

**Handling:** fetch each album's assets via `POST /api/search/metadata` with an `albumIds` filter,
paginating on the response's `assets.nextPage` cursor. Album *names* are resolved once in bulk via
`GET /api/albums`. This path also works on Immich 2.x.

Why per-album rather than one batched `albumIds: [all]` search: `search/metadata` does not report
*which* album an asset came from, and that per-asset membership is needed by both the SOURCE_POOL
overlay and the smart-slideshow album-weighted sampling. So names are batched (one call) but
asset fetches stay per-album (run concurrently).

### 2. `GET /api/assets/random` removed → `POST /api/search/random`

The random pool uses `POST /api/search/random` with a `SearchMetadataRequest` body.

### 3. Face bounding boxes moved out of the asset detail

In v2, `GET /api/assets/{id}` returned `people[].faces[]` with bounding boxes. In **v3 the person
objects no longer embed `faces`**; face detection results moved to a dedicated endpoint:
`GET /api/faces?id={assetId}`, returning one entry per detected face with `boundingBoxX1..Y2` plus
the `imageWidth`/`imageHeight` of the frame the detection ran on. The portrait face-bias feature
(see [portrait-rendering.md](portrait-rendering.md)) uses this endpoint.

### 4. Album-list `shared` param renamed to `isShared`

`GET /api/albums` renamed the `shared` query param to `isShared`. This only affects the album
*picker*'s shared/owned split (v3 ignores the unknown `shared` param rather than erroring, so
albums still list). Low-impact; note it if touching `fetchAlbums()`.

## Pagination pattern

`POST /api/search/metadata` returns `{ assets: { items, total, count, nextPage } }`. Page with a
1-based `page` field in the request; stop when `items` is empty or `nextPage` is null. Page size
250 is Immich's practical maximum.

## Asset fields used

From search/album items: `id`, `type` (IMAGE/VIDEO), `originalPath`, `localDateTime`,
`width`/`height` (post-orientation pixel dims, used for portrait detection), `exifInfo`
(when `withExif=true`, for the LOCATION overlay). Thumbnails/originals are fetched by asset id.
