---
name: skill-setup
description: Prepare, verify, repair, update, or remove the local execution environment required by one or more Agent Skills. Use when a user asks to set up a Skill, satisfy an independently distributed CLI or other runtime dependency, initialize an application instance, make an executable stable across Codex/Qoder/OpenCode or scheduled tasks, diagnose version or protocol incompatibility, repair a drifting runtime receipt or entry point, or configure a Skill such as work-ledger. Treat installation commands as adaptable implementation choices, not as the setup contract.
---

# Skill Setup

Prepare a selected Skill's execution environment by satisfying its declared outcomes. Adapt to the current Agent and machine; do not impose one installer or fixed command sequence.

## Workflow

1. Identify the requested Skill from the current Agent inventory, repository `catalog.yaml`, or a user-provided Skill path.
2. Validate catalog schema, resolve `depends_on` in topological order, and merge `runtime_requires`; require intersecting version ranges and identical protocols for the same runtime.
   Add outcome-specific capability requirements declared by the target setup reference without changing unrelated consumers' minimum versions.
3. Read each target Skill's linked setup reference completely before changing the environment.
4. Probe existing runtimes without side effects. Compare executable, SemVer, protocol, capabilities, and any local runtime receipt.
5. Report what already works, what must change, exact trusted source and version, affected user-level paths, network use, and any required authorization.
6. Complete Runtime preparation first: select an allowed strategy, install into an isolated user-level location, verify it with a temporary smoke test, and keep the prior stable executable available until any required persisted-data migration commits successfully.
7. Complete Instance setup second: configure application paths, data stores, external services, and schedules only after the runtime is verified.
8. When persisted schema differs, generate a migration plan, verify its exact affected paths and risks, then apply only that plan digest through the candidate runtime's transactional migration surface; atomically switch the stable executable and receipt immediately after success.
9. Verify declared outcomes directly. Do not equate “a command exited zero” with setup success.
10. Preserve compatible configuration, keep a working runtime when an upgrade fails, and make repeated setup idempotent.

Read [references/setup-protocol.md](references/setup-protocol.md) for catalog discovery, safety, verification, repair, and update rules.

## Boundaries

- Do not recreate an Agent's Skill installation or publication ecosystem.
- Do not download and execute undeclared remote scripts.
- Do not install a mutable branch, unpinned source, or unverified “latest” artifact without explicit user direction.
- Do not invent a package-index URL, silently fall back to a public index, or expose package-index credentials in commands, logs, or receipts.
- Do not hard-code Agent-specific installation paths into shared Skill files.
- Do not store Vault paths, Git remotes, identities, tokens, or other personal configuration in this repository.
- Do not modify real application data before the selected runtime passes compatibility and temporary smoke checks.
- Do not remove a runtime, configuration, schedule, or data store unless the user explicitly requests that scope.
- Do not run `sudo` automatically.
- Stop and explain a version, dependency, or migration conflict instead of inventing a migration.
