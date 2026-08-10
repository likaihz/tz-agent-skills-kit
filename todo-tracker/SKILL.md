---
name: todo-tracker
description: Create, organize, date, query, reprioritize, schedule, transition, reopen, cancel, or archive projects and tasks in the user's work ledger. Use for requests such as “记个待办”, “这个项目从八月做到十月”, “开始做 X”, “X 卡住了”, “完成 X”, “调整优先级”, “安排到下周”, “拆成子任务”, “把任务移到另一个项目”, “有哪些进行中的任务”, or “这个任务误标完成了”. Manage project ranges, task state, and hierarchy through work-ledger; route mere records of what happened without a requested state change to work-log.
---

# Todo Tracker

Manage project ranges and task identity, state, priority, schedule, hierarchy, and project membership through `work-ledger`. Never edit Vault Markdown directly.

## Workflow

1. Determine whether the request needs a tracked task. Use an event instead for a one-off action with no future state, priority, schedule, or hierarchy.
2. Route execution through the `work-ledger` Skill so it verifies the standalone CLI version, protocol, and capabilities. Stop before any Vault write and use `skill-setup` when the runtime is unavailable or incompatible.
3. Read the CLI/data references when exact request fields or transitions are needed.
4. Resolve named projects and tasks deterministically. Ask the user when multiple candidates remain.
5. Query `show` immediately before an update and include the returned `revision`.
6. Obtain a new operation ID, write one atomic apply request, execute it, and parse the JSON result.
7. Return the affected title, Obsidian wikilink, new state or fields, commit, and any push warning in plain language.

## Creation

Extract title, either a project or a parent task, priority, initial status, visibility, project start/end dates, task deadline, planned date, tags, and useful body.

- Default priority to `P2`.
- Default status to `planned` when intent is actionable; use `inbox` when explicitly captured for later organization.
- Use `project-inbox` only when the user explicitly wants to classify later.
- When creating an ordinary Project and the user does not specify visibility, omit `visibility` so work-ledger applies the `reportable` default. Pass `private` only when the user requests it or the content is explicitly personal.
- Reuse a unique project from context; otherwise ask instead of inventing one.
- Choose a concise, durable, unique project title at creation because it becomes the permanent Obsidian note name.
- Choose a concise, durable, globally unique task title at creation because it becomes the permanent Obsidian note name.
- Never rename a project or include `title` in `project.update`.
- Never rename a task or include `title` in `task.update`.
- Create related project/task/event objects in one apply request using local refs.
- A root task requires `project_id` or `project_ref` and has no parent.
- A child task requires `parent_id` or `parent_ref`; omit `project_id` and `project_ref` because project membership is inherited recursively.
- Do not create cross-project parent-child relationships. Make the task a root in the other project when it needs independent project membership.
- Reparenting under a task changes the whole subtree's effective project to the new parent's project. Detaching a child materializes its previously inherited project on the new root.

## Project Dates

- Use nullable ISO `start_date` and `end_date` for the project's inclusive business range.
- Allow either boundary to remain unknown and clear a boundary only when the user asks.
- Require `start_date <= end_date` when both are present.
- Do not infer project dates from task `planned_for`, task `due_date`, event timestamps, fiscal labels, or the current date.
- Do not archive a project or reject an out-of-range task or event merely because a boundary is reached.
- Keep both dates unset for `project-inbox`.

## State Intent

- “开始” → transition to `in_progress` with a progress summary.
- “卡住” → transition to `blocked` with the blocker reason.
- “完成” → transition to `done` with the concrete result.
- “取消” → transition to `cancelled` with a reason.
- “重新打开” → transition to an allowed active state with a reason.
- Mere progress language → record through `work-log`; do not change state.

Do not infer a state change without explicit status intent. Do not auto-complete parents or cascade cancellation. When reopening a child under terminal ancestors, include each required ancestor reopen in the same transaction.

## Corrections and Visibility

Use transition compensation for an erroneous state change; do not rewrite its event. Keep the repair and any correct-task transition in one apply request when possible.

Never promote an existing private object implicitly. Show a visibility preview and obtain explicit confirmation before submitting an exposure digest. The creation default for a new Project does not authorize changing historical objects.
