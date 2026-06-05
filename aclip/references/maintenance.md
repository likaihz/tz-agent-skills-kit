# Retrieval and Maintenance

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

Use resource IDs when available. If the user does not provide an ID, search with `aclip list` and ask only when multiple plausible matches remain.

If an `aclip add` result has `duplicate: true`, run:

```bash
aclip show <id>
```

Report the existing resource instead of treating it as a fresh capture.

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

After capture or update, run `aclip show <id>` and report:

```text
已保存：<title>
状态：<capture_status> / <analysis_status>
分类：<content_class>
用途：<usage>
路径：<path>
```
