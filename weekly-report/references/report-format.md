# Weekly Report Format

## Contents

1. Shared narrative
2. Personal report
3. Reportable report
4. Synthesis
5. Clean export
6. Evidence
7. Fact boundaries
8. Scheduled behavior

## Shared narrative

Use this main structure without a report title:

```markdown
**本周进展：**

（1）完成或推进某个工作流……
  <!-- evidence: task-...,event-... -->

**下周计划：**

（1）完成某个目标……
  <!-- evidence: task-... -->
```

Each numbered item is one physical Markdown paragraph followed immediately by its evidence marker. Use consecutive full-width numbering. Prefer 3–6 progress items and 2–4 plan items only when facts support them. An empty required section contains `无已记录事项。` without numbering or evidence.

## Personal report

Use the shared `本周进展` and `下周计划` narrative. Include private workstreams, decisions, risks, diagnostics, and unclassified work only when useful to the owner. When diagnostics do not fit a workstream, append an optional `**内部补充：**` section after the shared narrative. Retain useful Obsidian links.

## Reportable report

Use exactly the two shared headings and no others. Use only the reportable facts package. Emphasize supported outcomes, scope, mechanisms, verification, and next steps; omit personal diagnostics and inbox events. Place a decision or risk in the workstream item it affects instead of creating another section. If the package is empty, place `本周无已标记为可上报的事项。` under `本周进展`.

## Synthesis

Before writing, build an internal evidence map grouped by Project and workstream. Use Project and Task bodies plus `context_task_ids` and parent links to understand scope. A workstream may span one parent Task, related child Tasks, or several Events that describe the same delivered capability or resolved problem.

For each workstream:

1. Identify the supported problem or target.
2. Merge implementation and investigation Events that led to the same result.
3. Prefer the latest verified result over superseded progress statements.
4. Add supported verification, decision, risk, or next step from Event bodies and transitions.
5. Write the smallest number of numbered items that still preserves meaningful context.

A strong item starts with an outcome or stage conclusion and usually connects two or more supported elements: scope, action or mechanism, verification, decision, risk, or next step. Keep all facts for that workstream in the same item. If facts support only one element, state only that element; never pad with invented impact.

Do not:

- use a Task title or Event summary unchanged as the final item;
- turn each Event into a separate chronological item;
- repeat the same workstream in an overview, outcome, progress, decision, or risk section;
- claim impact, adoption, completion, or commitments absent from the facts package.

## Clean export

Managed Report files retain frontmatter, evidence comments, and Obsidian wikilinks for validation and navigation. Never remove them from the source Report merely to make it sendable.

Use `report export` for a copyable projection:

- `markdown` keeps headings, lists, and ordinary Markdown while removing machine metadata and reducing wikilinks to visible labels.
- `text` also removes Markdown presentation syntax for chat or mail paste.
- `reportable` is the default audience for sending.
- `personal` requires a private-content warning.

## Evidence

Place a marker immediately after each factual numbered item:

```markdown
（1）完成 work-ledger 状态机与事务实现。[[Work/Tasks/完成 work-ledger|完成 work-ledger]]
  <!-- evidence: task-...,event-... -->
```

- Completed results cite a result event or task with a done transition.
- Blockers cite a blocker event or blocked task.
- Next-week items cite an ID from `next_week_task_ids`.
- `（1）`, `(1)`, and `1.` items are all validated by the CLI, but authored reports use full-width `（1）` numbering.
- Section placeholders need no evidence and must not use a numbered prefix.
- Never cite an ID absent from the current audience's facts package.

## Fact boundaries

- Facts are authoritative; prose may summarize but not add claims.
- A date-precision event has no known exact time.
- A task completed and reopened within the week is not a completed result at period end.
- A due date without next-week `planned_for` is not a commitment.
- Compare personal and reportable `next_week_task_ids` only to detect a visibility gap. Never derive reportable prose from the personal package or reveal omitted private titles in the reportable report.
- A project start or end date does not by itself prove progress, completion, impact, or a next-week commitment.
- Reportable prose must not reveal the existence, title, relationship, or diagnostics of private objects.

## Scheduled behavior

The intended schedule is Friday 21:00 in Asia/Shanghai. A wakeup calls `report due` across the configured eight-week lookback so missed weeks can be backfilled. Scheduled runs do not wait for ambiguity or visibility decisions, never promote objects, and never send reports automatically. Report omitted next-week plan counts for later review without exposing their private content.
