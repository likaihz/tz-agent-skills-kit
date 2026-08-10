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
- `VISIBILITY_EXPOSURE_REQUIRED`: run a fresh visibility preview, show exposure, and obtain explicit confirmation.
- `EXPOSURE_PREVIEW_STALE`: rerun preview.
- `COMPENSATION_CONFLICT`: inspect audit history and construct an explicit new sequence.
- `REPORT_MODIFIED`: ask whether to replace, save a candidate, or stop.

## Git outcomes

- `GIT_PUSH_PENDING`: local commit succeeded; retry only `sync` later.
- `REMOTE_DIVERGED`: stop automatic synchronization. Do not pull, merge, rebase, or force-push. Ask the user or use explicit Git tooling to reconcile.
- `GIT_COMMIT`: the helper rolls back its business file changes. Inspect hooks, identity, and repository state before retrying.

## Doctor

Run `doctor` after a crash, schema error, setup drift, or Git problem. Treat fatal findings as a write barrier. Doctor is read-only and does not silently repair business facts.

An unfinished transaction requires explicit recovery. Do not use broad reset or checkout commands; limit any repair to the manifest paths and preserve unrelated worktree changes.

If the executable is missing, the CLI version or protocol is incompatible, or a scheduled path has drifted, stop before reading or writing the Vault and use `skill-setup`. Do not fall back to a script inside the Skill directory.

## Migration

Run `migrate plan` first and verify `plan_digest`. For schema 1 or 2, inspect every project/task `from`/`to` path, target collision, affected file digest, and unmanaged-note link risk. For schema 1, 2, or 3, inspect every `child_project_migrations` entry before upgrading to schema 4. Migration must stop with `MIGRATION_CONFLICT` when a child stores a project different from its parent's project; never silently choose which membership wins. Apply only the exact forward plan. Never invent a downgrade or rewrite Git history; restore old data through a new commit when necessary.
