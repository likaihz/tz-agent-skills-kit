# CLI Reference

## Contents

1. Protocol
2. Command surface
3. Apply
4. Queries
5. Consistent snapshots
6. Reports
7. Exit and Git results

## Protocol

Invoke the independently installed `work-ledger` executable resolved and verified by the current environment. Never invoke a path relative to this Skill. Except for `--version`, stdout is one JSON object; diagnostics use stderr. Request files must be absolute UTF-8 JSON paths. All writes require `schema_version: 1` and `operation_id: op-<UUIDv4>`.

Create an operation ID:

```text
work-ledger operation new
```

Discover the runtime:

```text
work-ledger --version
work-ledger version
work-ledger capabilities
```

Require product `work-ledger-cli`, CLI version `>=0.10.0,<1.0.0`, protocol version `1`, `features.readable_project_notes=true`, `features.immutable_project_titles=true`, `features.readable_task_notes=true`, `features.immutable_task_titles=true`, `features.inherited_child_projects=true`, `features.reportable_project_default=true`, and any capability required by the request. Project date operations require `features.project_date_range=true`; evidence-rich weekly Reports require `features.rich_report_facts=true`. Use `skill-setup` instead of attempting a business operation when any check fails.

## Command surface

```text
work-ledger init --request-file <absolute-path>
work-ledger apply --request-file <absolute-path>
work-ledger project list|show
work-ledger task list|show
work-ledger event list|show
work-ledger snapshot [--events-from <date|RFC3339>] [--events-to <date|RFC3339>] [--event-limit <1-5000>]
work-ledger resolve --type project|task --query <text>
work-ledger visibility preview --type project|task --id <id> --to reportable
work-ledger report due --at <RFC3339>
work-ledger report facts --week <YYYY-Www> --audience personal|reportable [--source latest|<commit>]
work-ledger report export --week <YYYY-Www> --audience personal|reportable --format markdown|text
work-ledger report write --request-file <absolute-path>
work-ledger doctor [--scope config|git|data|reports|transactions|all]
work-ledger migrate plan --to <version>
work-ledger migrate apply --request-file <absolute-path>
work-ledger sync
```

## Apply

Top-level request:

```json
{
  "schema_version": 1,
  "operation_id": "op-UUIDv4",
  "source": {"agent": "codex", "surface": "desktop"},
  "mutations": []
}
```

Supported mutation types:

- `project.create`, `project.update`
- `task.create`, `task.update`, `task.transition`, `task.compensate-transition`
- `event.add`, `event.correct`

Use `ref: "m-name"` on a create and `project_ref`, `task_ref`, or `parent_ref` in later mutations to make a multi-object atomic request. A ref can only point backward in the same request.

Updates require the exact `expected_revision` returned by `show` or `resolve`. Status changes must use `task.transition`; the helper derives the linked event type. Use one apply request for related changes such as compensating the wrong task and transitioning the correct task.

`project.create` and `project.update` accept nullable ISO `start_date` and `end_date`. When both are present, require `start_date <= end_date`. A missing boundary remains `null`; setting a boundary to `null` clears it. These dates describe the project range but do not change status or constrain task and event dates.

Omitting `project.create.visibility` creates an ordinary Project as `reportable`. Pass `private` explicitly when the Project should remain personal. This default never applies to the protected Inbox and never authorizes changing an existing private Project.

`project.create.title` becomes the permanent readable note name after portable filename projection. It must be unique after projection. `project.update.changes` never accepts `title`; a rename returns `IMMUTABLE_FIELD`.

`task.create.title` follows the same permanent readable-note rule and must be globally unique after projection. `task.update.changes` never accepts `title`; a rename returns `IMMUTABLE_FIELD`. Use stable IDs for CLI addressing even though human-facing wikilinks target readable filenames.

For `task.create`, provide exactly one ownership edge: `project_id`/`project_ref` for a root task, or `parent_id`/`parent_ref` for a child. Never provide both. Child project membership is derived recursively and still appears as `project_id` in query and report results.

For `task.update`, `project_id` is valid only when the resulting task is a root. Setting a non-null `parent_id` makes the task and all descendants inherit that parent's project. Setting `parent_id` to null keeps the child's previously effective project and writes it on the new root.

## Queries

Lists support deterministic filters, stable ordering, `limit`, and opaque `cursor`. Reusing a cursor with different filters returns `CURSOR_QUERY_MISMATCH`.

Resolution order is ID, normalized exact title, substring, then all-token match. Multiple candidates remain ambiguous.

Use `event show --view effective` for corrected business content and `--view audit` for original, correction, transition, and compensation records.

## Consistent snapshots

Use `snapshot` only for a read-only visualization or navigation client that needs Project, Task, effective Event, and Report summaries from one shared read lock. Require CLI `>=0.7.0,<1.0.0`, command `snapshot`, `features.read_only_snapshot=true`, Vault schema 4, and `snapshot_schema_version=1`.

The response includes body-free entities, relative Obsidian paths, child Tasks with their derived `project_id`, Vault identity, Git HEAD, Work digest, event window, and snapshot digest. Load Project, Task, or Event bodies only with the corresponding `show` command when the user opens that detail.

Treat `events-from` as inclusive and `events-to` as exclusive. Date-only values mean local midnight in the configured timezone. Reuse `event_window.next_cursor` only with `event list` using the exact normalized window and limit from that snapshot.

Never reconstruct inherited project membership, effective visibility, correction, compensation, or report state in the client. Never persist snapshot business content outside the intended in-memory client state.

## Reports

Always request personal and reportable facts independently. Never derive reportable facts by filtering the personal package in the Agent. CLI 0.9 facts include Project and Task bodies plus ancestor Tasks in `derived.context_task_ids` so the Agent can group Events into workstreams. Bodies remain subject to the audience's effective visibility.

`report write` accepts both bodies in one request and writes them atomically. Each body file is an absolute temporary Markdown path. Supply evidence markers immediately after factual items; validation recognizes Markdown bullets, `（1）`, `(1)`, and `1.` prefixes:

```markdown
（1）完成确定性流水实现。[[Work/Tasks/完成确定性流水实现|完成确定性流水实现]]
  <!-- evidence: task-...,event-... -->
```

Use `conflict_policy: fail` by default. Use `replace` or `candidate` only after the user chooses how to handle an externally modified report.

`report export` is a read-only projection of one existing, digest-valid managed Report. Require CLI `>=0.8.0,<1.0.0` and `features.clean_report_export=true`. It requires an explicit audience and format. Markdown export removes controlled frontmatter and evidence comments while reducing Obsidian wikilinks to their visible labels; text export also removes Markdown presentation syntax. A personal export returns `PERSONAL_EXPORT_MAY_CONTAIN_PRIVATE_CONTENT`. Never treat export as proof of sending or copy a managed Report file verbatim for external use.

## Exit and Git results

- `0`: success, including a committed business write whose push is pending or diverged.
- `2`: input/configuration error.
- `3`: not found or ambiguous.
- `4`: business validation or conflict.
- `5`: lock, transaction, or filesystem failure.
- `6`: Git commit failed and business files were rolled back.
- `7`: unsupported schema or damaged controlled data.
- `8`: standalone `sync` did not complete.

For a committed write, `git.push` is `pushed`, `pending`, `diverged`, or `not_needed`. Never retry the business mutation merely because push was not completed.
