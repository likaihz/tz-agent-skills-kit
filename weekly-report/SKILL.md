---
name: weekly-report
description: Generate, inspect, backfill, regenerate, or export clean copies of personal and reportable weekly work reports from work-ledger facts. Use when the user asks to “写周报”, “总结本周工作”, “生成个人版和上报版”, “看看上周做了什么”, “补写漏掉的周报”, “重跑本周周报”, “导出可发送周报”, or when the Friday 21:00 Asia/Shanghai schedule wakes the Agent. Request independent fact packs, cite evidence IDs, save both versions atomically, and never invent facts or send the report automatically.
---

# Weekly Report

Generate two evidence-backed Markdown reports through `work-ledger`: a detailed personal version and a paste-ready, privacy-filtered reportable version.

Read [references/report-format.md](references/report-format.md) before drafting.

## Workflow

1. Route execution through the `work-ledger` Skill so it verifies CLI `>=0.9.0,<1.0.0`, protocol 1, and `features.rich_report_facts=true`. Stop before reading report facts or writing reports and use `skill-setup` when the runtime is unavailable or incompatible.
2. Determine the ISO week. Scheduled runs first call `report due` and process `missing`, `incomplete_pair`, and `stale_facts`; leave `modified` for user choice.
3. Run `doctor` and stop the affected week on fatal data or transaction findings.
4. Request personal and reportable facts independently. Never filter personal facts in the Agent to construct the reportable package.
5. Build a private evidence map for the personal package, using Project and Task bodies plus parent context to group facts into workstreams rather than raw event order.
6. Repeat the mapping and drafting independently for the reportable package. Never reuse personal prose as the reportable draft. Compare only the two `next_week_task_ids` sets to detect plans excluded by visibility; never copy omitted private titles or prose into the reportable draft.
7. Add an evidence marker immediately after every factual numbered item and validate all referenced IDs against that audience's facts.
8. Write both bodies with one `report write` operation. Use `conflict_policy: fail` unless the user explicitly chooses replacement or a candidate copy.
9. Return both Obsidian paths, commit, and push state. Do not send the report.

## Writing Rules

- Use `**本周进展：**` and `**下周计划：**` as the only reportable headings. Do not add a report title, overview, outcomes, decisions, or risk sections.
- Write each reportable item as a full-width numbered paragraph such as `（1）...`; do not use Markdown bullets. Prefer 3–6 progress items and 2–4 plan items when facts support them, but never pad to meet a count.
- Make each item represent one workstream. Lead with its outcome or stage conclusion, then connect supported scope, action or mechanism, verification, and any remaining risk or next step in that same paragraph.
- Merge repetitive events for the same outcome. A later result or verified state subsumes earlier implementation steps unless the steps explain a meaningful decision or risk.
- Use Project and Task `body`, parent context, Event `body`, and transition evidence when they add supported problem, action, verification, conclusion, or next-step detail.
- Do not copy a Task title or Event summary as the finished item, and do not produce a lightly rephrased event timeline or repeat one workstream across sections.
- Preserve useful project/task wikilinks.
- Distinguish completed results, active progress, decisions, blockers, and plans.
- Treat `planned_for` in the next ISO week as the only automatic next-week commitment source.
- Treat project `start_date` and `end_date` as context only; they do not prove progress, completion, impact, or a weekly commitment.
- Do not invent impact figures, completion state, commitments, risks, or missing context.
- Keep both required sections. For an empty section, write `无已记录事项。` without a number or evidence marker.
- Do not expose private objects or personal diagnostics in the reportable version.
- On an interactive run, warn when personal facts contain next-week IDs absent from reportable facts and ask whether the user wants to review visibility; never promote automatically. Scheduled runs leave the reportable plan empty and surface the count for later review.

## Export

When the user asks for a copyable or sendable report, require CLI `>=0.8.0,<1.0.0`, command `report.export`, and `features.clean_report_export=true`. Use explicit `--audience` and `--format markdown|text`; never copy the managed Report file verbatim. Prefer `reportable` for material intended to be sent. Warn before exporting `personal`, because it may contain private content. Export is read-only and does not prove that a report was sent.

## Regeneration

Use `latest` to reflect currently known corrections and task metadata. Use `pinned` only when the user explicitly asks to reproduce the original source commit. If a formal report was externally modified, ask whether to replace it, save a timestamped candidate, or stop; never merge silently.
