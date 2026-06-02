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

Builds climb the ladder **without rebuilding** — the same signed
artifacts move forward. The two **public** channels (`preview`,
`release`) only ever show **real releases**; all candidate churn stays
on the **internal** channels, so the public never sees a non-final build.

| Stage   | Repo                      | Visibility | Holds                            | Tag form                          |
|---------|---------------------------|------------|----------------------------------|-----------------------------------|
| develop | `simplemotion/sm-develop` | internal   | earliest dev builds              | `v0.1.0-develop-N`                |
| testing | `simplemotion/sm-testing` | internal   | test builds + release candidates | `v0.1.0-testing-N`, `v0.1.0-rcN` |
| preview | `simplemotion/sm-preview` | public     | public preview / beta releases   | `v0.1.0-preview-N`                |
| release | `simplemotion/sm-release` | public     | **GA only**                      | `v0.1.0` (bare, Latest)           |

Flow: `develop → testing (→ rcN) → preview → release (GA)`. Release
candidates (`v0.1.0-rcN`) are vetted **internally on testing**; once
blessed they promote to a public `preview` release and finally to the
bare **`v0.1.0` GA** on `release` (always marked Latest).

**No GitHub prerelease flag is used anywhere** — identity is the repo
(and its visibility), and the public repos simply never receive a
non-preview / non-GA tag. `sm-release` accepts **only bare `vX.Y.Z`
(GA)** promotion targets; cut rc/preview builds on `testing`/`preview`.

Each receiving channel repo has `.github/workflows/sm-promote.yml`
(manual dispatch) that copies a release from the previous channel into
this one under the next tag. `develop` is the entry point — source
repos in `3400-0000-SM-Software` dispatch builds there.
