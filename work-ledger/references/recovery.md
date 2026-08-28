# Recovery and Diagnostics

## Contents

1. Safe retries
2. Conflicts
3. Git outcomes
4. Doctor
5. Migration

## Safe retries

Retry the identical write with the same `operation_id`. A changed request must use a new ID. `OPERATION_CONFLICT` means the ID was reused for different content.

Do not retry a mutation when the response is `ok=true` and the commit exists, even if push is pending or diverged.

## Conflicts

- `CONFLICT`: refresh the object and its revision; do not overwrite unseen changes.
- `AMBIGUOUS`: show candidates and ask the user.
- `VISIBILITY_EXPOSURE_REQUIRED` for Project/Task promotion: run the standalone `visibility preview`, show its exposure, obtain explicit confirmation, then submit the update with that digest.
- Event correction exposure uses an embedded preview: submit the complete `event.correct` without `exposure_digest`, show the returned exposure, obtain confirmation, then retry that unchanged complete correction with the returned digest.
- `EXPOSURE_PREVIEW_STALE`: repeat the matching flow—standalone preview for Project/Task, or the complete correction without a digest for `event.correct`—and never reuse the stale digest.
- `COMPENSATION_CONFLICT`: inspect audit history and construct an explicit new sequence.
- `REPORT_MODIFIED`: ask whether to replace, save a candidate, or stop.
- `KNOWLEDGE_SLUG_CONFLICT`: choose a different slug at create time; never move an existing managed file.
- `INVALID_KNOWLEDGE_SOURCE`: correct missing, correction-ID, forward-ref, wrong-kind, or malformed source input.
- `VISIBILITY_CONFLICT`: keep the Knowledge private or explicitly resolve every private Project/Event dependency; never silently downgrade or promote.

## Git outcomes

- `GIT_PUSH_PENDING`: local commit succeeded; retry only `sync` later.
- `REMOTE_DIVERGED`: stop automatic synchronization. Do not pull, merge, rebase, or force-push. Ask the user or use explicit Git tooling to reconcile.
- `GIT_COMMIT`: the helper rolls back its business file changes. Inspect hooks, identity, and repository state before retrying.

## Doctor

Run `doctor` after a crash, schema error, setup drift, or Git problem. Treat fatal findings as a write barrier. Doctor is read-only and does not silently repair business facts. `KNOWLEDGE_SOURCE_SUPPRESSED` is a warning: inspect the compensated origin and update sources explicitly when needed; do not auto-archive or hide the Knowledge. Knowledge visibility-closure and digest-coverage findings are errors.

An unfinished transaction requires explicit recovery. Do not use broad reset or checkout commands; limit any repair to the manifest paths and preserve unrelated worktree changes.

If the executable is missing, the CLI version or protocol is incompatible, or a scheduled path has drifted, stop before reading or writing the Vault and use `skill-setup`. Do not fall back to a script inside the Skill directory.

## Migration

Run `migrate plan` first and verify `plan_digest`. For schema 1 or 2, inspect every project/task `from`/`to` path, target collision, affected file digest, and unmanaged-note link risk. For schema 1, 2, or 3, inspect every `child_project_migrations` entry before the schema 4 step. Migration must stop with `MIGRATION_CONFLICT` when a child stores a project different from its parent's project; never silently choose which membership wins.

The schema 4 to schema 5 step is `4->5-knowledge-documents`. It may claim an absent `Work/Knowledge/` directory or one containing no Markdown. Any `.md` file there, including nested or mixed-case suffixes, returns `MIGRATION_CONFLICT` with sorted relative paths and leaves the schema-4 marker and files unchanged. Do not parse, import, re-key, merge, or overwrite those pre-existing notes. Non-Markdown files remain untouched. The final schema 5 to schema 6 step is `5->6-document-knowledge-kind`; it changes only the Vault marker and must preserve every existing Knowledge byte. Apply only the exact forward plan to 6 with request `schema_version: 1`, a fresh operation ID, `to_version: 6`, and the reviewed digest. Never invent a downgrade or rewrite Git history; restore old data through a new commit when necessary.
