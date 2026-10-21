# DependencyLens

**One read-only upgrade report for Python and Node.js projects.**

DependencyLens inspects direct dependencies, checks public release metadata, and
turns the results into terminal, Markdown, or JSON reports. It does not install
dependencies, run project code, or edit manifests.

Version **0.1.0** is an initial implementation. It has not been published to a
package registry or established a user base. The distribution name
`dependency-lens-cli` is proposed; availability has not been checked.

## Quick start

Requires Python 3.11 or newer. From the extracted project directory:

```sh
python -m venv .venv
# macOS / Linux:
. .venv/bin/activate
# Windows PowerShell instead: .venv\Scripts\Activate.ps1
python -m pip install .
dependency-lens --help
```

Try the bundled offline demo (all demo package names and releases are synthetic):

```sh
dependency-lens examples/demo --metadata examples/metadata.json --include-dev
```

Example result, abbreviated:

```text
OUTDATED    npm:example-widget  1.2.0 → 2.0.0  major
CURRENT     npm:example-utils  2.0.0 → 2.0.0
OUTDATED    pypi:example-client  1.0.0 → 1.1.0  minor
UNPINNED    pypi:example-api  (unpinned) → 2.5.0
```

Scan a real project using public registries:

```sh
dependency-lens /path/to/project
dependency-lens /path/to/monorepo --recursive --include-dev
dependency-lens /path/to/project --format markdown --output upgrades.md
dependency-lens /path/to/project --format json --output upgrades.json
dependency-lens /path/to/project --fail-on-outdated
```

## Supported inputs

| Input | Behavior |
| --- | --- |
| `package.json` | Runtime and optional dependencies; add `--include-dev` for dev and peer dependencies |
| Adjacent `package-lock.json` | Uses direct installed versions from npm lockfile v1, v2, or v3 |
| `requirements.txt` | PEP 508 requirements, extras, markers, pins, ranges and inline comments |
| `pyproject.toml` | PEP 621 project dependencies; add `--include-dev` for optional and PEP 735 development groups |

Pass a directory or a supported manifest. A lockfile alone cannot identify all
direct dependencies, so scan its adjacent `package.json`. Recursive discovery
skips `node_modules`, `.git`, virtual environments, build output and directory
symlinks. Requirements includes/options and dependency-group includes generate
warnings rather than being expanded. Legacy Poetry tables are not supported.

## Reading reports

| Status | Meaning |
| --- | --- |
| `outdated` | The pinned or locked version precedes the target release |
| `current` | The versions have equal precedence |
| `ahead` | The pin or lock version is newer than the selected registry target |
| `unpinned` | There is no exact or locked version to compare; latest is shown for review |
| `unsupported` | Local path, URL, Git, workspace or npm alias needs manual review |
| `error` | Registry lookup failed or offline metadata is missing |
| `unknown` | The versions cannot be interpreted |

Python versions follow PEP 440. npm versions follow SemVer precedence, including
prerelease ordering and ignoring build metadata. Upgrade labels describe changes
to numeric version components; they do **not** guarantee API compatibility,
especially for 0.x releases. Python epochs and release transitions also need
manual review.

The latest target is npm's `latest` dist-tag or the highest stable, non-yanked
PyPI release with available distribution files. DependencyLens does not resolve
the newest release allowed by a version range, assess runtime compatibility,
evaluate Python environment markers, or inspect transitive dependencies.
Markers and original declarations remain in JSON reports. A locked npm version
can be stale relative to its manifest; the report does not validate lockfile
consistency. Python lockfiles, npm overrides, pnpm and Yarn lockfiles are not
supported in this release.

This is an upgrade inventory, not a vulnerability scanner or automatic updater.
Review release notes, constraints and runtime requirements before upgrading.

## CI and automation

Exit codes:

- **0:** scan completed, including unsupported or unpinned items needing review.
- **1:** `--fail-on-outdated` was supplied and outdated dependencies were found.
- **2:** invalid input, manifest warning, registry failure, or uncomparable version.

Errors take precedence over exit 1. `--fail-on-outdated` checks known outdated
versions only; it does not make unpinned or unsupported dependencies a CI failure.
JSON output includes `schema_version`, `summary`, `warnings` and `dependencies`.
Default concurrency is 8 requests, bounded by `--workers 1..32`. Set a per-request
timeout with `--timeout 10`. Each distinct registry/package pair is fetched once
per invocation; there is no disk cache or retry loop.

An example GitHub Actions workflow for this project's tests is in
`.github/workflows/tests.yml`.

## Offline mode and privacy

Online mode sends each supported dependency name to its public registry over
HTTPS. Avoid online mode for private package names. It uses no account tokens and
sends no manifest files. Network failures are reported per dependency.

Offline mode makes zero network requests. Supply a JSON object mapping
`ecosystem:name` to a release string:

```json
{"pypi:requests": "2.32.0", "npm:@example/widget": "1.2.0"}
```

These are illustrative values, not claims about current releases. Offline
reports explicitly label fixture metadata. In npm metadata, the response size is
limited to 20 MB; unusually large packages may require offline metadata.

## Development

```sh
python -m pip install -e .
python -m unittest discover -s tests -v
```

The test suite uses the standard library and mock registry responses, so tests
do not depend on public network access. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Release checklist

1. Run tests on the supported Python versions; the included CI defines a matrix.
2. Choose and verify the repository and package distribution names.
3. Create a public repository, review the MIT license and configure maintainers.
4. Build and inspect wheel/source distributions, then publish from your account.
5. Track actual adoption through registry downloads, dependents, and GitHub activity.

The source archive alone does not publish a repository or package. No adoption,
download, dependent-repository, or funding-program eligibility claims are made.

## References

- [PyPI JSON API](https://docs.pypi.org/api/json/)
- [npm package metadata](https://github.com/npm/registry/blob/main/docs/responses/package-metadata.md)
- [Semantic Versioning](https://semver.org/)
- [Python version specification](https://packaging.python.org/en/latest/specifications/version-specifiers/)

## License

MIT. See [LICENSE](LICENSE).
