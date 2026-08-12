---
name: work-log
description: Use when the user asks to record, backfill, query, or correct a timestamped work event, including an unverified idea, reusable insight, progress, decision, blocker, result, note, wrong association, or “今天做了什么”.
---

# Work Log

Record what happened through `work-ledger` while leaving task state unchanged unless the user explicitly asks for a transition.

## Read-Only Query

For read-only history, use `work-ledger event list --from <inclusive> --to <exclusive>` with relevant filters; use `work-ledger event show --id <event-id> --view effective` for details. Do not obtain an operation ID and do not submit `event.add` or `event.correct`. Return and stop.

## Workflow

1. Route through `work-ledger` for runtime verification; use `skill-setup` before Vault access when incompatible.
2. Classify the event as `progress`, `decision`, `blocker`, `result`, `note`, `idea`, or `insight`.
3. Resolve Project/task by explicit object, Task-derived Project, unique active Task, then unique context Project; otherwise use protected private Inbox (`project-inbox`).
4. If task matching is ambiguous, either ask or record only against the unambiguous project when task linkage is not essential. State that the task link was omitted.
5. Derive visibility from the resolved task/project unless the user explicitly requests a valid downgrade or promotion.
6. Obtain an operation ID and submit one `event.add` mutation through `work-ledger`.
7. Return the event summary, time precision, links, commit, and push warning.

Every classification produces only an Event. Never create Knowledge from an `idea` or `insight`; that requires a separate explicit Knowledge request. Never transition a Task unless the user explicitly asks; route that transition through `todo-tracker`.

Project `start_date` and `end_date` are descriptive context. Do not reject or reassign an event only because its occurrence is outside that range, and do not mutate project dates while recording an event.

## Classification

- `progress`: meaningful advancement without final completion.
- `decision`: a choice or agreement that affects later work.
- `blocker`: a dependency, impediment, or inability to proceed.
- `result`: a concrete output or completed one-off action.
- `note`: useful context outside the other categories.
- `idea`: an unverified (`未验证`) possibility, question, direction, or hypothesis.
- `insight`: a confirmed reusable (`可复用`) judgment, pattern, lesson, or conclusion.

When `idea` versus `insight` is hard to distinguish (`难分`), prefer `idea`. An adopted choice remains `decision`; a completed deliverable remains `result`; ordinary background remains `note`. When `progress` versus `result` is unclear, prefer `progress`. Never complete a task as a side effect of classification.

## Time and Backfill

- Use `exact` when the user provides an exact time or refers to a just-completed event.
- Use `date` when only a date is known; encode local midnight but never claim the event occurred at midnight.
- Keep `recorded_at` as helper-controlled current time.
- Do not disguise a backfilled event as a real-time record.

## Correction

Resolve and inspect the original event, then append `event.correct` with a complete replacement view and reason. Show old and corrected associations to the user. Use task transition compensation instead when the error changed task state.
