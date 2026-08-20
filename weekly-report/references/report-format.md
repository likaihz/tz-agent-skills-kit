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
**一句话总结：**

本周围绕一个核心目标取得最重要的结果或阶段变化。
  <!-- evidence: task-...,event-... -->

**本周进展：**

（1）**工作流：** ……；**结果/阶段：** ……；**关键推进：** ……；**验证/影响：** ……；**风险/下一步：** ……
  <!-- evidence: task-...,event-... -->

**本周思考：**

（1）围绕……，本周进一步意识到……，这也提示……。
  <!-- evidence: event-...,knowledge-... -->
```

Use exactly these three headings and no report title or fourth section.

The summary is exactly one non-list sentence in one physical paragraph, ending with one sentence-final `。`, `！`, or `？`. It states the most important supported outcome or stage change instead of compressing several bullets with semicolons. Follow it immediately with one evidence marker. If the audience has no usable facts, write `本周无已记录的有效进展。`; for an empty reportable package write `本周无已标记为可上报的事项。`. Empty-summary placeholders need no evidence.

Each progress or thought item is one physical Markdown paragraph followed immediately by its evidence marker. Use consecutive full-width numbering independently within each numbered section. Prefer 3–6 progress items and 1–3 thought items only when facts support them. An empty progress section contains `无已记录事项。`; an empty thought section contains `本周无已记录的灵感或洞察。`. Empty placeholders have no numbering or evidence.

Progress items use labeled clauses rather than nested Markdown bullets, so one evidence marker covers the complete item. `工作流`, `结果/阶段`, and `关键推进` are the normal core. Include `验证/影响` and `风险/下一步` only when the facts explicitly support them; omit unsupported labels rather than writing filler such as “暂无”.

Thought items are polished, natural reflection paragraphs without labels such as `灵感（待验证）`, `洞察`, `来源/触发`, or `启发`. Smooth out ledger-style wording and connect the trigger, reflection, and implication when facts support them. Preserve the source meaning and degree of certainty: a possibility or hypothesis must remain tentative even though the final prose does not expose its Event type.

## Personal report

Use exactly the shared three-section narrative. Include private workstreams and private `idea`/`insight` Events only when useful to the owner. Fold a relevant decision, risk, or diagnostic into its progress item; omit unrelated diagnostics rather than appending another heading. Retain useful Obsidian links.

## Reportable report

Use exactly the three shared headings and no others. Draft every section independently from the reportable facts package. Emphasize supported outcomes, scope, mechanisms, verification, and useful reportable insights; omit personal diagnostics and inbox Events. Never reveal or count private thoughts that were excluded. If the package is empty, use the reportable empty summary plus the standard empty progress and thought placeholders.

## Synthesis

Before writing, build two internal maps for the current audience:

1. A progress evidence map grouped by Project and workstream. Use Project, Task, Event, and Knowledge bodies plus `context_task_ids`, parent links, and Knowledge source Event links to understand scope.
2. A thought candidate map whose entry points are the exact Events listed in `derived.idea_event_ids` and `derived.insight_event_ids`. Join optional Knowledge only when its effective source chain links it to one of those Events.

A workstream may span one parent Task, related child Tasks, several Events, and related Knowledge that describe the same delivered capability or resolved problem. Knowledge is fact material, not a fixed section.

For each workstream:

1. Identify the supported problem or target.
2. Merge implementation and investigation Events that led to the same result.
3. Prefer the latest verified result over superseded progress statements.
4. Fill the structured progress clauses from supported scope, mechanism, verification, decision, risk, or next-step facts; omit absent clauses.
5. Merge a Knowledge body with its effective source Event in that same workstream; avoid repeating the same claim as separate Event and Knowledge items.
6. Write the smallest number of numbered items that still preserves meaningful context.

A strong progress item starts with an outcome or stage conclusion and usually connects two or more supported elements: scope, action or mechanism, verification, decision, risk, or next step. Keep all facts for that workstream in the same item. If facts support only one element, state only that element; never pad with invented impact.

After progress synthesis, select only the most useful thought candidates. Rewrite Event bodies and linked Knowledge into fluent reflection prose rather than copying titles, summaries, or internal type names. Preserve meaning and epistemic strength while polishing: express a tentative source as a possibility or question in natural language, and an established learning as a supported reflection. If the same evidence supports both an operational result and a learning, make the progress item describe what changed and the thought item describe the reusable principle; do not repeat the same sentence.

Do not:

- use a Task title or Event summary unchanged as the final item;
- use a Knowledge title unchanged as the final item;
- present draft Knowledge as finished work or a stable conclusion; qualify it as “草稿” or “在研”;
- create a fixed Knowledge section or use Knowledge without a qualifying `idea`/`insight` source as a thought candidate;
- turn each Event into a separate chronological item;
- repeat the same operational claim in the summary, progress, and thought sections;
- treat a `progress`, `result`, `decision`, or `note` Event as a thought solely to fill the section;
- present an `idea` as an established conclusion, decision, delivery, or commitment;
- claim impact, adoption, completion, or commitments absent from the facts package.

## Clean export

Managed Report files retain frontmatter, evidence comments, and Obsidian wikilinks for validation and navigation. Never remove them from the source Report merely to make it sendable.

Use `report export` for a copyable projection:

- `markdown` keeps headings, lists, and ordinary Markdown while removing machine metadata and reducing wikilinks to visible labels.
- `text` also removes Markdown presentation syntax for chat or mail paste.
- `reportable` is the default audience for sending.
- `personal` requires a private-content warning.

## Evidence

Place a marker immediately after the factual summary sentence and each factual numbered item:

```markdown
（1）完成 work-ledger 状态机与事务实现。[[Work/Tasks/完成 work-ledger|完成 work-ledger]]
  <!-- evidence: task-...,event-... -->
```

- The one-sentence summary cites the evidence for its central claim.
- Completed results cite a result Event or Task with a done transition.
- Blockers cite a blocker event or blocked task.
- Thought items cite at least one exact `idea` or `insight` Event from the current audience's derived ID sets. Cite linked Knowledge only when that exact Knowledge ID is present in the same facts package.
- `（1）`, `(1)`, and `1.` items are all validated by the CLI, but authored reports use full-width `（1）` numbering.
- Section placeholders need no evidence and must not use a numbered prefix.
- Never cite an ID absent from the current audience's facts package.
- A Knowledge ID is valid evidence only when that exact ID appears in the current audience's `knowledge` facts collection. A source Event ID does not implicitly admit its Knowledge, and a Knowledge ID does not implicitly admit a source outside the package.

## Fact boundaries

- Facts are authoritative; prose may summarize but not add claims.
- A Knowledge body is supporting fact material. Its `updated_at` does not prove weekly activity; period inclusion must come from creation or an effective in-period source Event as represented by the facts package.
- Draft Knowledge requires qualified wording such as “草稿” or “在研”; it does not establish completion, publication, or a stable conclusion.
- A date-precision event has no known exact time.
- A task completed and reopened within the week is not a completed result at period end.
- A due date or `next_week_task_ids` membership does not require a standalone plan section or prove a commitment.
- Never reuse personal prose in the reportable draft, derive reportable prose from the personal package, or reveal omitted private Event/Knowledge titles, bodies, relationships, counts, or diagnostics.
- A project start or end date does not by itself prove progress, completion, impact, or a commitment.
- Reportable prose must not reveal the existence, title, relationship, or diagnostics of private objects.

## Scheduled behavior

The intended schedule is Friday 21:00 in Asia/Shanghai. A wakeup calls `report due` across the configured eight-week lookback so missed weeks can be backfilled. Scheduled runs do not wait for ambiguity or visibility decisions, never promote objects, and never send reports automatically. They use the same three-section shell and leave unsupported thoughts empty rather than inventing reflection.
