# Tether Autolaunchd APT Repo Dev Plan

## Purpose

This repository currently works as a small flat APT feed for `tether-autolaunchd`, but it is not yet a productized Debian archive. The goal of this plan is to move it from "installable by informed operators" to "safe, repeatable, supportable distribution channel" for Debian and Kali users.

This document is intentionally execution-oriented. It defines the current state, target state, workstreams, acceptance criteria, and the first backlog to implement.

## Status Update

As of 2026-04-03, the first archive-hardening slice is complete:

- Signed `stable/` and `rc/` channels are now published in this repository.
- The archive now ships `Release`, `InRelease`, `Release.gpg`, and a public keyring.
- Installation docs now use `signed-by=` instead of `trusted=yes`.
- Release-candidate publication now normalizes Debian prerelease ordering with `~rcN`.
- The source repository release flow that builds and publishes these channels has been merged to `main`.
- This repository now validates both signed channels in CI.

The highest-priority remaining work is:

- Move off `raw.githubusercontent.com` to a more durable APT host if required.
- Convert the source package to proper Debian packaging and filesystem layout.
- Finish runtime package hardening, especially service behavior on real end-user systems.
- Add install, upgrade, purge, and rollback verification against the published archive.
- Document support policy, retention, and signing-key rotation.

## Current State

As of 2026-04-02, this repository contains:

- A `README.md` with installation instructions pointing at `raw.githubusercontent.com`.
- A flat set of committed `.deb` artifacts in the repository root.
- `Packages` and `Packages.gz` indexes committed directly in git.
- No `Release`, `InRelease`, or `Release.gpg` metadata.
- No archive signing key or `signed-by=` installation path.
- No visible CI pipeline in this repository to build, index, sign, and publish releases.

The currently published package shape also shows productization gaps:

- The install instructions require `trusted=yes`, which disables archive trust.
- Package metadata still uses placeholder maintainer identity.
- Release candidate versions use `0.1.3rcN` instead of Debian-safe prerelease ordering like `0.1.3~rcN`.
- The packaged user service contains hardcoded `/home/runner/...` environment file paths.
- Python source and `__pycache__` content are shipped directly under `/opt`.
- The repository appears to be managed as a manually updated artifact bucket rather than a generated archive output.

## Product Goals

- Install cleanly on supported Debian and Kali systems using normal APT trust semantics.
- Support predictable upgrade behavior from one published version to the next.
- Publish from CI, not by manual editing of archive metadata.
- Produce packages with Debian-appropriate filesystem layout and metadata.
- Make the runtime service robust across real end-user home directories.
- Provide enough documentation and diagnostics for supportable operation.

## Non-Goals

- Turning this repository into a multi-package general-purpose Debian mirror in the first iteration.
- Supporting every Debian derivative immediately.
- Building a full Launchpad or aptly-style archive management platform before basic correctness is in place.

## Target State

The target end state is:

- `tether-autolaunchd-apt` is a generated publish target with signed archive metadata.
- The source repository for `tether-autolaunchd` builds the package and drives publishing.
- Users install through a documented `signed-by=` flow, not `trusted=yes`.
- Stable and prerelease channels are clearly separated.
- Package builds are reproducible enough for CI verification and rollback.
- Fresh install, upgrade, and purge/reinstall paths are tested on Debian and Kali.

## Productization Principles

- Prefer generated outputs over hand-maintained archive files.
- Keep this repo minimal: published artifacts, indexes, signatures, and documentation only.
- Fail closed on trust. If signing is broken, publication should stop.
- Treat upgrade behavior as a first-class compatibility surface.
- Ship packages that match Debian conventions closely enough that operators are not surprised.

## Workstreams

### 1. Archive Trust And Delivery

Objective: make the repository consumable through normal APT trust mechanisms.

Deliverables:

- Choose a durable publish location suitable for APT metadata delivery.
- Add `Release`, `InRelease`, and `Release.gpg`.
- Create and manage a dedicated archive signing key.
- Update install instructions to use key distribution plus `signed-by=`.
- Remove `trusted=yes` from all documentation and examples.

Acceptance criteria:

- `apt-get update` succeeds without trust bypass flags.
- Repository metadata is signed and verified by APT.
- A documented key rotation path exists.

### 2. Debian Packaging Correctness

Objective: move from an ad hoc `.deb` artifact to a proper Debian package build.

Deliverables:

- Add or adopt a standard `debian/` packaging layout in the source repo.
- Build with `debhelper` and `dh-python`.
- Replace placeholder package metadata with real maintainer and project metadata.
- Normalize package install paths to Debian conventions.
- Exclude `__pycache__` and other accidental build artifacts from shipped payloads.
- Define explicit versioning rules for stable and prerelease builds.

Acceptance criteria:

- The package installs files into expected Debian/Python locations.
- Package metadata is complete and project-authentic.
- Prerelease versions sort correctly below final releases.
- Build outputs are created by a documented, repeatable command.

### 3. Runtime And Service Hardening

Objective: make the installed service behave correctly on end-user machines.

Deliverables:

- Remove hardcoded `/home/runner` assumptions from the systemd user unit.
- Use user-relative paths such as `%h` where appropriate.
- Ensure maintainer scripts are idempotent and do not overreach.
- Add a lightweight validation command for config and runtime prerequisites.
- Decide which setup steps remain manual versus auto-configured at install time.

Acceptance criteria:

- A non-`runner` user can install and start the service successfully.
- Reinstall and upgrade do not leave the service in a broken state.
- Misconfiguration failures produce actionable diagnostics.

### 4. CI/CD Publication Pipeline

Objective: remove manual archive updates and make releases traceable.

Deliverables:

- CI job to build the `.deb` from tagged source.
- CI job to generate `Packages` and `Packages.gz`.
- CI job to generate and sign `Release` metadata.
- CI job to publish the archive contents to the chosen host.
- CI artifacts for `.deb`, checksums, and build metadata.

Acceptance criteria:

- A tagged release can be built and published without local manual steps.
- Published metadata matches the uploaded package set.
- Failed signing or publication blocks release completion.

### 5. Verification Matrix

Objective: verify the archive and package against the actual supported operating environments.

Deliverables:

- Clean install tests for at least one Debian target and one Kali target.
- Upgrade tests from the previous stable release.
- Purge and reinstall tests.
- Service startup verification under `systemd --user`.
- A smoke test covering the package's `doctor` or runtime validation path.

Acceptance criteria:

- CI or release verification proves install, update, and startup on supported environments.
- Regressions in archive metadata or dependency declaration are caught before release.

### 6. Documentation And Supportability

Objective: make the package operable by someone other than the publisher.

Deliverables:

- Installation instructions for stable and prerelease channels.
- Upgrade and rollback notes.
- Description of supported operating systems and architectures.
- Changelog or release notes expectations.
- Troubleshooting section for common service startup and config errors.

Acceptance criteria:

- A fresh operator can install and start the package from docs alone.
- Release notes are sufficient to understand upgrade risk.

## Release Channel Model

Recommended release model:

- `stable`: final releases only.
- `rc`: release candidates only.

Versioning rules:

- Final release example: `0.1.3`
- Release candidate example: `0.1.3~rc2`
- Post-release patch example: `0.1.4`

Why this matters:

- Debian version ordering treats `0.1.3~rc2` as lower than `0.1.3`.
- The current `0.1.3rc2` form sorts higher than `0.1.3`, which will break normal upgrade expectations.

## Repository Architecture Recommendation

Recommended ownership split:

- `tether-autolaunchd` source repo owns application code, Debian packaging, tests, and release tags.
- `tether-autolaunchd-apt` owns published archive outputs and operator-facing archive docs.

This repository should ideally contain:

- Published `.deb` files that are still in the retention window.
- `Packages`, `Packages.gz`, `Release`, `InRelease`, and `Release.gpg`.
- Public archive key material or instructions for retrieving it.
- Minimal documentation for subscribing to the repo.

This repository should ideally not contain:

- Hand-edited package metadata.
- Manually assembled archive indexes.
- Source-of-truth packaging logic.

## Phased Execution Plan

### Phase 0: Stop The Most Dangerous Gaps

Scope:

- Fix versioning policy.
- Replace placeholder metadata.
- Remove hardcoded user-home assumptions from the runtime package.
- Decide archive host and signing approach.

Exit gate:

- The next published package can become the first candidate for a signed archive release.

### Phase 1: Build Proper Debian Packaging

Scope:

- Establish standard Debian packaging in the source repo.
- Normalize filesystem layout and dependencies.
- Eliminate accidental payload artifacts.

Exit gate:

- Package builds reproducibly from source through a documented command or CI job.

### Phase 2: Introduce Signed Archive Publishing

Scope:

- Generate `Release` metadata.
- Sign archive metadata.
- Publish via CI.
- Replace `trusted=yes` instructions.

Exit gate:

- A clean host installs from a signed archive using `signed-by=`.

### Phase 3: Add Verification And Upgrade Gates

Scope:

- Fresh install tests.
- Upgrade tests.
- Service startup tests.
- Rollback notes.

Exit gate:

- Release publication is blocked unless install and upgrade checks pass.

### Phase 4: Reach GA Operational Readiness

Scope:

- Finalize docs.
- Define support matrix.
- Define retention and rollback policy.
- Define signing key custody and rotation procedures.

Exit gate:

- The archive is supportable without relying on publisher memory.

## First Sprint Backlog

These items should be executed first because they remove the largest correctness and trust risks:

1. Create `stable` and `rc` release rules and switch prerelease numbering to `~rcN`.
2. Replace placeholder maintainer identity and package metadata in the source package.
3. Fix the systemd user service so it does not depend on `/home/runner/...`.
4. Decide the archive host and signing-key custody model.
5. Add CI to generate archive indexes instead of committing them manually.
6. Add signed `Release` metadata and update install docs to use `signed-by=`.
7. Run a clean Debian and Kali install test against the published archive.

## Definition Of Done

This repository is productized when all of the following are true:

- Users can install with standard APT trust, without `trusted=yes`.
- The archive is signed and the signing flow is automated.
- Fresh install, upgrade, and reinstall paths are verified on supported targets.
- The package layout and metadata follow Debian norms closely enough to avoid operator surprise.
- Publication is CI-driven and reproducible.
- Documentation covers installation, upgrade, rollback, and troubleshooting.

## Open Decisions

The following choices should be made explicitly before implementation spreads:

- Publish host: GitHub Pages, object storage, or another static host.
- Channel model: separate repo paths versus separate suites.
- Artifact retention: keep all historical `.deb` files or only a bounded window.
- Signing key custody: local hardware-backed key, CI-injected subkey, or another model.
- Supported targets: exact Debian and Kali versions for the first supported matrix.

## Suggested Next Change Set

The next implementation pass should focus on the minimum viable productization slice:

1. Add signed archive metadata.
2. Fix install instructions.
3. Fix version ordering.
4. Fix service path assumptions.
5. Wire publication through CI.

That slice converts the repository from an unsigned convenience feed into a basic but real Debian archive.
