---
name: weekly-report
description: Use when generating, inspecting, backfilling, regenerating, or cleanly exporting a personal or reportable weekly work report, including “写周报”, “总结本周工作”, or a scheduled wakeup.
---

# Weekly Report

Generate personal and privacy-filtered reportable Markdown through `work-ledger`.

Read [references/report-format.md](references/report-format.md) before drafting.

## Read-Only Inspect

For an existing report, require week and explicit audience; use `work-ledger report export --week <YYYY-Www> --audience personal|reportable --format markdown|text`, warning before personal export. Do not run `report facts` or `report write`. Return and stop. Only regenerate or write for generation, backfill, regeneration, or a scheduled due run.

## Workflow

1. Route through `work-ledger`; require CLI `>=0.11.0,<1.0.0`, protocol 1, `rich_report_facts=true`, and `knowledge_documents=true`. Use `skill-setup` before facts or writes when incompatible.
2. Determine the ISO week. Scheduled runs first call `report due` and process `missing`, `incomplete_pair`, and `stale_facts`; leave `modified` for user choice.
3. Run `doctor` and stop the affected week on fatal data or transaction findings.
4. Request personal and reportable facts independently. Never filter personal facts in the Agent to construct the reportable package.
5. Map personal Project, Task, Event, and Knowledge bodies plus parent/source context into workstreams.
6. Map and draft reportable facts independently. Never reuse personal prose. Compare only `next_week_task_ids` sets for visibility gaps; never copy omitted content.
7. Add an evidence marker immediately after every factual numbered item and validate all referenced IDs against that audience's facts.
8. Write both bodies with one `report write` operation. Use `conflict_policy: fail` unless the user explicitly chooses replacement or a candidate copy.
9. Return both Obsidian paths, commit, and push state. Do not send the report.

## Writing Rules

- Follow the exact headings, numbering, synthesis, evidence, empty-section, and audience rules in `references/report-format.md`.
- Synthesize one item per workstream. Merge repetitive Events plus related Knowledge body/source Event context; never add a fixed Knowledge section.
- Qualify draft Knowledge as “草稿” or “在研”. `updated_at` does not prove weekly activity.
- Do not use a Task/Knowledge title or Event summary unchanged as finished prose. Do not invent impact, completion, commitments, or risk.
- Treat next-week `planned_for` as the only automatic plan source. Project date ranges are context only.
- Never reuse personal prose for the reportable draft. Cite only IDs present in that audience's facts; a Knowledge ID is not admitted by a source or wikilink.
- On interactive visibility gaps, request review without promotion. Scheduled runs leave excluded plans empty and report only their count.

## Export

For a copyable report, require CLI `>=0.11.0,<1.0.0`, `report.export`, and `clean_report_export=true`. Set `--audience` and `--format markdown|text`; never copy the managed Report. Prefer `reportable`; warn before exporting `personal`. Export is read-only and does not send.

## Regeneration

Use `latest` for current corrections and metadata; use `pinned` only to reproduce an explicitly requested source commit. For externally modified reports, ask whether to replace, save a candidate, or stop; never merge silently.
