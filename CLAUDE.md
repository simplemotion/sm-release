# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this repo is

`simplemotion/sm-release` is the **public** binary-distribution surface for SimpleMotion's **release** channel.

- **Audience:** All consumers — stable production builds.
- **Stability:** Stable. Promoted from `preview` after validation. Owns `releases/latest` of this repo.

Releases here are GitHub Release assets (binaries + `.sha256` + `.sigstore.jsonl`). No source code lives here — this repo is purely a distribution surface.

## Four-repo architecture

| Repo | Visibility | Role |
|---|---|---|
| `simplemotion/sm-install` | public | Installer scripts + `install.simplemotion.com` Pages |
| `simplemotion/sm-release` | public | Production binaries |
| `simplemotion/sm-preview` | public | Preview / beta binaries |
| `simplemotion/sm-develop` | internal | Development — earliest builds |
| `simplemotion/sm-testing` | private | In-flight test builds |

Each channel has its own `releases/latest` namespace — no prerelease-flag coordination across repos. Consumers reach this channel via `--channel release` in the installer scripts at `install.simplemotion.com`.

## How releases land here

Releases arrive via `repository_dispatch` from per-product source repos (in the `3400-0000-SM-Software` org). Source-repo `release.yml` workflows tag-route to the channel-appropriate target: bare `vX.Y.Z` → `simplemotion/sm-release`; `vX.Y.Z-preview-NNN` → `simplemotion/sm-preview`; `vX.Y.Z-develop-NNN` → `simplemotion/sm-develop`; `vX.Y.Z-testing-NNN` → `simplemotion/sm-testing`.

The receiver is `.github/workflows/sm-publish-release.yml`. It uses the SM-Binary-Bridge App to download artifacts from the source run and `gh release create` them here.

## What this repo is NOT

- **Not the installer.** Installer scripts live in `simplemotion/sm-install` and route here via `--channel release`.
- **Not the source.** Per-product source repos under `3400-0000-SM-Software` build the binaries that land here.
- **Not the build pipeline.** Builds run in the source repos; this repo only receives + publishes.

## Working rules

- **Public visibility is load-bearing.** Anything committed here is permanently public; do not paste internal docs, customer info, or credentials.
- **No "Co-Authored-By" trailers** in commits.
- **All IP assigned to SimpleMotion.Global Pty Ltd** per `ASSIGN.md`.
- **No binaries in git history.** Binaries are GitHub Release assets only. Keep the working tree small.
- **Versioning follows the SimpleMotion enterprise policy** (see appendix in `CHANGE.md`).

## Secrets

`sm-publish-release.yml` consumes two secrets/variables provisioned at this repo:

- `BRIDGE_APP_PRIVATE_KEY` (secret, b64-wrapped PEM) — SM-Binary-Bridge GitHub App private key.
- `BRIDGE_APP_CLIENT_ID` (variable) — App's client ID.

The App must be installed on `simplemotion` with `Contents:Write` on this repo, and on `3400-0000-SM-Software` with `Actions:Read` on every source repo that dispatches here.

## When in doubt, ask

Before adding new top-level files or changing `sm-publish-release.yml`'s dispatch contract — the contract is consumed by every source repo's `release.yml`.


## Promotion ladder

Builds are promoted UP the channel ladder **without rebuilding** — the
same signed artifacts move forward, so what ships is byte-identical to
what was tested:

    develop → testing → preview → release(-rcN) → GA

Tags (zero-padded 3-digit iteration; rc is 1-based):

| Stage   | Repo                      | Tag form               |
|---------|---------------------------|------------------------|
| develop | `simplemotion/sm-develop` | `v0.1.0-develop-NNN`   |
| testing | `simplemotion/sm-testing` | `v0.1.0-testing-NNN`   |
| preview | `simplemotion/sm-preview` | `v0.1.0-preview-NNN`   |
| release | `simplemotion/sm-release` | `v0.1.0-release-rcN`   |
| GA      | `simplemotion/sm-release` | `v0.1.0` (bare, latest)|

Each receiving channel repo (testing/preview/release) has a
`.github/workflows/sm-promote.yml` (manual dispatch) that copies a
release from the previous channel and re-publishes it here under the
next tag. **develop** is the entry point — source repos in
`3400-0000-SM-Software` dispatch builds there via `publish-release.yml`.
