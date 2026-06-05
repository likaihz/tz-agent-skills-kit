# Recall Workflow

Use this workflow when the user is trying to find or synthesize saved knowledge.

Common triggers:

- "我记得之前收藏了一篇关于 <topic> 的文章"
- "找找我之前关于 <topic> 的收藏"
- "我是不是存过某个关于 <topic> 的知乎回答"
- "帮我总结整合下我关于 <topic> 的收藏"

## Remembered-Item Lookup

For remembered-item lookup, extract possible metadata from the user's wording:

- topic
- tag
- content class
- source platform
- source type
- title words
- author or publication hints

Search metadata first:

```bash
aclip list --topic <topic>
aclip list --tag <tag>
aclip list --class <class>
aclip list --platform <platform>
```

If the user asks about later-reading items, include:

```bash
aclip list --usage later_read --state to_read --read-status unread
```

If metadata filters are too narrow or return nothing, run a broader query:

```bash
aclip list
```

Then rank candidates by title, summary, collection reason, possible uses, topics, tags, platform, and content preview.

When CLI metadata is insufficient, use read-only vault search with `rg`. Do not modify notes during this fallback.

```bash
rg -n "<keyword>" <vault-path>
```

## Candidate Reporting

Report candidates with uncertainty. Do not present weak matches as certain hits.

```text
我找到 <n> 个可能相关的收藏：

1. <title>
   相关原因：<why this candidate matches>
   置信度：high|medium|low
   id: <id>
   path: <path>
```

Run `aclip show <id>` before giving detailed analysis of any candidate.

## Collection Synthesis

Use collection synthesis when the user asks to summarize, integrate, compare, or extract conclusions from saved resources about a topic.

1. Recall candidate resources with the lookup flow above.
2. Pick a focused candidate set; if there are too many, present the set and ask whether to narrow.
3. Run `aclip show <id>` for each selected resource.
4. Synthesize only from saved Aclip content, not live pages.
5. Include source ids or paths in the answer.

Synthesis output should include:

- topic overview
- source list
- agreements
- differences
- reusable conclusions
- open questions

If recall is sparse, say what was found and what is missing instead of filling gaps from memory.
