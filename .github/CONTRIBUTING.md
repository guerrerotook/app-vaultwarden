# Contributing

When contributing to this repository, please first discuss the change you wish
to make via issue, email, or any other method with the owners of this repository
before making a change.

Please note we have a code of conduct, please follow it in all your interactions
with the project.

## Issues and feature requests

You've found a bug in the source code, a mistake in the documentation or maybe
you'd like a new feature? You can help us by submitting an issue to our
[GitHub Repository][github]. Before you create an issue, make sure you search
the archive, maybe your question was already answered.

Even better: You could submit a pull request with a fix / new feature!

## Pull request process

1. Search our repository for open or closed [pull requests][prs] that relates
   to your submission. You don't want to duplicate effort.

1. You may merge the pull request in once you have the sign-off of two other
   developers, or if you do not have permission to do that, you may request
   the second reviewer to merge it for you.

## Releasing

Publishing a new version is fully driven by GitHub releases:

1. Create a GitHub release tagged `vX.Y.Z` (for example `v0.28.0`). Always use
   this scheme: the container tag is the git tag with the leading `v` stripped,
   and Home Assistant compares that exact string with the `version` in the app
   catalog.

1. The `Deploy` workflow builds and pushes
   `ghcr.io/<owner>/bitwarden/<arch>:X.Y.Z`, then creates the multi-arch
   manifests `ghcr.io/<owner>/bitwarden:X.Y.Z` and
   `ghcr.io/<owner>/bitwarden:stable`.

1. Finally it dispatches an `update` event to the app catalog repository
   (`<owner>/repository`), whose repository updater bumps `bitwarden/config.yaml`
   to the new version. This requires the `DISPATCH_TOKEN` secret in this
   repository to be a token with write access to the catalog repository, and the
   `UPDATER_TOKEN` secret to be configured there.

The `version` key in `vaultwarden/config.yaml` stays at `dev`; the real version
is injected during the release build. The container packages on GHCR must be
public, otherwise Home Assistant cannot pull them.

### Keeping the app catalog in sync

For the dispatch to actually update `bitwarden/config.yaml` in the catalog
repository, that repository needs:

1. GitHub Actions enabled (Settings → Actions → General), so that
   `.github/workflows/repository-updater.yaml` on `master` is registered and
   can receive the `repository_dispatch` event.

1. An `UPDATER_TOKEN` secret with write access, used by the repository updater
   to commit the version bump back to `master`.

1. A `DISCORD_WEBHOOK` secret, or the `announce` job removed, so a missing
   webhook does not fail the run.

If enabling Actions in the catalog repository is not an option, this repository
can write the update itself instead. Set the repository variable
`SYNC_CATALOG_DIRECTLY` to `true`: the `sync-catalog` job then copies
`vaultwarden/config.yaml` (plus `DOCS.md`, `icon.png`, `logo.png` and
`translations/`) into `bitwarden/` in the catalog repository, sets `version` and
`image`, and pushes the change using the `DISPATCH_TOKEN` secret. The
`publish-stable` dispatch job is skipped while this variable is set, so only one
writer ever updates the catalog.

[github]: https://github.com/hassio-addons/app-vaultwarden/issues
[prs]: https://github.com/hassio-addons/app-vaultwarden/pulls
