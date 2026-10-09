# Build the migration kit

This repository is the authoritative source for the app-specific implementation,
tests and documentation. Its build does not need the OpenClaw source checkout.

```sh
python3 packaging/build.py --output /absolute/new-build-directory
python3 packaging/verify.py /absolute/new-build-directory
```

Commit before a release build. The literal allowlist produces a self-contained
archive, manifest and full-source archive; nothing is installed or published.
Use the app archive for downloads, not the full source archive.

`SOURCE.json` records the upstream revision and SHA-256 pins for bundled shared
helpers. Those helpers remain authoritative in antenna-openclaw. To update them,
copy the selected upstream files, update provenance and hashes, then run the
extracted-package verifier. The builder refuses unrecorded shared-file changes.
App-specific changes are maintained here, not copied back to OpenClaw.

Publish the matching manifest and SHA256SUMS beside the archive. Migration compatibility and recovery qualification are documented in the
migration guide.
