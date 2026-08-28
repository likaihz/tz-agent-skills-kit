---
name: weekly-report
description: Generate, inspect, backfill, regenerate, or cleanly export evidence-backed personal and reportable weekly work reports with a one-sentence summary, structured progress, and selected idea/insight reflections. Use for “写周报”, “总结本周工作”, existing-report inspection/export, or a scheduled weekly-report wakeup.
---

# Weekly Report

Generate personal and privacy-filtered reportable Markdown through `work-ledger`.

Read [references/report-format.md](references/report-format.md) before drafting.

## Read-Only Inspect

For an existing report, require week and explicit audience; use `work-ledger report export --week <YYYY-Www> --audience personal|reportable --format markdown|text`, warning before personal export. Do not run `report facts` or `report write`. Return and stop. Only regenerate or write for generation, backfill, regeneration, or a scheduled due run.

## Workflow

1. Route through `work-ledger`; require CLI `>=0.11.0,<1.0.0`, protocol 1, Vault schema 7, `rich_report_facts=true`, `knowledge_documents=true`, `knowledge_kind_document=true`, and `knowledge_kind_memo=true`. Use `skill-setup` before facts or writes when incompatible.
2. Determine the ISO week. Scheduled runs first call `report due` and process `missing`, `incomplete_pair`, and `stale_facts`; leave `modified` for user choice.
3. Run `doctor` and stop the affected week on fatal data or transaction findings.
4. Request personal and reportable facts independently. Never filter personal facts in the Agent to construct the reportable package.
5. Map personal Project, Task, Event, and Knowledge bodies plus parent/source context into progress workstreams. Separately collect thought candidates from the exact Events in `derived.idea_event_ids` and `derived.insight_event_ids`, optionally enriched by Knowledge linked to those Events.
6. Map and draft reportable facts independently. Never reuse personal prose or reveal omitted thought candidates.
7. Draft exactly three sections: one-sentence summary, structured progress, and selected thoughts. Add an evidence marker immediately after the summary and every factual numbered item; validate all IDs against that audience's facts.
8. Write both bodies with one `report write` operation. Use `conflict_policy: fail` unless the user explicitly chooses replacement or a candidate copy.
9. Return both Obsidian paths, commit, and push state. Do not send the report.

## Writing Rules

- Follow the exact three headings, one-sentence constraint, labeled progress fields, thought selection, numbering, evidence, empty-section, and audience rules in `references/report-format.md`.
- Write the summary as exactly one sentence in one paragraph. State the most important supported outcome or stage change; do not join several status bullets into a sentence-shaped list.
- Synthesize one progress item per workstream. Structure it with `结果/阶段`, `关键推进`, and only fact-supported optional `验证/影响` or `风险/下一步` fields. Merge repetitive Events plus related Knowledge body/source Event context.
- Build `本周思考` from selected `idea` and `insight` Events. Polish them into natural reflection prose without Event-type labels or field-name prefixes. Preserve the source meaning and degree of certainty; do not turn a possibility or hypothesis into an established conclusion. Use linked Knowledge only as supporting context, not as a fixed Knowledge section.
- Prefer 3–6 progress items and 1–3 thought items only when facts support them. Do not fill quotas or turn every Event into an item.
- Qualify draft Knowledge as “草稿” or “在研”. `updated_at` does not prove weekly activity.
- Do not use a Task/Knowledge title or Event summary unchanged as finished prose. Do not invent impact, completion, commitments, or risk.
- Project date ranges and `next_week_task_ids` remain context only; do not add a fourth `下周计划` section. Mention a risk or next step inside its progress item only when the current audience's facts explicitly support it.
- Never reuse personal prose for the reportable draft. Cite only IDs present in that audience's facts; a Knowledge ID is not admitted by a source or wikilink.
- Both personal and reportable reports use exactly the same three-section shell. Personal facts may support private reflections; reportable output must neither expose nor count omitted private ideas, insights, Knowledge, or diagnostics.

## Export

For a copyable report, require CLI `>=0.11.0,<1.0.0`, `report.export`, and `clean_report_export=true`. Set `--audience` and `--format markdown|text`; never copy the managed Report. Prefer `reportable`; warn before exporting `personal`. Export is read-only and does not send.

## Regeneration

Use `latest` for current corrections and metadata; use `pinned` only to reproduce an explicitly requested source commit. For externally modified reports, ask whether to replace, save a candidate, or stop; never merge silently.
