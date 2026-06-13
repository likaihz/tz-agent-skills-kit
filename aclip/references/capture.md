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

## Douyin Video Capture

For Douyin or 抖音 shares, preserve the original share text and use the CLI.

Default Douyin media engine is `douyin_downloader`; do not assume `yt-dlp` is primary. Use `--media-provider yt_dlp` only as explicit fallback/debug for legacy behavior.

Before the first Douyin video capture, check dependencies:

```bash
aclip deps douyin-downloader check
aclip deps ffmpeg check
aclip deps transcriber check
```

If `douyin-downloader` is missing, install it and check again:

```bash
aclip deps douyin-downloader install --yes
aclip deps douyin-downloader check
```

The preferred Douyin sidecar install is a pip install from GitHub into Aclip's
managed virtualenv, exposing the `douyin-dl` console script. A local source
checkout with `run.py` is fallback/debug compatibility only.

Transcriber installation is provider-specific. For the first supported local provider, use the `whisper-cpp` checks, install commands, and model commands:

```bash
aclip deps whisper-cpp check
aclip deps ffmpeg install --yes
aclip deps whisper-cpp install --yes
aclip deps whisper-cpp install-model --model small
aclip deps whisper-cpp list-models
```

`aclip deps transcriber check` is an aggregate health check for configured transcription providers, not an install command.

To remove optional Douyin dependencies, ask Aclip for uninstall guidance:

```bash
aclip deps douyin-downloader uninstall
aclip deps ffmpeg uninstall
aclip deps whisper-cpp uninstall
aclip deps whisper-cpp uninstall-model --model small
```

Use `aclip add "<share text or URL>"`. If the share is an opaque Douyin command without a visible URL, report the CLI `next_step` and ask the user to copy a visible link from Douyin before retrying.

For cookie JSON, Aclip automatically uses the managed default path when it exists:

```bash
~/.local/share/aclip/tools/douyin-downloader/config/cookies.json
```

To override the default cookie JSON, pass:

```bash
aclip add "<share text or URL>" --douyin-cookies-file <path>
```

For real Douyin e2e validation with the default engine, use a douyin-downloader
JSON cookie file. If the run only partially captures media, inspect the saved
`douyin-downloader.stdout.txt` and `douyin-downloader.stderr.txt` first; missing
`ttwid`, `odin_tt`, or `passport_csrf_token` plus `Empty 200 response` means
Douyin anti-bot blocked the detail API and fresh cookies are required.

Use the legacy provider only when explicitly debugging fallback behavior:

```bash
aclip add "<share text or URL>" --media-provider yt_dlp --no-transcribe
aclip retry <id> --force-media --media-provider yt_dlp --no-transcribe
```

For transcription, keep the default `--transcription-provider auto` unless the user asks for a specific engine or diagnostics point to one. Use `--transcription-provider whisper_cpp` for local whisper.cpp, `--transcription-provider openai` for OpenAI Speech to Text, and `--transcription-provider manual` or `--no-transcribe` when the user wants to save media without transcript extraction.

Optional transcription flags apply to both `aclip add` and `aclip retry`:

```bash
aclip add "<share text or URL>" --transcription-provider whisper_cpp --transcription-model small --transcription-language zh
aclip add "<share text or URL>" --transcription-provider openai --transcription-model gpt-4o-mini-transcribe --transcription-language zh
aclip add "<share text or URL>" --no-transcribe
aclip retry <id> --force-media --transcription-provider whisper_cpp --transcription-model small --transcription-language zh
aclip retry <id> --force-media --no-transcribe
```

Use `aclip retry <id> --force-media` when a saved Douyin resource should refresh or repair downloaded media/transcript artifacts instead of reusing existing media state.

Douyin video transcript content must be read from the saved Aclip note or `transcript_path`, not from the live page.
