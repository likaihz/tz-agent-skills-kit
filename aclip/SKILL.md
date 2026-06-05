---
name: aclip
description: Use when the user wants to save, ingest, classify, retrieve, update, retry, archive, delete, or recommend captured reading resources with Aclip and Obsidian.
---

# Aclip

## Core Rule

Save first, analyze second. Do not summarize, classify, or recommend from a live URL until Aclip has persisted the URL and capture result.

Use the `aclip` CLI for deterministic operations whenever a command exists. Do not edit Aclip notes by hand unless the CLI cannot perform the requested operation.

If any `aclip` command returns JSON with `next_step` or `next_steps`, report those exact instructions before adding explanation.

## Routing

Read only the reference documents needed for the user's request:

- New environment, CLI installation, vault initialization, or first-resource validation: read `references/bootstrap.md`.
- CLI commands, arguments, JSON results, error handling, or command safety: read `references/cli.md`.
- URL capture, WeChat, Zhihu, OpenCLI, failed captures, retries, or manual ingest: read `references/capture.md`.
- Classification, usage, analysis metadata, or `update-analysis`: read `references/analysis.md`.
- Later-reading queues, retrieval, recommendation, updates, archive, delete, or duplicate handling: read `references/maintenance.md`.
- Reading recommendations, random later-reading picks, topic-based reading, or digesting unread resources: read `references/digestion.md`.
- Remembered-item lookup, fuzzy recall, or synthesis across saved resources: read `references/recall.md`.
- Applying saved knowledge during planning, design, research, implementation, debugging, or writing work: read `references/application.md`.

## Default Workflow

For a new user or uncertain environment:

1. Follow `references/bootstrap.md`.
2. Stop on vault/config errors and report the returned `next_steps`.
3. Do not move to protected-site setup until generic first-resource capture works unless the user explicitly asks for WeChat/Zhihu setup first.

For one or more links:

1. Save each URL first with the CLI.
2. Use `--later` when the user intends later reading.
3. Read the CLI JSON result and keep the resource `id` and `path`.
4. Run `aclip show <id>` before reporting title, class, usage, reason, or preview.
5. If capture succeeded and enough saved content exists, perform initial analysis from `references/analysis.md`.
6. Apply that analysis with `aclip update-analysis <id> --json-file <path>`.
7. Run `aclip show <id>` again and report the updated class and usage.
8. Analyze only saved note content after enough content exists.

For failed captures:

1. Preserve the note.
2. Follow `references/capture.md` for OpenCLI retry or manual ingest.
3. Report exact `next_step` or `next_steps` when present.

For classification or maintenance:

1. Use enums and commands from `references/analysis.md` and `references/maintenance.md`.
2. Do not invent enum values.
3. Prefer CLI operations over direct note edits.

For digestion, recall, or application:

1. Use the relevant workflow reference.
2. Base reasoning on saved Aclip content, not live pages.
3. Report candidate uncertainty when matches are weak.
4. Do not invent saved knowledge that was not found.
