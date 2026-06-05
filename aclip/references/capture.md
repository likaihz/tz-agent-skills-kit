# Capture Workflow

When the user provides one or more links:

1. Decide whether the request is normal capture, later-reading capture, or capture with explicit topics/tags.
2. Run `aclip add <url>` for each link.
3. Add `--later` when the user intends later reading.
4. Pass user-provided topics and tags with `--topic` and `--tag`.
5. Read the CLI JSON result and keep the resource `id` and `path`.
6. If the result has `duplicate: true`, follow `maintenance.md`.
7. Before reporting title, class, usage, reason, or preview, run `aclip show <id>` because `aclip add` returns capture status but not the full resource metadata.
8. If `capture_status: captured` and the saved note has enough readable content, perform initial analysis from `analysis.md`.
9. Apply the initial analysis with `aclip update-analysis <id> --json-file <path>`.
10. Run `aclip show <id>` again before the final report.
11. Analyze only the saved note content after enough content exists.

Do not leave successful captures at `content_class: unknown` or `usage: quick_capture` when the saved content is sufficient for an initial judgment. Those values are acceptable only as temporary fallbacks when content is missing, incomplete, or explicitly just a transient quick capture.

## Provider Choice

`aclip add` defaults to `--capture-provider auto`, which tries OpenCLI for WeChat/Zhihu and falls back to direct HTTP.

Use `--capture-provider opencli` when direct HTTP is known to be useless, or `--capture-provider direct` when you explicitly want the deterministic direct probe.

Treat direct HTTP capture as an opportunistic probe only. WeChat pages may return script-heavy or hidden bootstrap HTML. Zhihu pages may return a `zse-ck` anti-bot challenge or HTTP 403. Do not trust direct capture unless the CLI returns `capture_status: captured` and the saved content is visibly article/answer text.

## WeChat and Zhihu

For WeChat Official Account and Zhihu links, prefer browser capture through OpenCLI. If capture fails because OpenCLI may be missing, the daemon may be stopped, or the Chrome extension may not be connected, run:

```bash
aclip deps opencli install
aclip deps opencli check
aclip doctor
```

Ask the user to confirm the Chrome extension if needed, then report returned `next_steps` for protected-site capture.

If `capture_status` is `failed` or `partially_captured`, retry once with OpenCLI for WeChat/Zhihu:

```bash
aclip retry <id> --capture-provider opencli
```

If the retry returns `zhihu_auth_required`, tell the user to log in to Zhihu in the Chrome profile connected to OpenCLI, then retry:

```bash
aclip retry <id> --capture-provider opencli
```

## Manual Ingest

If retry is still incomplete, prefer browser/manual ingest: open the page in the user's browser context, copy visible article/answer text or rendered HTML, then use one of:

```bash
aclip ingest <id> --html-file <path>
aclip ingest <id> --text-file <path>
aclip ingest <id> --from-clipboard
```

For failed captures, preserve the note. Offer retry first, then manual ingest.
