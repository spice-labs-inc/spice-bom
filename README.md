# spice-bom

The canonical dependency-version BOM for Spice Labs JVM projects
(`spice-labs-cli`, `goatrodeo`, `sassafras`, `allspice`). Import it to inherit a
single, converged set of dependency versions.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>io.spicelabs</groupId>
      <artifactId>spice-bom</artifactId>
      <version>1.0.0-SNAPSHOT</version> <!-- pin a released version for production -->
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## Where it's published

| Channel | Version kind | How |
|---|---|---|
| GitHub Packages (`maven.pkg.github.com/spice-labs-inc/spice-bom`) | `-SNAPSHOT` (every push to `main`) and releases (`v<x.y.z>`) | `publish-bom.yml` (on tag) + `snapshot.yml` (on push) |
| Maven Central | releases only (`v<x.y.z>`) | `publish-bom.yml` |

GitHub Packages requires auth even for public repos, so every consumer needs a
`~/.m2/settings.xml` carrying a `github` server entry with a `read:packages`-scoped
token (see [RELEASING.md](RELEASING.md)).

## Dependency order

`spice-plugin-api` (in `spice-labs-inc/spice-labs-cli`) ← **spice-bom**
(references a released plugin-api) ← consumers. Release lower layers first.
See [RELEASING.md](RELEASING.md).

## Version-bump policy

The BOM's contract is the set of dependencies it manages. Relative to the last
`v*` release, bump `pom.xml`'s `<version>`:

- **patch** — a managed version changed (e.g. `logback` 1.5.18 → 1.5.19, or the
  pinned `spice-plugin-api` moved to a new release)
- **minor** — a managed artifact was added
- **major** — a managed artifact was removed
