---
name: work-ledger
description: Operate the independently installed work-ledger CLI behind an Obsidian-compatible, Git-backed personal work ledger. Use when another Skill or the user needs exact project/task/event CRUD, transitions, correction or compensation, object resolution, consistent read-only snapshots, visibility previews, weekly-report facts or writes, doctor, migration, initialization, Git sync, CLI compatibility checks, or low-level diagnosis. This Skill is the CLI manual and safety contract; environment preparation routes to skill-setup, while business-facing requests normally route through todo-tracker, work-log, or weekly-report.
---

# Work Ledger

Use the independently distributed CLI as the only writer for managed `Work/` Markdown. Let the Agent interpret intent; let the CLI enforce IDs, schemas, links, locks, transactions, Git history, and privacy.

## Start

1. Resolve `work-ledger` from the current execution environment; never derive an executable from this Skill directory.
2. Run `work-ledger version`. Require `product=work-ledger-cli`, `cli_version>=0.10.0,<1.0.0`, and `protocol_version=1`.
3. Run `work-ledger capabilities` and require `readable_project_notes`, `immutable_project_titles`, `readable_task_notes`, `immutable_task_titles`, `inherited_child_projects`, `reportable_project_default`, plus every feature needed by the request.
   For a visualization snapshot, additionally require command `snapshot` and `read_only_snapshot=true`.
   For a clean Report export, additionally require command `report.export` and `clean_report_export=true`.
   For evidence-rich weekly Report generation, additionally require command `report.facts` and `rich_report_facts=true`.
4. Stop before any business write and use `skill-setup` when the command is missing, the version or protocol is incompatible, or the stable entry point has drifted.
5. Read [references/cli.md](references/cli.md) before constructing a command or request.
6. Read [references/data-model.md](references/data-model.md) for state, hierarchy, visibility, correction, compensation, or historical-report decisions.
7. Read [references/recovery.md](references/recovery.md) for conflicts, transaction errors, Git warnings, doctor findings, or migration.
8. Read [references/setup.md](references/setup.md) when the CLI, configuration, Vault, Git, or schedule is not ready; then use `skill-setup` to satisfy it.

## Invocation Rules

- Pass complex writes through an absolute UTF-8 JSON request file.
- Obtain a fresh `operation_id` with `operation new` and preserve it for safe retries of the same request.
- Query an object and use its current `revision` before updating it.
- Parse stdout as exactly one JSON object and use the exit code plus `ok`, `error`, `warnings`, and `git.push`.
- Treat `ok=true` with `GIT_PUSH_PENDING` or `REMOTE_DIVERGED` as a completed local business write. Do not replay the mutation.
- Present ambiguity candidates to the user; never select a fuzzy match silently.
- Treat a project title as its permanent Obsidian note name. Never submit `title` in `project.update`.
- Treat a task title as its permanent Obsidian note name. Never submit `title` in `task.update`.
- Never edit managed Project, Task, Journal, or Report files directly.
- Never auto-promote visibility, pull, merge, rebase, force-push, or physically delete business history.
