# Data Model

## Contents

1. Objects and links
2. Task state
3. Visibility
4. Events and corrections
5. Reports

## Objects and links

- Projects: `Work/Projects/<permanent-readable-title>.md`
- Tasks: `Work/Tasks/<permanent-readable-title>.md`
- Journal: `Work/Journal/YYYY/MM/YYYY-MM-DD.md`
- Reports: `Work/Reports/YYYY/YYYY-Www-{personal|reportable}.md`

Project and task IDs never change and remain in controlled frontmatter. Each creation title is permanent and projects to a portable, readable Obsidian filename; the CLI rejects a rename or a filename collision. Task note names are globally unique because all tasks share one directory. Visible relationships use Obsidian wikilinks with aliases, while event metadata stores stable IDs. `project-inbox` is the protected private `Inbox.md` project for unclassified work.

Vault schema 4 records this path and hierarchy contract in `Work/.work-ledger.json`. Vault schema 1 used project and task ID filenames; schema 2 used readable project filenames but retained task ID filenames; schema 3 used readable filenames but stored both project and parent links on child Tasks. All older schemas require a planned forward migration before business operations.

Projects have nullable `start_date` and `end_date` ISO dates. Either boundary may be absent; when both exist, `start_date` must be on or before `end_date`. The range is descriptive and inclusive: it does not archive a project automatically or reject tasks and events outside the range. `project-inbox` always has both boundaries unset. Vaults created by CLI 0.2.x may omit these fields and are read as `null`.

A root Task stores a `project` wikilink and an empty `parent`. A child Task stores a `parent` wikilink and omits `project`; its effective project is inherited recursively. CLI query and report results always expose the derived `project_id`. Cross-project parent-child relationships are not supported.

Moving a root Task to another project moves its complete descendant tree implicitly. Reparenting a Task makes its subtree inherit the new parent's project. Detaching a child into a root materializes its previously effective project as the root's `project` link.

## Task state

Statuses are `inbox`, `planned`, `in_progress`, `blocked`, `done`, and `cancelled`.

- Entering `in_progress` records `progress`.
- Entering `blocked` records `blocker` with a reason.
- Entering `done` records `result`.
- Cancellation, pause, and reopen record `note` with a reason.
- A parent cannot become terminal while any descendant is active.
- Reopening a child under terminal ancestors requires reopening those ancestors in the same transaction.
- Hierarchy depth is at most 16 and cycles are forbidden.

Use `due_date` for deadlines and `planned_for` for intended work dates. Do not infer either from the other.

## Visibility

Declared and effective visibility are `private` or `reportable`. A new ordinary Project defaults to `reportable` when omitted; explicit `private` remains valid and `project-inbox` remains private. A new Task or Event copies the current effective visibility of its project or parent task; it is not dynamic inheritance. Existing objects are never rewritten when a creation default changes.

A task is effectively reportable only when its declaration, derived project, and complete parent chain are reportable. An event also requires its project and optional task chain to be reportable. Parent downgrades immediately cap descendants without rewriting them.

Never promote implicitly. Before a parent promotion that could expose descendants, run `visibility preview` and include its current `exposure_digest` in the update.

## Events and corrections

Event types are `progress`, `decision`, `blocker`, `result`, and `note`. Events always link a project and may link a task. A date-only backfill uses local midnight plus `time_precision: date`; never present it as an exact midnight event.

Correction appends a new event with a complete replacement view and preserves the original. Transition evidence only permits summary/body correction or visibility downgrade. Use `task.compensate-transition` for a wrong state change; compensation appends the precise reverse transition and preserves both audit records.

## Reports

Facts apply corrections, reconstruct historical task status from `initial_status` plus transition evidence, and filter with effective visibility. Evidence-rich facts include visible Project and Task bodies and add the visible ancestor chain of relevant Tasks as `context_task_ids`; these context Tasks explain a workstream but do not imply activity. Reports never read Reports as facts. Personal and reportable files are written together, carry evidence-backed digests, and are never sent automatically. A clean export is a transient projection of a digest-valid Report: it does not create another Vault object, change report history, or weaken the audience boundary.
