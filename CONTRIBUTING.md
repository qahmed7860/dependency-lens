# Contributing to DependencyLens

Start with a small reproducible example. For parser issues, include a minimal
manifest without credentials, private package names, or unrelated project data.
For comparison issues, include the ecosystem and two version strings.

Install with `python -m pip install -e .` and run
`python -m unittest discover -s tests -v`.

Keep scans read-only, make unsupported syntax visible, and preserve deterministic
report ordering. Add meaningful regression tests for parsing, comparison and
failure behavior. Mock registry calls in tests; do not rely on current releases.
Explain user-visible behavior and any supported-input limitations in your change.

Potential next contributions: Python lockfiles, package-manager workspace
resolution, compatibility-aware targets, cache support, and richer CI policies.
These are future ideas, not implemented features.
