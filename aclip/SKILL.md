---
name: aclip
description: Use when the user wants to save, ingest, classify, retrieve, update, retry, archive, delete, or recommend captured reading resources with Aclip and Obsidian.
---

# Aclip

## Core Rule

Save first, analyze second. Do not summarize, classify, or recommend from a live URL until Aclip has persisted the URL and capture result.

Use the `aclip` CLI for deterministic operations whenever a command exists. Do not edit Aclip notes by hand unless the CLI cannot perform the requested operation.

## Setup Workflow

Before capture, ensure Aclip has a vault. If `aclip add`, `aclip list`, or `aclip show` reports that no vault is configured, ask for the intended vault path and run:

```bash
aclip init-vault <path> --set-default
```

For WeChat Official Account and Zhihu capture, ensure OpenCLI is installed and connected:

```bash
aclip deps opencli install
aclip deps opencli check
```

If install returns `connected: false`, tell the user to confirm the OpenCLI Chrome extension install in the opened Chrome page, then rerun `aclip deps opencli check`.

## Capture Workflow

When the user provides one or more links:

1. Decide whether the request is normal capture, later-reading capture, or capture with explicit topics/tags.
2. Run `aclip add <url>` for each link.
3. Add `--later` when the user intends later reading.
4. Pass user-provided topics and tags with `--topic` and `--tag`.
5. Read the CLI JSON result and keep the resource `id` and `path`.
6. If the result has `duplicate: true`, run `aclip show <id>` and report the existing resource instead of treating it as a fresh capture.
7. For WeChat Official Account and Zhihu links, prefer browser capture through OpenCLI. If capture fails because OpenCLI may be missing, the daemon may be stopped, or the Chrome extension may not be connected, run `aclip deps opencli install`, ask the user to confirm the Chrome extension if needed, then run `aclip deps opencli check`.
8. `aclip add` defaults to `--capture-provider auto`, which tries OpenCLI for WeChat/Zhihu and falls back to direct HTTP. Use `--capture-provider opencli` when direct HTTP is known to be useless, or `--capture-provider direct` when you explicitly want the old deterministic probe.
9. Treat direct HTTP capture as an opportunistic probe only. WeChat pages may return script-heavy or hidden bootstrap HTML. Zhihu pages may return a `zse-ck` anti-bot challenge or HTTP 403. Do not trust direct capture unless the CLI returns `capture_status: captured` and the saved content is visibly article/answer text.
10. If `capture_status` is `failed` or `partially_captured`, retry once with `aclip retry <id> --capture-provider opencli` for WeChat/Zhihu. Zhihu answers may require the connected Chrome profile to be logged in to Zhihu.
11. If retry is still incomplete, prefer browser/manual ingest: open the page in the user's browser context, copy visible article/answer text or rendered HTML, then use `aclip ingest <id> --html-file <path>`, `--text-file <path>`, or `--from-clipboard`.
12. Before reporting title, class, usage, reason, or preview, run `aclip show <id>` because `aclip add` returns capture status but not the full resource metadata.
13. Analyze only the saved note content after enough content exists.

## Analysis Rules

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

## Retrieval Workflow

For later-reading or recommendation requests, default to:

```bash
aclip list --usage later_read --state to_read --read-status unread
```

Narrow with the safest matching filters:

- `--topic`
- `--tag`
- `--class`
- `--platform`
- `--priority`

Present concise results with title, class, topics/tags, reason, and note path. Use `aclip show <id>` when the user selects an item.

## Update and Maintenance

Use resource IDs when available. If the user does not provide an ID, search with `aclip list` and ask only when multiple plausible matches remain.

Common operations:

```bash
aclip show <id>
aclip update <id> --priority high
aclip update <id> --class tool
aclip update <id> --usage reusable --topic "AI Agent" --tag important
aclip update <id> --state to_read
aclip update <id> --read-status reading
aclip mark-read <id>
aclip archive <id>
aclip retry <id> --capture-provider opencli
aclip delete <id>
```

For failed captures, preserve the note. Offer retry first, then browser/manual ingest.

After capture or update, run `aclip show <id>` and report:

```text
已保存：<title>
状态：<capture_status> / <analysis_status>
分类：<content_class>
用途：<usage>
路径：<path>
```
