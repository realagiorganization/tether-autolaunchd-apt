# Dev Plan

## Goal

Turn this repository into a reliable publish target for `tether-autolaunchd`
Debian and Kali packages instead of a hand-maintained artifact bucket.

## Current direction

- keep this repository generated, not hand-authored
- publish a signed flat APT archive from the source repo release workflow
- remove malformed prerelease versions that sort above their future stable
  equivalents
- validate repository metadata and signatures on every push

## Work streams

1. Archive trust
- publish `Release`, `InRelease`, `Release.gpg`, and an exported public keyring
- remove `trusted=yes` from install instructions
- fail releases when signing secrets are unavailable

2. Version hygiene
- publish Debian prereleases as `~rcN`, `~bN`, or `~aN`
- prune legacy malformed prerelease artifacts such as `0.1.3rc2`

3. Publish automation
- build the `.deb` in the source repo
- regenerate repository metadata on every release
- push only generated outputs into this repository

4. Validation
- verify archive signatures in CI
- verify `apt-get update` against a local `file://` source
- reject malformed prerelease versions in `Packages`

## Definition of done

- `apt-get update` works with `signed-by=...` and without `trusted=yes`
- the archive contains signed metadata and an exported keyring
- prerelease ordering no longer blocks later stable releases
- repository pushes are validated automatically
