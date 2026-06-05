# Analysis Rules

Save first, analyze second. Analyze only saved note content after enough content exists.

When updating analysis, produce a JSON payload and apply it with:

```bash
aclip update-analysis <id> --json-file <path>
```

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
