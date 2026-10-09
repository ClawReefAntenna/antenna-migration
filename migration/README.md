# Antenna migration kit — v1.6.9

Bring your existing identity and permissions from the relay into native
[Antenna for OpenClaw](https://github.com/ClawReefAntenna/antenna-openclaw), which handles everyday agent messaging.
This standalone kit prepares the conversion, retires the old relay and keeps
private recovery copies. Install the native plugin and companion separately.

[Download the kit](https://github.com/ClawReefAntenna/antenna-migration/releases/download/v1.6.9/antenna-migration-1.6.9.tgz), verify its
[SHA-256 checksum](https://github.com/ClawReefAntenna/antenna-migration/releases/download/v1.6.9/SHA256SUMS), and extract it into its own directory. Run the
commands below from the extracted kit root. Requires Node supported by OpenClaw
and a Linux host for the stopped-writer cutover.

## Supported transition

Migrate relay installations running **v1.6.3–v1.6.7** to native schema-2/plugin-v2.
The kit carries forward the global inbox and trusted-peer settings in v1.6.3–v1.6.5,
and the session-policy settings in v1.6.6–v1.6.7. Remote peers need Ed25519 signing
keys; existing signed pairings retain their pins. Unknown policy formats and
unsigned remote peers require explicit review or signed pairing first.

A fresh relay installation also has a local self-peer record without signing
keys. The kit preserves that identity and reports it in `omittedUnsignedSelf`,
but does not grant it native inbound access or select it for native outbound
transport. If you use self-messaging, configure its signing identity separately.
Already signed self-peers keep their normal permissions.

Native v1.6.8 and later use the normal update path, not relay migration.

## Rehearse on an isolated copy

Choose existing receiving conversations and confirm destination names with your
peers. Save a selection JSON like this, using your own destinations, paths and
paired peer names:

```json
{
  "mcs": "dumb",
  "destinations": {"research": "agent:research:main"},
  "inboxFile": "/absolute/new-state/inbox.json",
  "replayFile": "/absolute/new-state/replay.json",
  "outbound": {
    "lab": {
      "profile": "antenna-plugin-v2",
      "default_target": "research"
    }
  }
}
```

Include every receiving conversation allowed by the old policy. The new inbox
and replay paths must be distinct and absent. `LEGACY_ROOT` is the old Antenna
installation, `HOST_JSON` the intended resolved OpenClaw configuration, and
`SELECTION_JSON` your saved selection. Choose a new directory for `NEW_STAGE`.
Keep the isolated rehearsal's host and state paths separate from live files.

A dry-run prints only the report. Supplying a new staging directory writes private
preimages, disabled native policy, peer/config candidates and a hash-bound plan.

```sh
node migration/cutover.mjs stage LEGACY_ROOT HOST_JSON SELECTION_JSON
node migration/cutover.mjs stage LEGACY_ROOT HOST_JSON SELECTION_JSON NEW_STAGE APPROVALS_JSON
```

The optional approvals file defaults to `exec-approvals.json` beside HOST_JSON
when present. Supply its actual path if the host uses another state directory.
All inputs must be bounded (8 MiB per file), owned regular files without symlinks
or hard links. Host JSON must already be resolved; use OpenClaw's configuration
commands for JSON5/includes. The token must also be private and at most 16 KiB.
No credential is printed. Stage directories are 0700, files 0600; they contain
credentials and are recovery material, not distributable kit contents.

## Cut over while stopped

Stop the selected gateway and **all** Antenna writers first. This interrupts host
service; schedule it deliberately. Review the staged changes, then:

```sh
node migration/cutover.mjs apply NEW_STAGE --writers-stopped
```

Keep the gateway and all Antenna writers stopped throughout cutover.
`--writers-stopped` records that prerequisite; the kit also checks for running
local writers. It leaves services and Antenna activation under your control.

The kit retires relay entries it can identify as belonging to this installation,
including their relay-specific hook and shell approvals. Custom approvals and
other agents remain. If ownership is ambiguous, it stops for manual review.
Broad hooks, hook tokens, session visibility,
model registrations, signing identities, lists and group grants stay unchanged.
Legacy pending, approved-but-unsent and failed inbox items remain untouched.
Existing HTTP settings are preserved; HTTPS connections stay HTTPS.

No temporary-file sweep occurs. To remove confirmed obsolete relay input, add
`obsoleteTemps` entries to the selection with exact `path`, `sha256` and
`obsolete: true`. Only private, owned, single-link historical temporary filenames
under `/tmp` are eligible. Staging retains a private recovery copy. Cleanup occurs
only after stopped-host retirement; inbox, replay, keys and recovery paths do not
qualify. If a file is still unresolved, leave it out.

## Interrupted cutover and recovery

The kit stops if source or staged files have changed. Keep writers stopped.
If cutover is interrupted, leave Antenna disabled and preserve `NEW_STAGE` and its
`RECOVERY.txt`. Follow those recovery instructions; resuming the same plan skips
completed changes.
Do not restore the old relay or broad shell approvals from preimages. To abandon
cutover, leave Antenna disabled and recover unrelated settings manually from the
private copies. New holds and replay data must not be replaced by old snapshots.
This is operator recovery, not seamless downgrade or automatic rollback.

The native plugin/companion assets still need installation separately. Then verify
existing destinations, operator credential separation and scanner selection,
explicitly enable/restart, and live-probe admission before advertising migration.
The guide describes optional general-hook token rotation; it is not mandatory.
Coordinate the peer-profile switch before resuming message delivery.

## Check the prepared installation

Run Doctor against the intended host configuration and matching companion:

```sh
node plugin/migration-check.mjs doctor HOST_JSON COMPANION_ROOT
```

## Advanced: export a schema-1 policy

Experimental schema-1 combined-smart-v1 policy can also be exported as schema 2:

```sh
node migration/cli.mjs HOST_JSON NEW_POLICY_JSON
```

Schema export never applies host settings or changes inbox data. Explicit per-peer
and per-destination approval bits remain independent of MCS. Combined legacy
Smart becomes Both. The artifact manifest binds version, source and file hashes.
