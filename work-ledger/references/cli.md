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

Require product `work-ledger-cli`, CLI version `>=0.11.0,<1.0.0`, protocol version `1`, `features.readable_project_notes=true`, `features.immutable_project_titles=true`, `features.readable_task_notes=true`, `features.immutable_task_titles=true`, `features.inherited_child_projects=true`, `features.reportable_project_default=true`, `features.knowledge_documents=true`, `features.knowledge_kind_document=true`, commands `knowledge.list` and `knowledge.show`, and any capability required by the request. Project date operations require `features.project_date_range=true`; evidence-rich weekly Reports require `features.rich_report_facts=true`. Use `skill-setup` instead of attempting a business operation when any check fails.

## Command surface

```text
work-ledger init --request-file <absolute-path>
work-ledger apply --request-file <absolute-path>
work-ledger project list|show
work-ledger task list|show
work-ledger event list|show
work-ledger knowledge list|show
work-ledger snapshot [--events-from <date|RFC3339>] [--events-to <date|RFC3339>] [--event-limit <1-5000>]
work-ledger resolve --type project|task|knowledge --query <text>
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
- `knowledge.create`, `knowledge.update`

Use `ref: "m-name"` on a create and `project_ref`, `task_ref`, or `parent_ref` in later mutations to make a multi-object atomic request. A ref can only point backward in the same request.

`event.add` may also define a ref for a later `knowledge.create` or `knowledge.update`. Supply that earlier Event ref in `source_event_refs`; refs are unique, backward-only, and typed. Knowledge source IDs and refs are merged, de-duplicated, and stored in canonical Event order. This fixture-aligned request creates both objects atomically:

```json
{
  "schema_version": 1,
  "operation_id": "op-22222222-2222-4222-8222-222222222222",
  "source": {
    "agent": "codex",
    "surface": "cli"
  },
  "mutations": [
    {
      "type": "event.add",
      "ref": "m-inbox-idea",
      "occurred_at": "2026-08-11T10:00:00+08:00",
      "time_precision": "exact",
      "event_type": "idea",
      "project_id": "project-inbox",
      "summary": "Capture a query contract",
      "body": "Knowledge list must bind every filter into its cursor.",
      "visibility": "private"
    },
    {
      "type": "knowledge.create",
      "ref": "m-query-note",
      "title": "Knowledge query contract",
      "slug": "Knowledge query contract",
      "kind": "technical_note",
      "status": "draft",
      "project_id": null,
      "source_event_ids": [],
      "source_event_refs": ["m-inbox-idea"],
      "body": "The cursor signature covers kinds, statuses, project, visibility, tag, and query.",
      "visibility": "private",
      "tags": ["cli", "pagination"]
    }
  ]
}
```

`knowledge.create` requires `title` and `kind`. Optional fields are `ref`, `slug`, `status`, `project_id`, `project_ref`, `source_event_ids`, `source_event_refs`, `body`, `visibility`, and `tags`. Supply at most one of `project_id`/`project_ref`. Omitting both selects a null Project. Other defaults are projected slug, draft, private, empty sources/body/tags. Stable requires a non-empty body. Reportable Knowledge requires a reportable Project and every source Event chain to have reportable effective visibility; otherwise the whole apply fails with `VISIBILITY_CONFLICT`.

`knowledge.update` requires `knowledge_id`, the current `expected_revision`, and `changes`. It may change `title`, `kind`, `status`, `project_id`, `source_event_ids`, `source_event_refs`, `body`, `visibility`, and `tags`. A status, Project, visibility, or complete source replacement also requires a non-empty `reason`; title, kind, body, and tags do not. `id`, immutable slug, and `created_at` reject with `IMMUTABLE_FIELD`. A title update preserves the existing slug/path. Sources are complete replacement, not a patch. Use this fixture-aligned shape:

```json
{
  "schema_version": 1,
  "operation_id": "op-33333333-3333-4333-8333-333333333333",
  "source": {
    "agent": "codex",
    "surface": "cli"
  },
  "mutations": [
    {
      "type": "knowledge.update",
      "knowledge_id": "knowledge-20260811-001",
      "expected_revision": "sha256:0000000000000000000000000000000000000000000000000000000000000000",
      "reason": "Publish reviewed evidence",
      "changes": {
        "title": "Reviewed knowledge query contract",
        "status": "stable",
        "project_id": null,
        "source_event_ids": ["event-20260811-100000-001"],
        "source_event_refs": [],
        "body": "Every pagination filter is part of the canonical cursor signature.",
        "visibility": "private",
        "tags": ["cli", "pagination", "reviewed"]
      }
    }
  ]
}
```

Updates require the exact `expected_revision` returned by `show` or `resolve`. Status changes must use `task.transition`; the helper derives the linked event type. Use one apply request for related changes such as compensating the wrong task and transitioning the correct task.

`project.create` and `project.update` accept nullable ISO `start_date` and `end_date`. When both are present, require `start_date <= end_date`. A missing boundary remains `null`; setting a boundary to `null` clears it. These dates describe the project range but do not change status or constrain task and event dates.

Omitting `project.create.visibility` creates an ordinary Project as `reportable`. Pass `private` explicitly when the Project should remain personal. This default never applies to the protected Inbox and never authorizes changing an existing private Project.

`project.create.title` becomes the permanent readable note name after portable filename projection. It must be unique after projection. `project.update.changes` never accepts `title`; a rename returns `IMMUTABLE_FIELD`.

`task.create.title` follows the same permanent readable-note rule and must be globally unique after projection. `task.update.changes` never accepts `title`; a rename returns `IMMUTABLE_FIELD`. Use stable IDs for CLI addressing even though human-facing wikilinks target readable filenames.

For `task.create`, provide exactly one ownership edge: `project_id`/`project_ref` for a root task, or `parent_id`/`parent_ref` for a child. Never provide both. Child project membership is derived recursively and still appears as `project_id` in query and report results.

For `task.update`, `project_id` is valid only when the resulting task is a root. Setting a non-null `parent_id` makes the task and all descendants inherit that parent's project. Setting `parent_id` to null keeps the child's previously effective project and writes it on the new root.

## Queries

Lists support deterministic filters, stable ordering, `limit`, and opaque `cursor`. Reusing a cursor with different filters returns `CURSOR_QUERY_MISMATCH`.

Project and Task resolution order is ID, normalized exact title, substring, then all-token match. Knowledge resolution order is ID, normalized exact title, normalized exact slug, then fuzzy substring/all-token matching. Multiple candidates remain ambiguous.

Use `event show --view effective` for corrected business content and `--view audit` for original, correction, transition, and compensation records.

Use these Knowledge queries:

```text
work-ledger knowledge list [--kind <kind>]... [--status <status>]... [--project <id|none>] [--visibility <private|reportable>] [--tag <tag>] [--query <text>] [--limit <1-500>] [--cursor <cursor>]
work-ledger knowledge show --id <knowledge-id>
work-ledger resolve --type knowledge --query <text> [--project <id|none>] [--status <status>]... [--limit <1-20>]
```

Knowledge list excludes archived by default. Repeated kind/status filters are OR within their type; different filters are AND. Visibility means effective visibility. Query searches title, slug, tags, and body. A cursor binds the complete normalized query. Show returns body, revision, resolved Project, effective source Events, and `suppressed_source_event_ids`. Corrections resolve to their origin Event; Event show returns sorted `knowledge_ids` backlinks.

## Consistent snapshots

Use `snapshot` only for a read-only visualization or navigation client that needs Project, Task, effective Event, Knowledge, and Report summaries from one shared read lock. Require CLI `>=0.11.0,<1.0.0`, command `snapshot`, `features.read_only_snapshot=true`, `features.knowledge_documents=true`, `features.knowledge_kind_document=true`, Vault schema 6, and `snapshot_schema_version=2`.

The response includes body-free entities, relative Obsidian paths, child Tasks with their derived `project_id`, immutable Knowledge slug/path, effective visibility, Vault identity, Git HEAD, Work digest, event window, and snapshot digest. Load Project, Task, Event, or Knowledge bodies only with the corresponding `show` command when the user opens that detail.

Treat `events-from` as inclusive and `events-to` as exclusive. Date-only values mean local midnight in the configured timezone. Reuse `event_window.next_cursor` only with `event list` using the exact normalized window and limit from that snapshot.

Never reconstruct inherited project membership, effective visibility, correction, compensation, or report state in the client. Never persist snapshot business content outside the intended in-memory client state.

## Reports

Always request personal and reportable facts independently. Never derive reportable facts by filtering the personal package in the Agent. CLI 0.11 facts include Project and Task bodies, Knowledge facts, and ancestor Tasks in `derived.context_task_ids` so the Agent can group Events and Knowledge into workstreams. Bodies remain subject to the audience's effective visibility. Only IDs present in that audience's `projects`, `tasks`, `events`, or `knowledge` facts collections are valid evidence.

`report write` accepts both bodies in one request and writes them atomically. Each body file is an absolute temporary Markdown path. Supply evidence markers immediately after factual items; validation recognizes Markdown bullets, `（1）`, `(1)`, and `1.` prefixes:

```markdown
（1）完成确定性流水实现。[[Work/Tasks/完成确定性流水实现|完成确定性流水实现]]
  <!-- evidence: task-...,event-... -->
```

Use `conflict_policy: fail` by default. Use `replace` or `candidate` only after the user chooses how to handle an externally modified report.

`report export` is a read-only projection of one existing, digest-valid managed Report. Require CLI `>=0.11.0,<1.0.0` and `features.clean_report_export=true`. It requires an explicit audience and format. Markdown export removes controlled frontmatter and evidence comments while reducing Obsidian wikilinks to their visible labels; text export also removes Markdown presentation syntax. A personal export returns `PERSONAL_EXPORT_MAY_CONTAIN_PRIVATE_CONTENT`. Never treat export as proof of sending or copy a managed Report file verbatim for external use.

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

Knowledge-specific business errors include `KNOWLEDGE_SLUG_CONFLICT`, `INVALID_KNOWLEDGE_SOURCE`, `VISIBILITY_CONFLICT`, `IMMUTABLE_FIELD`, `INVALID_TRANSITION`, `CONFLICT`, and `MIGRATION_CONFLICT`. Doctor reports invalid closure or digest coverage as errors and a compensated source as the `KNOWLEDGE_SOURCE_SUPPRESSED` warning. Do not invent alternate warning or collision codes.
