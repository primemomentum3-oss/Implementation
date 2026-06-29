# Agent-Facing MCP Tool Specification

## Purpose

This document defines the intended normal agent-facing MCP tools for the Agent OS redesign. It focuses on what each tool means, when agents use it, what it should return at a product level, and what responsibilities should stay outside the tool.

This is not a low-level implementation spec. Final schemas should be designed from the current codebase when implementation starts.

## Design Principles

### Simple Names

Tool names should be short and practical. The normal agent-facing surface should not require `agent_os_` prefixes or backend-domain wording.

### One Clear Job Per Tool

Each tool should have a specific responsibility. Avoid tools that mix project navigation, context reading, evidence, reports, and task state into one broad output.

### Agent OS Owns Proof

Agents do work. Agent OS records proof/evidence automatically where possible. Agents should not be asked to manually maintain the proof system.

### Task Context Is Central

`get_task` is the main continuation tool. It should help an agent understand the current task without needing to reconstruct history from many raw backend lists.

### Backend/Admin Tools Stay Separate

Backend/admin/setup/explicit-scope tools can still exist, but they should not be part of the normal agent path unless a task explicitly requires that system-level work.

## Confirmed Tool List

- `list_tools`
- `list_tasks`
- `create_task`
- `get_task`
- `list_files`
- `search_files`
- `read_files`
- `get_commands`
- `get_checks`
- `add_observation`
- `request_review`

## Tool Details

### `list_tools`

Purpose:

Show the agent what normal agent-facing tools are available in the current context.

Used when:

- the agent starts work and needs to know the available MCP surface.
- the agent is unsure which tool should be used next.
- the system needs to show a clean tool list without backend/admin clutter.

Should return:

- available agent-facing tools.
- short purpose for each tool.
- whether a tool requires a current task.
- whether a tool is read-only or action-producing.
- simple guidance about when to use each tool.

Should not return:

- every backend/admin/global tool by default.
- raw tool governance internals.
- long technical policy dumps.

Current-system lesson:

The current `agent_os_list_available_project_tools` is useful, but it exposes too much backend/admin categorization for the normal path. The new `list_tools` should be cleaner and agent-facing first.

### `list_tasks`

Purpose:

Show available tasks as lightweight task summaries.

Used when:

- the agent needs to choose or inspect existing tasks.
- the agent is not yet attached to a task.
- the user asks what tasks exist.

Should return:

- task id.
- title.
- status/stage in simple language.
- short goal/summary.
- updated time.
- whether the task is active, blocked, review-ready, or done.

Should not return:

- full reports.
- raw evidence lists.
- full backend task records.
- huge nested objects.

Design note:

`list_tasks` is for scanning. `get_task` is for depth.

### `create_task`

Purpose:

Create a task record for meaningful work.

Used when:

- the agent is given new meaningful project work and no suitable task exists.
- Agent OS/user flow decides a new task should be created.

Should return:

- created task id.
- title.
- goal.
- initial status/stage.
- whether the current runtime/session/pane is attached, if applicable.
- next recommended tool, usually `get_task`.

Should not require the agent to understand:

- project/repository ids in normal managed context.
- runtime/pane binding internals.
- proof backfill internals.

Important pending dependency:

Stig will provide more information later about future Agent OS task creation/attachment behavior. Do not finalize low-level behavior until that direction is received.

Current-system lesson:

Current-project task creation currently showed a structural weakness in tests: agents tried to create tasks, but the current-project create path failed while broader explicit-scope task creation could work. The redesign should make the normal create/attach path unambiguous and reliable.

### `get_task`

Purpose:

Get the full useful task context for the current/selected task.

Used when:

- the agent starts or resumes a task.
- the agent needs to understand previous work.
- the agent needs attached files/context.
- the agent needs latest report/summary/continuation state.

Should return:

- task id/title/status/goal.
- attached task context.
- latest task summary/report.
- previous agent work summary.
- files already attached/relevant to the task.
- files previous agents read/changed/worked on.
- context expansion summary: extra files searched/read beyond the initial task context.
- checks already run and their results.
- known blockers/risks/open questions.
- next suggested step.
- lightweight evidence/proof summary when useful.

Useful context-tool inspiration:

- include context packet/source revision when available.
- include whether the task context appears narrow, multi-area, or broad.
- include warnings when the context route seems low-confidence, too broad, missing likely tests, or missing verification.

Should not return:

- unfiltered raw backend dumps.
- old terminal chat text as if it were guaranteed proof.
- every evidence/artifact row unless specifically needed.

Replaces/absorbs normal usage of:

- `get_task_context`
- `get_task_report`
- parts of `get_current_task_handoff`
- parts of evidence/history lists for continuation purposes

Important decision:

Stig decided there should not be a separate normal `get_task_context`. The context belongs inside `get_task`.

Decision history:

`get_task_context` was considered as an Agent OS-prepared task context tool, then narrowed to only return already-attached context, and finally removed from the normal agent-facing surface. The final product direction is simpler: agents call `get_task` for task details, attached context, latest summary/report, and continuation state.

### `list_files`

Purpose:

List project files/folders.

Used when:

- the agent needs a project map.
- the agent needs to understand folder structure.
- the agent wants to decide where to search next.

Should return:

- file/folder paths.
- simple type indicators.
- generated/runtime/source classification where known.
- optionally a bounded tree or filtered map.

Useful context-tool inspiration:

- classify generated/runtime folders such as `node_modules`, `.git`, `dist`, `build`, `coverage`, `.agent-os`, and generated output.
- identify likely source, test, config, database, package manifest, and README files.
- keep output bounded and scannable.

Should not return:

- file contents.
- guessed relevance.
- large raw filesystem dumps without limits.

Important decision:

Stig said `list_files` should only list files. It should not blend into context retrieval.

### `search_files`

Purpose:

Find files or content relevant to the task.

Used when:

- relevant files are not already clear from `get_task`.
- the agent needs to locate code/docs by name or content.
- the agent needs to narrow the next `read_files` call.

Should return:

- matching file paths.
- short match snippets or reasons.
- enough context to decide what to read.
- optional match source such as path match, symbol match, package script connection, direct import, test-for-file, or context route.

Useful context-tool inspiration:

- source indexing can improve search by path, file kind, exported symbols, tests, entrypoints, package scripts, and simple dependency edges.
- first version can be simpler than the skeleton, but it should keep enough metadata to explain why a file was suggested.

Should not return:

- full file contents.
- broad unbounded result dumps.
- hidden/generated folders unless explicitly allowed.

Important decision:

When relevant files are unclear, the path is `search_files` then `read_files`.

### `read_files`

Purpose:

Read selected project files.

Used when:

- the agent has selected files from `get_task`, `list_files`, or `search_files`.
- the agent needs actual contents to work.

Should return:

- requested file contents.
- clear file path labels.
- truncation/limit notices if content is too large.
- maybe file metadata if helpful.

Should automatically record:

- which files were read.
- when they were read.
- task/runtime association if available.
- whether the file was in the initial task context or discovered through expansion.
- a short reason/source when available, such as `attached_context`, `search_result`, `direct_import`, `test_for_file`, or `user_requested`.

Should not:

- read arbitrary hidden/generated/runtime paths by default.
- silently skip files without explanation.
- be treated as strong proof of correctness by itself.
- block normal project context expansion just because a file was not in the initial task context.

Expansion rule:

Normal expansion should be allowed and recorded. A blocked path is different from an unsuggested path. Generated/runtime/secrets/destructive areas can be blocked or require stronger policy, but ordinary project files should be expandable.

Important decision:

Agent OS should automatically record which files agents read.

### `get_commands`

Purpose:

Show useful commands for working in the project.

Used when:

- the agent needs to know how to start, build, run, or inspect the project.
- the agent needs safe command guidance.

Should return:

- command name.
- command text.
- short purpose.
- when to use it.
- safety notes if needed.
- source of the command, such as package script, project docs, current context, or history.

Useful context-tool inspiration:

- package scripts can be indexed and classified as test, lint, build, typecheck, dev, script, or unknown.
- command suggestions should stay separate from verification requirements.

Should not:

- decide which checks are required for completion.
- run commands.
- record command evidence manually.

Boundary:

`get_commands` gives useful commands. `get_checks` owns verification expectations.

Managed-terminal bridge:

A managed-terminal work package may include suggested commands from this command knowledge, but suggested commands are not proof. Proof comes from recorded command/check evidence interpreted by Agent OS.

### `get_checks`

Purpose:

Show required/recommended/optional checks for the current task/project.

Used when:

- before or after implementation to know what must be verified.
- before requesting review.
- when an agent needs to understand quality gates.

Should return:

- project checks.
- task-relevant checks.
- required/recommended/optional marking.
- check commands or manual verification descriptions.
- simple explanation of when each check applies.
- source of each check, such as Agent OS setting, project docs, package script, test history, or fallback inference.

Useful context-tool inspiration:

- classify package scripts into likely test/lint/build/typecheck checks.
- connect likely tests to files when possible.
- surface confidence plainly instead of pretending weak inference is certain.

Should not:

- run the checks.
- manually record evidence.
- hide project-level checks just because the task is narrow.

Important decision:

Show all project checks, but mark which are recommended/required for the current task.

Managed-terminal bridge:

`get_checks` is the agent-facing way to understand verification expectations. The managed-terminal work package should select the active `quality gate` from the same check source, project setting, or project documentation. The quality gate is the main proof expectation for the managed run.

Quality rule:

A check suggestion is not proof. Proof requires recorded command/check evidence, or an accepted exception recorded by Agent OS.

### `add_observation`

Purpose:

Add a weak manual observation to the current task when Agent OS cannot capture the information automatically.

Used when:

- Stig gives a clarification in chat.
- an assumption/risk should be preserved.
- a dependency on another instance or future design should be noted.
- something useful happened outside automatic capture.

Should return:

- observation id.
- task id.
- summary saved.
- quality/status label showing this is weak/manual.

Should not:

- be called normal evidence.
- replace automatic proof.
- mark work as verified.
- become approval-ready proof by itself.

Design language:

Use `add_observation`, not `record_evidence`, because Stig wants normal evidence automatic and the manual tool to feel smaller and clearer.

### `request_review`

Purpose:

Let an agent say the work is ready for review.

Used when:

- implementation/investigation work is complete enough for review.
- required checks have been run or missing checks are explained.
- the task should move to a review path.

Should return:

- review request id/status.
- task id.
- what Agent OS recorded as readiness information.
- next step for review/approval.

Should not:

- mark task final done.
- bypass required approval.
- pretend checks passed if they did not run.
- bypass the Result Finalizer for managed terminal work.

Important decision:

Agents should not directly mark tasks as final done. They request review; Agent OS/review/Stig approval controls final completion.

Managed-terminal bridge:

For managed terminal work, `request_review` should mean: the agent believes work is ready, so Agent OS should run or consult the Result Finalizer and then route to Needs repair, Needs review, Ready for approval, Blocked, or Failed.

## Tools To Remove From Normal Agent Surface

### `record_evidence`

Remove from everyday agent-facing flow.

Reason:

Evidence should be automatic. Manual notes should use `add_observation` and be labeled weak/manual.

### `submit_patch`

Remove from everyday agent-facing flow.

Reason:

Patch/file-change evidence should be recorded automatically by Agent OS. If a future sandbox/work-area design needs an explicit submit step, define it with that design later.

Implementation dependency:

Removing `submit_patch` from the normal surface does not remove the need for patch/changelist proof. The backend or sandbox/work-area system must still capture changed files, diffs, validation errors, and safe artifacts automatically or internally.

### `list_evidence`

Remove from everyday agent-facing flow.

Reason:

Agents should get useful continuation context through `get_task`, not browse raw evidence lists as a normal step. Stig/UI may need proof views separately.

### `get_task_report`

Remove as normal separate tool.

Reason:

Latest task report/summary belongs inside `get_task` for continuation.

### `get_task_context`

Remove as normal separate tool.

Reason:

Stig decided `get_task` is the clearer name and should include task context.

## Normal Agent Workflow Summary

Expected normal flow:

1. Agent starts work.
2. Agent OS/task system creates or attaches a task according to the future task design.
3. Agent calls `get_task` to understand the task and continuation context.
4. Agent uses `list_files`, `search_files`, and `read_files` to inspect project files.
5. Agent uses `get_commands` and `get_checks` to understand work commands and verification expectations.
6. Agent does the work.
7. Agent OS records evidence automatically.
8. Agent uses `add_observation` only for uncaptured useful context.
9. Agent uses `request_review` when ready.

## Open Dependencies

- final task creation/attachment design from Stig.
- sandbox/work-area design from Stig/other instance.
- final context storage design.
- UI visibility decisions for how Stig sees evidence/observations/history.
