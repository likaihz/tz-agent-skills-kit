# Analysis Rules

Save first, analyze second. Analyze only saved note content after enough content exists.

When updating analysis, produce a JSON payload and apply it with:

```bash
aclip update-analysis <id> --json-file <path>
```

## Initial Analysis

After a successful capture, perform initial analysis by default before reporting the resource as finished. The minimum useful payload must update at least `content_class` and `usage`; include `summary`, `collection_reason`, and `possible_uses` when the saved content supports them.

Do not leave successful captures at `content_class: unknown` when the saved content can be classified. Do not leave successful captures at `usage: quick_capture` when the content has a clearer durable use.

Use `quick_capture` only when the user explicitly wants a transient save, the captured content is too thin to judge, or the item has no evident later-reading or reuse value.

For `usage`, choose from the user's intent and the content type:

- If the user explicitly asked to read later, include `later_read`.
- If `content_class` is `methodology`, `tech_update`, `case_study`, or `opinion`, include `later_read` by default because these usually require deliberate reading.
- If `content_class` is `reference`, `tool`, or `tip`, default to a retrievable collection use such as `reusable`, `verify_later`, or `inspiration` instead of `later_read`, unless the user requested later reading.
- If the item is mainly a factual source to check later, use `verify_later`.
- If the item is a reusable method, tool, checklist, or pattern, use `reusable`.
- If the item is mainly idea material, use `inspiration`.

When classification is uncertain, use `content_class: unknown`, `analysis_status: needs_review`, and `needs_manual_review: true`.

Do not invent enum values.

Use exactly one `content_class`:

- `tip`
- `methodology`
- `tech_update`
- `tutorial`
- `case_study`
- `opinion`
- `reference`
- `tool`
- `question`
- `unknown`

Use only these `usage` values:

- `later_read`
- `quick_capture`
- `reusable`
- `verify_later`
- `inspiration`
- `archive_only`

Use `content_class: unknown`, `analysis_status: needs_review`, and `needs_manual_review: true` when classification is uncertain.
