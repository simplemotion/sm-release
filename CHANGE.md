<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/simplemotion/.github/main/assets/banners/SM-White.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/simplemotion/.github/main/assets/banners/SM-Black.svg">
    <img alt="SimpleMotion" src="https://raw.githubusercontent.com/simplemotion/.github/main/assets/banners/SM-Black.svg" width="800">
  </picture>
</p>

<p align="center">
  <em>Engineered for Architecture, Entertainment, Industry and Manufacturing.</em>
</p>

# CHANGE.md

Changelog for this repo (`simplemotion/sm-release`).

Versioning follows the SimpleMotion enterprise policy — `vX.Y.Z` releases, `vX.Y.Z-preview-NNN` candidates, `vX.Y.Z-cm-NNN` dev builds. Only releases and RCs are recorded here. **Full policy is in the appendix at the end of this file.**

This repo hosts release-asset binaries for the **release** channel. Releases land here via repository_dispatch from per-product source repos.

---

## Changelog

| Version | Hash | Date | Author | Notes |
|---------|------|------|--------|-------|
| (auto) | &mdash; | 2026-08-28 01:35 UTC | Greg Gowans | **Adopt the shared public-safe PR gate, so this repo's PRs are finally checked.** It is public with no workflows at all, and its housekeeping PRs - appendix reconciliation, canonical ASSIGN.md text, changelog format - have all gone unchecked. It could not use the canonical stub because GitHub refuses a public repo calling an internal reusable workflow and fails it at startup with zero jobs, reporting nothing at all. The checks now live once in simplemotion/.github, which is itself public, and this is the identical caller stub every public repo carries. Writing a bespoke workflow here was the obvious move and was rejected: two inlined copies already existed and had diverged, and a third would have made the problem worse rather than better. |
| (auto) | &mdash; | 2026-08-17 | Greg Gowans | **Remove `sm-publish-release.yml` — dead since `sm-ci` moved its dispatch target.** This workflow received the `publish-binary` dispatch and created the release here. `sm-ci` now dispatches to `3400-0000-SM-Software/3400-9993-SM-Publish`, which writes the release into this repo cross-repo with a scoped SM-Binary-Bridge token, so nothing reaches this file any more. **Verified dead before deleting, not assumed:** an `sm-ci` run was re-run after the target changed, its dispatch job succeeded, and the resulting `repository_dispatch` run on the publisher completed successfully — the release was written here by the new path while this workflow sat idle. That was the last of four byte-identical copies of this logic, and on the two PUBLIC channels those copies could never have been deduplicated into a shared caller anyway, since a public repo cannot call an internal reusable workflow. With this gone the repo holds no workflows at all: it is an artifact store, which is the whole point of the arrangement. |
| — | — | 2026-08-17 01:58 UTC | Greg Gowans | Retired two obsolete workflows: `sm-promote.yml` (promotion now runs from `3400-0000-SM-Software/3400-9993-SM-Publish`, "Promote up the channel ladder", cross-repo via scoped SM-Binary-Bridge tokens) and `sm-ci.yml` (this repo holds no source code, and as a **public** repo it can never call the internal `simplemotion/sm-ci` reusable workflow — every run since 2026-07-11 was a zero-job `startup_failure`). `sm-publish-release.yml` is untouched and still live. Channel repos are becoming pure artifact stores. |
| _(no releases yet)_ | | | | |

The first release tag will be `v0.1.0`.

---

## Versioning

This repo follows the SimpleMotion enterprise versioning policy. It is kept in
one canonical place rather than copied into every repo, so an amendment lands
once instead of being reconciled across the fleet:

<https://github.com/simplemotion/.github/blob/main/ANNEXE.md>
