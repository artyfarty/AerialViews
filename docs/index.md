# Docs index — AerialViews (Immich fork)

Index of this fork's documentation. Read the relevant doc **before** planning work, and keep this
index in sync when you add, rename, or remove docs. These docs cover only what this fork *adds or
changes* over upstream; for general AerialViews behaviour see the upstream project.

| Document | What it covers |
| --- | --- |
| [fork-and-branches.md](fork-and-branches.md) | How this fork relates to upstream, the feature/deploy branch layout, and the build-variant matrix. |
| [immich-provider.md](immich-provider.md) | Immich API integration: endpoints used, the Immich **v3** breaking changes and how they're handled, pagination, source pools. |
| [features.md](features.md) | The Immich-specific features: smart slideshow (temporal clustering + album-weighted sampling), source pools, and overlay metadata tags. |
| [portrait-rendering.md](portrait-rendering.md) | Portrait handling: vertical Ken Burns pan, face-aware biasing, the face-bbox coordinate-frame gotcha, and the known EXIF-orientation limitation. |
| [build-and-deploy.md](build-and-deploy.md) | Building the app and deploying a debug build to a TV device over network ADB, without disturbing a Play-installed copy. |

## Conventions

- One doc per topic/area. Record hard-won knowledge: API quirks, edge cases, known limitations,
  and the *why* behind decisions.
- When code changes invalidate a doc, update the doc in the same change.
- **Public repo:** no personal data in docs — no real server/device IPs, hostnames, signing
  identities, personal album names, or local user paths. Use placeholders (`<immich-host>`,
  `<device-ip>`).
