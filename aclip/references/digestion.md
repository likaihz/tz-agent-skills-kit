# Digestion Workflow

Use this workflow when the user wants to read, digest, or receive a recommendation from saved Aclip resources.

Common triggers:

- "给我推一篇"
- "随机找一篇我收藏的内容"
- "找一篇关于 <topic> 的稍后读"
- "今天读点什么"
- "我想消化一下之前收藏的内容"

## Candidate Query

Start from unread later-reading resources:

```bash
aclip list --usage later_read --state to_read --read-status unread
```

If the user specifies a topic, tag, class, platform, or priority, narrow with the safest matching filter:

```bash
aclip list --usage later_read --state to_read --read-status unread --topic <topic>
aclip list --usage later_read --state to_read --read-status unread --tag <tag>
aclip list --usage later_read --state to_read --read-status unread --class <class>
aclip list --usage later_read --state to_read --read-status unread --platform <platform>
aclip list --usage later_read --state to_read --read-status unread --priority <priority>
```

If exact metadata filters return no useful candidates, run the default query and choose by title, summary, collection reason, topics, tags, and content preview.

## Picking Strategy

If the user asks for random selection, pick one random item from the candidate set. If the user asks for a recommendation, prefer high-priority, high-confidence, analyzed resources with a clear summary or collection reason.

For topic-based reading, prefer resources whose topic, tag, title, summary, or content preview is visibly related to the requested topic.

After choosing a candidate, inspect it:

```bash
aclip show <id>
```

## Reading Card

Present a concise reading card before deeper discussion:

```text
推荐阅读：<title>
为什么推这篇：<collection_reason>
类型：<content_class>
用途：<usage>
建议阅读关注点：
1. <focus>
2. <focus>
路径：<path>
id: <id>
```

Do not over-summarize before the user asks to go deeper. The goal is to help the user decide what to read.

## After Reading

If the user says the item has been read, mark it read:

```bash
aclip mark-read <id>
```

If the user says the item should be archived, archive it:

```bash
aclip archive <id>
```

If the user extracts new conclusions, update the resource through `aclip update-analysis <id> --json-file <path>` rather than editing the note by hand.
