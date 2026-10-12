# simplemotion/sm-release-external

**Release channel** binary-distribution surface for SimpleMotion products.

- **Audience:** All consumers — stable production builds.
- **Stability:** Stable. Promoted from `preview` after validation. Owns `releases/latest` of this repo.

## What's here

GitHub Release assets — one release per product version, three files per platform:

```
<package>-<host-triple>                  ← binary
<package>-<host-triple>.sha256           ← SHA256 sidecar
<package>-<host-triple>.sigstore.jsonl   ← sigstore build-provenance bundle
```

where `<host-triple>` is `aarch64-apple-darwin` (Apple silicon; Intel Macs are not supported since 2026-10-08), `aarch64-unknown-linux-gnu`, `x86_64-unknown-linux-gnu`, `aarch64-pc-windows-msvc`, or `x86_64-pc-windows-msvc`.

## How to install

Use the SimpleMotion installer and select this channel with `--channel release`. Executive and employees cohorts use the internal installer copy. Freelance and customers cohorts use the external installer copy. `release` is the default channel, so you can omit `--channel release`.

The [`simplemotion/sm-install-int` README](https://github.com/simplemotion/sm-install-int#readme) gives the install commands for each copy. This repo does not copy those commands, because the installer URL can change.

The installer (in [`simplemotion/sm-install-int`](https://github.com/simplemotion/sm-install-int)) handles platform detection, SHA verification, sigstore attestation verification, and install. Do not download binaries directly from this repo, except to do a check of them out of band.

## Verification

See [`simplemotion/sm-install-int/SECURE.md`](https://github.com/simplemotion/sm-install-int/blob/main/SECURE.md) for the consumer verification recipe (`gh attestation verify --bundle …`).

## How releases get here

Releases arrive automatically via `repository_dispatch` from per-product source repos under `3400-0000-SM-Software`. The source-repo `release.yml` tag-routes by suffix. The package path selects the column:

| Tag pattern | External-path repo | Internal-path repo |
|---|---|---|
| `vX.Y.Z` | `simplemotion/sm-release-external` | `simplemotion/sm-release-internal` |
| `vX.Y.Z-preview-NNN` | `simplemotion/sm-preview-external` | `simplemotion/sm-preview-internal` |
| `vX.Y.Z-develop-NNN` | `simplemotion/sm-develop-external` | `simplemotion/sm-develop-internal` |
| `vX.Y.Z-testing-NNN` | `simplemotion/sm-testing-external` | `simplemotion/sm-testing-internal` |

Only external-path packages land here.

The receiver workflow here (`sm-publish-release.yml`) downloads the source-run artifacts and creates a GitHub Release.

## Reporting issues

- **Distribution / install bugs:** open an issue on [`simplemotion/sm-install-int`](https://github.com/simplemotion/sm-install-int).
- **Product bugs:** per-product source repo under `3400-0000-SM-Software` (internal).
- **Security:** email **security@simplemotion.com**.
