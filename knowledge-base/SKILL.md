---
name: knowledge-base
description: Use when a user wants durable Knowledge for 调研、对比、技术笔记、随笔、项目文档、备忘录、笔记或知识沉淀, or explicitly asks to 更新、归档、恢复 existing Knowledge.
---

# Knowledge Base

**REQUIRED SUB-SKILL:** Use `work-ledger` for every query and mutation.

## Workflow

1. Verify `work-ledger-cli` `>=0.11.0,<1.0.0`, protocol 1, Vault schema 7, `features.knowledge_documents=true`, `features.knowledge_kind_document=true`, and `features.knowledge_kind_memo=true`. Route incompatibility through `skill-setup`; do not touch the Vault.
2. For update, archive, or restore, run `resolve --type knowledge --query <text>` before mutation; add `--status archived` when restoring. Accept only an ID or exact title/slug directly. Show fuzzy or ambiguous candidates and ask.
3. Resolve the optional Project and every source Event. Inspect effective/audit content. Convert corrections to origin Event IDs. Keep compensated origins linked, surface `KNOWLEDGE_SOURCE_SUPPRESSED`, and let the user decide source changes.
4. Ask only when source ambiguity changes document meaning. Otherwise choose deterministically and disclose it. Never silently choose meaningfully different candidates.
5. Choose one kind and a flexible template. Unless explicitly requested otherwise, use default draft status and default private visibility.
6. For explicit `reportable`, obtain confirmation after an exposure preview. Manually assemble this user-facing exposure preview from read-only Knowledge, Project, and effective source-chain results. Never run `visibility preview --type knowledge`; it supports only Project/Task. Normal `knowledge.create`/`knowledge.update` include no `exposure_digest`; invalid closure fails with `VISIBILITY_CONFLICT`.
7. Prepare one atomic request, obtain an operation ID immediately before submission, then use one atomic apply. For updates, `knowledge show --id <id>` returns response field `revision`; copy it into mutation key `expected_revision`. Replace sources completely and include a non-empty `reason` for status, source, Project, or visibility changes.
8. Return the Knowledge ID, stable path, status, resolved source IDs, commit, and push state or warning.

### Related Event and Knowledge creation

For a new Event plus Knowledge, the one atomic request contains multiple mutations. Place `event.add` first, give it a typed `ref`, and use that backward typed ref in `source_event_refs`. Do not split the apply.

### Lifecycle and identity

- `stable` requires a complete, non-empty body; reject empty content even when asked to finalize.
- Restore `archived` to `draft` only. Promotion requires a later explicit update.
- A title change preserves the immutable slug/path. Return that stable path instead of renaming a file.

## Flexible Templates

These are writing aids, not schema-required fields. Remove empty sections; they are optional and not schema-required.

### `research`

```markdown
## Question
## Findings
## Evidence
## Open questions
```

### `comparison`

```markdown
## Decision context
## Options
## Trade-offs
## Recommendation
```

### `technical_note`

```markdown
## Context
## Mechanism
## Constraints
## Verification
```

### `essay`

```markdown
## Thesis
## Discussion
## Implications
```

### `document`

Use this for structured project artifacts whose primary purpose is durable reference or maintenance, such as project briefs, specifications, SOPs, runbooks, handoff notes, and operating guides. Prefer `technical_note` when the primary purpose is explaining a technical mechanism, practice, or troubleshooting lesson; prefer `note` for unstructured capture. When importing existing Markdown, preserve its useful section structure instead of wrapping it in this template, and remove only a duplicate leading H1 that matches the Knowledge title.

```markdown
## Overview
## Content
## Related material
```

### `memo`

Use this for personal facts that must be retained, looked up repeatedly, and
maintained as reality changes: cloud-resource inventories, subscription and
renewal ledgers, domains, licenses, devices, and warranty records. Prefer
`note` for one-off unstructured capture, `document` for a project artifact or
operating procedure, and a Task for anything that must be done by a deadline.
Default to private. Never store passwords, tokens, private keys, or complete
payment credentials; keep only masked identifiers or a reference to an external
secrets manager.

```markdown
## Overview
## Entries
## Verification
```

### `note`

```markdown
## Note
## Follow-ups
```

## Hard Boundaries

- Never edit or rename `Work/Knowledge/*.md` directly.
- Never infer `reportable` from a Project.
- Never promote `draft` to `stable` automatically.
- Never change Task status because Knowledge changed.
- Never silently choose an unresolved source candidate when it changes document meaning.
- Never fetch external material automatically; require explicit authorization.
- Never invent sources, weaken visibility closure, or hide a suppressed-source warning.
- Treat "archive this document into the knowledge base" as durable capture, not as a request for Knowledge status `archived`; archive the lifecycle only when the user means the document is retired.
