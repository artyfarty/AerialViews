# AerialViews

Personal fork of [theothernt/AerialViews](https://github.com/theothernt/AerialViews) — an
Android/Google TV screensaver — tuned as an **Immich photo-slideshow**: smart temporal
clustering, portrait Ken Burns + face bias, and support for the Immich v3 API.

> **Visibility:** Public

## What this fork is

Upstream AerialViews is a general Apple-TV-style aerial-video screensaver with several content
providers (local files, Apple/Comma aerials, Immich, …). This fork keeps all of that but invests
specifically in the **Immich provider**, which upstream treats as a minor feature. The extra work
lives in a small set of feature branches (see [docs/fork-and-branches.md](docs/fork-and-branches.md))
and is combined into the deploy branch that runs day-to-day on the maintainer's TV.

The added Immich behaviour — smart slideshow, portrait handling, overlay tags, and the Immich v3
API migration — is documented under `docs/`. Read [docs/index.md](docs/index.md) first.

This is a **GPL-3.0** fork; the upstream `LICENSE` and attribution are retained.

## Directives

- All configuration (server URLs, API keys, hostnames) **must** come from app settings / config
  files, never hardcoded. Use generic placeholders in examples — never a real server address.
- **Read `docs/` before planning** — consult existing design notes and API findings before
  proposing changes. Start from the docs index (`docs/index.md`). The docs capture hard-won
  Immich API quirks (e.g. the v3 breaking changes) and rendering gotchas (face-bbox coordinate
  frames, EXIF-orientation edge cases).
- **Update `docs/` before committing** — when implementation reveals new findings or changes
  architecture, update the relevant doc and keep `docs/index.md` in sync in the same change.
- **Language** — all committed artifacts (code, comments, commit messages, docs) are written in
  **English**. Conversation with the user may be in any language.

## Upstream hygiene (for anything that might be PR'd back)

- Code defaults and example values committed here must be generic enough to serve **all** users,
  not tailored to one personal library. Regex/pattern defaults tied to a private library must be
  **empty**, never a personal pattern. Keep feature commits upstream-mergeable — no hardcoded
  server addresses, user IDs, or album names.
- Personal-only tweaks (dev app-name, disabling dev tooling for daily-driver use) stay isolated on
  the deploy branch and are **not** part of PR-candidate branches. See
  [docs/fork-and-branches.md](docs/fork-and-branches.md).

## Public-project rules

- This is a **public** project (mirrored to GitHub with public visibility).
- **Never expose personal data** in code, comments, examples, commit messages, docs, or any
  committed file: IP addresses, hostnames or domains of personal infra, API keys/tokens, asset or
  user IDs, GPS coordinates of real photos, personal album names, or local filesystem paths that
  reveal personal info. Keep deployment-specific details (a real server IP, device IP, signing
  identities) in Claude memory or local ignored files — never in the repo. Treat everything
  committed as world-readable.
- A `LICENSE` (GPL-3.0) is present and inherited from upstream; keep it and the attribution intact.

## Sibling-project initialization

- If you create a sibling project (anything at `../<name>` outside this repo), give it the same
  baseline setup before writing substantial code: its own `CLAUDE.md` (purpose, directives,
  visibility), a `docs/` folder with an index, and an initialized git repo with a sensible
  `.gitignore`. Decide and record its visibility first, and apply matching precautions.
