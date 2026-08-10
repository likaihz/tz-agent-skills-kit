---
name: work-log
description: Record, backfill, query, or correct timestamped work events in the user's Obsidian work ledger, including progress, decisions, blockers, results, and notes linked to a project and optionally a task. Use for requests such as “记录一下刚做的事”, “记一笔流水”, “今天处理了 X”, “记录这个决定”, “补记昨天的进展”, “刚完成一个小动作”, “这条关联错任务了”, or “今天做了什么”. Do not change task status unless the user explicitly requests a transition; route that part through todo-tracker.
---

# Work Log

Record what happened through `work-ledger` while leaving task state unchanged unless the user explicitly asks for a transition.

## Workflow

1. Route execution through the `work-ledger` Skill so it verifies the standalone CLI version, protocol, and capabilities. Stop before any Vault write and use `skill-setup` when the runtime is unavailable or incompatible.
2. Classify the event as `progress`, `decision`, `blocker`, `result`, or `note`.
3. Resolve the project and optional task. Prefer explicit IDs/titles, then a unique active task in context, then a unique project.
4. If task matching is ambiguous, either ask or record only against the unambiguous project when task linkage is not essential. State that the task link was omitted.
5. Derive visibility from the resolved task/project unless the user explicitly requests a valid downgrade or promotion.
6. Obtain an operation ID and submit one `event.add` mutation through `work-ledger`.
7. Return the event summary, time precision, links, commit, and push warning.

Project `start_date` and `end_date` are descriptive context. Do not reject or reassign an event only because its occurrence is outside that range, and do not mutate project dates while recording an event.

## Classification

- `progress`: meaningful advancement without final completion.
- `decision`: a choice or agreement that affects later work.
- `blocker`: a dependency, impediment, or inability to proceed.
- `result`: a concrete output or completed one-off action.
- `note`: useful context outside the other categories.

When `progress` versus `result` is unclear, prefer `progress`. Never complete a task as a side effect of classification.

## Time and Backfill

- Use `exact` when the user provides an exact time or refers to a just-completed event.
- Use `date` when only a date is known; encode local midnight but never claim the event occurred at midnight.
- Keep `recorded_at` as helper-controlled current time.
- Do not disguise a backfilled event as a real-time record.

## Correction

Resolve and inspect the original event, then append `event.correct` with a complete replacement view and reason. Show old and corrected associations to the user. Use task transition compensation instead when the error changed task state.
