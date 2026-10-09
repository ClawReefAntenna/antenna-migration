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

## Operator archive contents

The operator archive contains the two current guides, license, dependency provenance,
three command entry points and their required modules/resources: 24 files total.
Regression tests, this maintainer guide, historical `plugin/MIGRATION.md`, and
build/verification tools stay in the source repository and full-source archive.

`SOURCE.json` separates shipped `sharedFiles` from source-only
`developmentSharedFiles`. The builder checks both sets of pins. The verifier
checks the pristine extracted inventory and dependency closure, refuses development
files in the operator archive, then copies only the five regression fixtures from
the matching clean source revision into a temporary extraction. Tests run against
shipped implementation bytes; the downloadable archive is never modified.

Run migration verification apart from other same-user Antenna CLI/gateway fixtures:
its stopped-writer check deliberately sees those processes. Do not bypass it.

## Regression checks

Run these from the source checkout root when maintaining the app. They
are development checks, not operator cutover steps.

```sh
node migration/schema-test.mjs
node migration/cutover-test.mjs
node migration/legacy-compat-test.mjs
node plugin/tests/config-write.mjs
node plugin/tests/migration-doctor.mjs
```

## Cutover implementation and attribution

`--writers-stopped` records the operator's stopped-writer prerequisite. A Linux
`/proc` check also refuses detected gateway/relay/native writers. It cannot observe
other PID namespaces, remote hosts or a future process start; keep the host
quiescent throughout. Other systems use the manual workflow. The app never
stops/starts services, invokes old setup, enables the plugin, rotates credentials,
sends messages or automatically rolls back.

Owned relay roster entries must have the configured relay ID and exactly the old
root's `agent` workspace. The kit removes only their agent-specific hook allow
entry and known Bash/echo/jq/cat approvals; custom approvals and other agents
remain. Ambiguous agent ownership or direct relay hook mappings require manual
review, not a wider automatic deletion.

Every target must match its original or planned hash before any write. Changes
to other inputs, newly appeared native state, or changed staged bytes stop the
operation. Each replacement is validated JSON, fsynced, privately backed up and
atomically renamed with a final source comparison. Cooperating writers use locks;
these checks are not an atomic CAS against unrelated programs. Keep writers stopped.

For operator recovery steps, use the [migration guide](../migration/README.md#interrupted-cutover-and-recovery).
