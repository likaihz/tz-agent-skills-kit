# Work Ledger Setup

## Outcomes

- An independently distributed `work-ledger-cli` satisfies CLI version `>=0.11.0,<1.0.0` and protocol version `1`.
- Python 3.11+ and Git can run the selected CLI distribution.
- The current Agent, dependent Skills, and scheduled Agent share one verified, stable executable.
- The skill-setup runtime receipt matches the executable, version, protocol, and trusted source.
- Configuration parses and points to one intended absolute Vault.
- The Vault is a Git repository root on the configured branch.
- Git identity is available; active mode has a remote, upstream, auto-push, and working authentication.
- Vault schema 5 is active, child Tasks inherit project membership without redundant Obsidian Graph edges, readable permanent project/task notes and Knowledge documents are supported, `Inbox.md` and `Work/Knowledge/.gitkeep` exist, `doctor` has no fatal findings, and a safe transaction creates one scoped commit.

## Runtime requirements

- Runtime name: `work-ledger-cli`.
- Command: `work-ledger`.
- CLI version: `>=0.11.0,<1.0.0`.
- Protocol version: `1`.
- Probe: `work-ledger version`, followed by `work-ledger capabilities`; require commands `knowledge.list` and `knowledge.show`, plus `features.project_date_range=true`, `features.readable_project_notes=true`, `features.immutable_project_titles=true`, `features.readable_task_notes=true`, `features.immutable_task_titles=true`, `features.inherited_child_projects=true`, `features.reportable_project_default=true`, and `features.knowledge_documents=true`.
- The probes must not require configuration, create state, access a Vault, or use the network.

When the requested outcome includes the Work Ledger Obsidian client, additionally require commands `snapshot` and `report.export`, plus `features.read_only_snapshot=true` and `features.clean_report_export=true`. Verify a temporary schema 5 Vault with a body-free `snapshot_schema_version=1` response, stable repeated `snapshot_digest`, a Knowledge projection, and clean Markdown/text export from a generated report. The stable executable must also work from a desktop-app environment that does not load the user's interactive shell. Prefer an isolated-environment console script with an absolute Python shebang, or a stable wrapper that invokes a verified Python 3.11+ interpreter by absolute path. Do not expose a zipapp whose `/usr/bin/env python3` can silently resolve to an older system Python unless that exact entry has passed the desktop-environment probe.

When the requested outcome includes `weekly-report`, additionally require command `report.facts` and `features.rich_report_facts=true`. In a temporary schema 5 Vault, verify that facts include visibility-filtered Project, Task, Event, and Knowledge bodies, add the visible ancestor of an event-linked child Task to `derived.context_task_ids`, accept only facts-package IDs as evidence, and reject a full-width numbered factual item without evidence. This outcome does not require a real Vault migration.

## Prerequisites

- A POSIX platform with `fcntl.flock` support and Python 3.11 or newer.
- Git with a user identity available globally or repository-locally.
- A user-selected Vault path and, for active mode, intended remote URL.
- Permission to create user-level configuration and state directories.

## Trusted sources

Choose sources in this order:

1. An already installed compatible executable.
2. A user-provided executable, wheel, zipapp, or local project.
3. An exact released `agent-ledger-harness` version from production PyPI.
4. `packages/work-ledger-cli` or its verified artifact when the trusted repository checkout is available.

Do not use TestPyPI for normal installation, search for and run arbitrary remote installers, or silently use a mutable branch or unverified “latest” artifact. Select an exact compatible version and keep credentials out of commands, logs, and receipts.

## Local configuration

Default configuration is `~/.config/work-ledger/config.toml`; work-ledger state is under `~/.local/state/work-ledger/`. The runtime receipt is `~/.local/state/skill-setup/runtimes/work-ledger-cli.json`. `WORK_LEDGER_CONFIG` and `WORK_LEDGER_STATE_DIR` may isolate tests. Do not commit these files or store credentials in them.

## Optional helpers

When the standalone source project is available, `tools/build_pypi.py` verifies wheel and sdist artifacts, while `tools/build_zipapp.py` produces the separate executable zipapp and checksums. Setup may choose a user-level tool manager, isolated virtual environment, wheel, or executable zipapp. Do not require one universal installation command, and do not use an editable source checkout for normal or scheduled use.

## Verification

Verify runtime before instance configuration:

1. Run `--version`, JSON `version`, and `capabilities`.
2. Compare product, CLI version, protocol, schemas, and required commands with catalog.
3. Confirm the runtime receipt resolves to the executable that was probed and records the exact production PyPI version or verified local artifact without credentials.
4. For a new or upgraded runtime, initialize a temporary schema 5 Vault, atomically create an Event plus sourced Knowledge through a backward `source_event_refs` local ref, query it, perform one scoped update using its revision, run `doctor` and `sync`, and preserve no temporary state.
5. For an Obsidian client outcome, also probe the stable entry with a minimal desktop-style `PATH`, run `snapshot`, verify Vault identity and schema, confirm Project/Task/Event bodies are absent, repeat the same query to verify a stable digest, and verify that `report export` returns clean Markdown and text without changing the Vault or Git HEAD.
6. For a weekly-report outcome, verify rich report facts and numbered-item evidence rejection in the temporary Vault without changing the real Vault.

Then verify the intended instance: configuration, Vault root, Vault schema 5, Git branch, identity, remote, upstream, one safe transaction, exact staging, one commit, push state, and the scheduled invocation's absolute executable.

## Repair and update

Preserve compatible configuration and Vault data. Repair a drifting receipt, executable, user-level entry, or schedule without recreating the Vault.

Install an upgrade into an isolated location, probe it, and run a temporary Vault smoke test before touching the stable executable or real Vault. For any schema 1–4 Vault, run `migrate plan --to 5`; review path collisions, unmanaged-link risk, every `child_project_migrations` entry, and the `4->5-knowledge-documents` step. Any Markdown already under a schema-4 `Work/Knowledge/` is a `MIGRATION_CONFLICT`, not import input. If the plan reports child-project or Knowledge-path conflicts, stop and request an explicit data decision instead of modifying the Vault. Apply only the exact reviewed digest as one Git-backed transaction, then re-run `doctor`. If migration fails before commit, preserve the old schema and restore the old stable executable and receipt.

## Removal impact

Removing the runtime disables dependent Skills and schedules but does not delete work-ledger configuration, state, Vault data, Git history, or remote. Removing local configuration makes the CLI unable to locate its Vault. Never delete any of them automatically.

## Safety

Do not create the real Vault until the user chooses its path and Git remote. Do not modify a real Vault until the runtime is compatible and verified. Do not change global Git identity automatically. Do not clone over a non-empty path, merge remote history into an empty local Vault automatically, expose secrets, use `sudo`, or force-push.
