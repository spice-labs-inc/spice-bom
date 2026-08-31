# Releasing spice-bom

This repo publishes a single artifact: `io.spicelabs:spice-bom` (a `pom`-packaged
BOM). It ships on its own GitHub Release, tagged `v<x.y.z>`, independently of
every consumer.

SNAPSHOT versions are published to GitHub Packages on every push to `main` (see
`.github/workflows/snapshot.yml`); releases go to GitHub Packages **and** Maven
Central (see `.github/workflows/publish-bom.yml`).

| Channel | Trigger | Workflow |
|---|---|---|
| GitHub Packages (SNAPSHOT) | push to `main` | `snapshot.yml` |
| GitHub Packages (release) | GitHub Release `v<x.y.z>` | `publish-bom.yml` |
| Maven Central (release) | GitHub Release `v<x.y.z>` | `publish-bom.yml` |

In-repo, the version is a `-SNAPSHOT` (e.g. `1.0.0-SNAPSHOT`). A release takes
its version from the release tag and publishes a real (non-SNAPSHOT) artifact.

Pushing a bare `v<x.y.z>` git tag does **not** publish anything — the workflow
runs on `release: [published]`, so the release itself is what triggers it.

## Dependency order

`spice-plugin-api` (source: `spice-labs-inc/spice-plugin-api`) ← **spice-bom**
(references a released plugin-api) ← `spice-labs-cli` (imports a released BOM).
Release lower layers first. Each release workflow **verifies** its inputs are
already published (`dependency:get`) and fails fast otherwise.

1. **plugin-api** — publish a GitHub Release tagged `v<x.y.z>` in
   `spice-plugin-api` → its `publish.yml` publishes it.
2. **BOM** — set `spice-plugin-api.version` here to the released `<x.y.z>` (a BOM
   *version change* → BOM **patch** bump; see below), then publish a GitHub
   Release tagged `v<x.y.z>` → `publish-bom.yml` verifies the pinned plugin-api
   is published, then publishes.
3. **CLI** — publish a GitHub Release tagged `v<x.y.z>` → `publish.yml` verifies
   that `spice-bom` is published, pins the CLI to it, and deploys.

## BOM version-bump policy

The BOM's contract is the set of dependencies it manages. Relative to the last
`v*` release, bump `pom.xml`'s `<version>`:

- **patch** — a managed version changed (e.g. `logback` 1.5.18 → 1.5.19, or the
  pinned `spice-plugin-api` moved to a new release)
- **minor** — a managed artifact was added
- **major** — a managed artifact was removed

## Cutting a release

1. Ensure the managed versions are what you want and the `<version>` reflects the
   required bump.
2. Publish a GitHub Release tagged `v<x.y.z>` against the tip of `main` →
   `publish-bom.yml` verifies the pinned plugin-api is published, then publishes
   the BOM to GitHub Packages and Maven Central:

   ```bash
   gh release create v<x.y.z> --target main --generate-notes
   ```

After a release, bump the in-repo `-SNAPSHOT` to the next intended version.
