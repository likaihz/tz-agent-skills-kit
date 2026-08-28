# Setup Protocol

## Contents

1. Discovery
2. Dependency resolution
3. Plan
4. Runtime setup
5. Instance setup
6. Receipts and verification
7. Repair, update, and removal
8. Safety

## Discovery

Resolve the target in this order:

1. Current Agent Skill inventory or explicitly loaded Skill.
2. `catalog.yaml` in the current repository.
3. A direct link from the target `SKILL.md` to its setup reference.
4. A user-provided local path.

If multiple versions or paths match, show their name, version, and path and ask the user to select one. Record the selected target, dependencies, version, path, merged runtime requirements, and setup reference only in the setup session or local diagnostics.

Validate catalog schema, name/path uniqueness, dependency existence, version ranges, acyclic ordering, `runtime_requires`, probes, and referenced file existence before acting.

## Dependency resolution

1. Expand `depends_on` recursively and topologically sort Skills.
2. Check each Skill version range before preparing its environment.
3. Merge runtime requirements in dependency order.
4. Merge outcome-specific commands, features, and tighter version ranges declared by the selected setup outcome.
5. For the same runtime name, intersect all SemVer ranges and require identical kind, command, protocol, and probe fields.
6. Stop on an empty version intersection, protocol mismatch, unknown runtime kind, or missing setup reference.
7. Prepare dependency runtimes before dependent instance configuration.

A business Skill may inherit a runtime through another Skill. Do not duplicate or independently install the same runtime for every dependent.

## Plan

Before modifying the environment, state:

- already satisfied outcomes;
- current and required runtime versions, protocols, and capabilities;
- selected trusted source and installation strategy;
- planned user-level changes;
- affected paths and configuration;
- local build or verification helpers that may run;
- network access and authorization needs;
- manual steps that cannot be completed safely.

The target setup reference defines the required results. Commands are implementation choices.

## Runtime setup

For each runtime:

1. Resolve a candidate command from the current environment and any valid receipt.
2. Run the catalog probe without application configuration or data access.
3. Compare product identity, version, protocol, and every required command and feature. For work-ledger, the base gate is CLI `>=0.11.0,<1.0.0`, protocol 1, commands `knowledge.list` and `knowledge.show`, `features.knowledge_documents=true`, and `features.knowledge_kind_document=true`.
4. If compatible, preserve it and refresh verification metadata without reinstalling.
5. If missing or incompatible, choose an exact version from a source allowed by the target setup reference.
6. For a package-index source, resolve the exact package and version coordinate declared by the trusted source; do not substitute TestPyPI for normal installation or guess another index. For a user-provided executable, wheel, zipapp, or local project, record its credential-free verified artifact identity instead of assigning it a package coordinate.
7. Install the candidate into an isolated user-level location using an environment-appropriate tool, wheel, zipapp, or other declared artifact.
8. Probe the candidate directly before changing a stable entry.
9. Run the target's temporary smoke test. Work-ledger must initialize schema 6 and query created `document` Knowledge through both required commands.
10. If no persisted-data migration is required, atomically switch the stable executable and write the runtime receipt. Otherwise retain the verified candidate by absolute path and defer the switch until its migration commits successfully.

A process that starts or exits zero is not necessarily compatible. If any version, protocol, command, feature, or schema gate is missing, stop and block all business writes. Do not route writes through an old executable, edit managed Vault files directly, or treat `report.facts` as a substitute for Knowledge commands.

Source checkout or editable installation is acceptable only for explicit development mode. Normal and scheduled operation must not depend on a mutable working tree.

Commands are implementation choices. Do not turn one successful `uv`, `pipx`, venv, symlink, or wrapper command into the universal contract.

## Instance setup

Only begin after all required runtimes pass compatibility and smoke checks.

- Prepare application configuration, data directories, repositories, external services, and scheduled jobs defined by the target setup reference.
- Keep secrets and personal paths in user configuration outside this repository.
- Preserve compatible existing values and unrelated files.
- Ask for user choices that define real data scope, such as a Vault path or Git remote.
- Store a scheduled executable as the verified absolute path, not an assumed interactive `PATH` lookup.
- Do not create or modify real data merely to prove runtime installation.

## Receipts and verification

Default runtime receipt:

```text
~/.local/state/skill-setup/runtimes/<runtime-name>.json
```

Canonical fields:

```json
{
  "schema_version": 1,
  "name": "work-ledger-cli",
  "version": "0.11.0",
  "protocol_version": 1,
  "executable": "/absolute/user-level/path/work-ledger",
  "source_kind": "pypi",
  "source": "agent-ledger-harness==0.11.0",
  "managed": true,
  "installed_at": "RFC3339",
  "verified_at": "RFC3339"
}
```

- Write receipts atomically.
- Use `managed=false` when setup verified but did not install the runtime.
- Do not store credentials, token-bearing URLs, private content, or repository remotes.
- Normalize package-index sources to a credential-free package and version coordinate.
- Treat a receipt as discovery metadata, not proof; always probe the executable.
- Replace or repair stale receipts only after a candidate passes verification.

Verify each declared outcome directly. For work-ledger, check at least:

- `--version`, JSON `version`, and `capabilities` agree on the standalone product and version;
- CLI is `>=0.11.0,<1.0.0`, protocol 1, and reports `features.reportable_project_default=true`, `features.knowledge_documents=true`, and `features.knowledge_kind_document=true`;
- capabilities contains commands `knowledge.list` and `knowledge.show` with that exact spelling;
- a temporary initialization produces Vault schema 6, and the intended Vault is schema 6 before any business write;
- CLI SemVer, protocol, commands, features, and schema satisfy catalog and the target setup reference;
- the receipt executable is the executable that was probed;
- Python is 3.11 or newer and Git is available;
- configuration parses and points at the intended absolute Vault;
- the Vault is a Git repository root with the intended branch and identity;
- active mode has a remote, upstream, and successful push authentication;
- `doctor` has no fatal findings;
- a temporary atomic Event-plus-Knowledge write can be listed and shown, and a safe real or temporary write produces exactly one scoped commit;
- the scheduled Agent calls the same stable executable by absolute path.

For an explicitly requested Work Ledger Obsidian client outcome, additionally verify commands `snapshot` and `report.export`, `features.read_only_snapshot=true`, `features.clean_report_export=true`, snapshot schema 2 from a schema 6 Vault, matching Vault identity, a `document` Knowledge projection, body-free entity summaries, a stable digest across repeated reads, and clean Markdown/text Report export without a Vault or Git mutation. Probe the exact stable executable from a minimal desktop-app environment without interactive shell initialization. If a zipapp's `/usr/bin/env python3` resolves below Python 3.11 there, use an isolated-environment console script or a stable wrapper with an absolute verified interpreter instead. Do not apply this tighter gate to unrelated todo-tracker, work-log, or weekly-report setup.

For an explicitly requested `weekly-report` outcome, additionally verify command `report.facts` and `features.rich_report_facts=true`. In a temporary schema 6 Vault, require audience-filtered Project/Task/Event/Knowledge facts, ancestor Task context for an event-linked child, Knowledge evidence validation, and evidence rejection for an unmarked `（1）` factual item. Do not migrate or rewrite the real Vault merely to verify this outcome, and do not apply this tighter gate to todo-tracker or work-log.

Do not create the user's real Vault or remote without their path and remote choice.

## Repair, update, and removal

On repeated setup:

1. Re-read catalog and setup references.
2. Compare Skill, runtime, protocol, schema, capabilities, executable, and receipt.
3. Repair only drifting receipts, stable entries, schedules, or unmet outcomes.
4. Install an update in isolation and keep the old runtime until the new candidate passes probes and temporary smoke tests.
5. For schema 1–5, use the verified candidate to run `migrate plan --to 6`; inspect collisions, unmanaged links, child-project changes, `4->5-knowledge-documents`, and byte-preserving `5->6-document-knowledge-kind`.
6. Apply only the exact reviewed `plan_digest` after the candidate runtime passes its temporary smoke test; verify the single migration commit, schema 6, and `doctor` result.
7. Switch the stable entry and receipt atomically only after migration succeeds; preserve or restore the old entry when verification or a pre-commit migration fails.
8. Explain upgrade and removal impact before replacing or removing anything.

Removal requires explicit user scope. Removing a runtime does not imply removing application configuration, local state, data stores, repositories, remotes, schedules, or dependent Skills. Never infer cascading deletion.

## Safety

- Trust installed Skills, target-declared sources, the current trusted repository, and user-provided local artifacts; do not search for and execute arbitrary scripts.
- Do not silently install a mutable branch, unpinned source, or unverified “latest” artifact.
- Do not guess a private index URL, use a public registry as fallback, or persist registry credentials.
- Never print credential-bearing remote URLs.
- Do not change global Git identity without explicit approval; prefer repository-local identity.
- Do not modify real application data before runtime verification.
- Do not automatically merge, rebase, force-push, remove a working runtime, or delete old configuration.
- Request authorization for privileged, system-wide, destructive, or externally consequential changes.
