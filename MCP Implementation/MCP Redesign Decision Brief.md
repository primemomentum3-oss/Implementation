# MCP Redesign Decision Brief

## Purpose

This document captures the confirmed product direction for the Agent OS MCP redesign from the working session with Stig. It is meant to let a future agent quickly understand what has been decided, what is still pending, and how to align new MCP work with the intended Agent OS direction.

The redesign is focused on the MCP tools that agents use while doing project work. It is not primarily a backend/admin cleanup, UI redesign, or general documentation cleanup.

## Live Notes Distillation

The raw live notes from `/Users/codex/Desktop/MCP Redesign/mcp-tool-review-live-notes.md` have been reviewed and distilled into this package.

Important distilled points:

- `get_task_context` was an intermediate idea, then removed from the final normal agent-facing surface.
- `get_task` is the single normal task retrieval and continuation tool.
- `get_task` must include attached context, latest useful report/summary, previous agent work, files read/changed, checks run, blockers, risks, and next steps.
- `list_tasks` must stay lightweight and must not dump full backend task records or long final reports.
- current-project task creation/attachment has shown a failure path where task creation can return `null` or roll back because startup/runtime proof attachment fails.
- Agent OS already has several proof layers, but they are action/service dependent; the redesign must strengthen task/session binding so proof keeps attaching to the right task across the whole session.
- backend proof can exist while UI visibility remains weak; future UI/read-model work must make automatic evidence understandable to Stig.

## Product Direction

Agent OS should give agents better project work context, clearer task lifecycle support, simpler tool names, and stronger automatic recording of what happened during work.

The normal agent-facing MCP surface should be simple, direct, and easy to reason about. Agents should not have to choose between several similar backend/admin tools or understand low-level backend lifecycle details just to work on a task.

The MCP redesign should make the agent workflow feel like this:

1. The agent knows what task it is working on.
2. The agent can get the task, attached context, previous work, and continuation summary.
3. The agent can inspect project files through clear file tools.
4. The agent can see available commands and required/recommended checks.
5. Agent OS records evidence automatically.
6. The agent can add a weak manual observation only when something useful cannot be captured automatically.
7. The agent can request review when work is ready.
8. Final done/approval remains controlled by Agent OS/review/Stig approval rules, not by normal agents directly.

### MCP Redesign Before Managed Terminals

Stig decided the MCP redesign should be implemented before the managed-terminal control implementation.

Reason:

```text
Managed terminals depend on agents receiving clear task context, file access, checks, observations, and review actions. Cleaning the MCP surface first gives managed terminals a clearer control layer to build on.
```

Implementation implication:

- Complete the normal agent-facing MCP tool redesign first.
- Keep sandbox/work-area design pending unless separately decided.
- Ensure `get_task`, `read_files`, `get_checks`, `add_observation`, and `request_review` are shaped so managed-terminal work can use them later.

### Attention Overview Compatibility

Stig decided the current top-tab `Attention Overview` should later be fully redesigned into the Agent OS agent-control tab.

MCP implication:

- the clean agent-facing tools should support starting/continuing task work from Agent OS
- `get_task` should provide the continuation context the tab needs
- `get_checks` should provide check/quality-gate expectations
- `add_observation` should support weak manual notes from Agent OS-mediated work
- `request_review` should support the tab's review/finalizer path later

This does not make the MCP redesign a full UI redesign. It means MCP must be compatible with the later UI control surface.

## Confirmed Scope

In scope:

- Agent-facing MCP tools used by agents during normal project work.
- Tool naming and tool responsibility cleanup.
- Separation of agent-facing tools from backend/admin tools.
- Better task continuation context.
- Clearer file discovery/read flow.
- Automatic evidence direction.
- Review request as an agent-facing action.
- Notes and planning for future implementation.

Out of main scope for this decision set:

- Backend/admin Agent OS setup tools, unless they affect the normal agent workflow.
- Final sandbox/work-area tool design, because Stig said that design is coming separately.
- UI redesign. Evidence visibility matters later, but this MCP redesign is backend/tool focused.
- Full context storage redesign. Stig said this is also being worked on and will be combined later.
- Safety/tool-policy process details, unless they directly affect the agent-facing tool set.

## Confirmed Agent-Facing Tool List

The first clean agent-facing MCP tool list is:

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

These are the names and roles agreed for the normal agent-facing surface. Old `agent_os_*` names should not be treated as the final internal design. Stig explicitly said not to keep old names internally just because they exist today.

## Confirmed Removals From Normal Agent Flow

The following should not be everyday agent-facing tools in the redesigned normal flow:

- `record_evidence`
- `submit_patch`
- `list_evidence`
- separate `get_task_report`
- separate `get_task_context`

Reason:

- Evidence should be recorded automatically by Agent OS.
- Patch/proof/report details are backend responsibilities or read models, not things normal agents should manually manage.
- `get_task` should include the latest useful task summary/report and attached context, so agents do not need separate context/report tools for normal continuation.

## Key Decisions

### Agent-Facing vs Backend/Admin Tools

Decision:

Normal agents need a clean MCP surface. Backend/admin/setup tools can still exist, but they should not clutter the normal agent-facing flow.

Current examples that are backend/admin or explicit-scope tools:

- project creation/listing
- repository registration/listing
- global task tools with explicit project/repo ids
- governance tools
- launcher/internal runtime tools
- memory candidate/admin tools
- final approval/operator decision tools

Design implication:

The new MCP should have a clear boundary between:

- tools agents normally use while doing a task
- tools Agent OS/backend/admin uses to manage the system
- tools that require explicit operator/admin context

### Task Tool Direction

Decision:

Use `get_task` as the main rich task context tool.

`get_task` should include:

- task id/title/status/goal
- attached task context
- files already attached to the task
- latest useful task summary/report
- previous agent work summary
- files previous agents read/changed/worked on
- checks run and results if available
- open blockers/risks/next steps
- continuation context so a new agent can quickly get up to speed

Do not keep a separate normal `get_task_context` tool. Stig decided `get_task` is clearer.

### Context Tooling Adoption

Decision:

Use the downloaded context-tool skeleton as design inspiration, not as a direct implementation.

Useful concepts to adopt into existing Agent OS/MCP:

- source/file index ideas for better `list_files` and `search_files`.
- file-read recording for `read_files`.
- context expansion records for files searched/read beyond initial task context.
- packet/read-budget checks as backend quality signals.
- stale-context and source-ownership warnings as later context-maintenance guardrails.
- package-script extraction as one input to `get_commands` and `get_checks`.

Do not adopt directly:

- the separate clean-room database schema.
- a second `agent_sessions` or `file_leases` model.
- strict read blocking outside the initial packet.
- the skeleton keyword router as the source of truth for task context.
- the rule that excludes `docs/**`, because Agent OS currently depends on `docs/context` for agent routing.

Design implication:

The context-tool ideas should strengthen the confirmed MCP tools: `get_task`, `list_files`, `search_files`, `read_files`, `get_commands`, and `get_checks`. They should not create a parallel context system.

### File Tool Direction

Decision:

Use three separate file tools:

- `list_files`
- `search_files`
- `read_files`

Meanings:

- `list_files` lists the project file/folder map only.
- `search_files` finds files or content when relevant files are not clear.
- `read_files` reads selected files.

Important rule:

Do not have `list_files` guess relevance and jump directly to reading. Stig specifically corrected the flow: relevant files should go through `search_files` and then `read_files` when not already attached to the task.

Agent OS should automatically record which files agents read.

### Commands And Checks

Decision:

Use both `get_commands` and `get_checks`.

`get_commands`:

- shows useful project commands.
- examples: start app, build app, run migration, run dev server.
- does not own what must pass before a task is ready.

`get_checks`:

- shows project/task verification expectations.
- should show all project checks and mark which are required/recommended/optional for the current task.
- examples: test, lint, build, typecheck, manual UI check.

### Evidence Direction

Decision:

Agent OS should record evidence automatically. Normal agents should not be responsible for manually proving ordinary work.

Automatic evidence should cover things like:

- tool calls
- file reads
- file changes
- commands/checks run
- patch artifacts
- runtime runs
- trace events
- reports/reviews

Manual evidence should be only a small fallback through `add_observation`.

### Manual Observation

Decision:

Keep `add_observation` as a small manual fallback tool.

Purpose:

- attach useful information to a task that Agent OS cannot automatically capture.

Examples:

- Stig clarified that the MCP redesign is backend/tool focused, not UI focused.
- A task depends on sandbox design from another instance.
- The agent noticed a risk or assumption that should help the next agent.

Rules:

- `add_observation` is weak/manual.
- It is not strong proof.
- It should not replace automatic evidence.
- It should be visible in task history/continuation context.

### Review And Done

Decision:

Keep `request_review` as a simple agent-facing action.

Agents should be able to say work is ready for review, but should not directly mark tasks as final done.

Final done should be controlled by Agent OS workflow/review/Stig approval rules as appropriate.

### Sandbox/Work Area

Decision:

Leave sandbox/work-area tools pending.

Stig said sandbox design is being worked on separately. The MCP redesign should not finalize worktree/sandbox names until that information arrives.

Current placeholder idea:

- later this may become something like `create_sandbox`
- but this is not confirmed as final

## Structural Lessons From Current-System Test Outputs

Stig tested the current Agent OS copy with three small tasks and shared raw terminal/MCP outputs. These outputs are discovery material for the redesign, not a fix request.

Observed:

- Agents received startup context.
- Startup context said no task was attached.
- Agents generally called `agent_os_get_current_project_context` first.
- Agents generally tried to create a task before meaningful work.
- Current-project task creation failed in some tests.
- In one test, the agent used broader explicit-scope `agent_os_create_task`, which succeeded.
- Evidence/reporting worked when a task id existed.
- The new `docs/context` structure helped agents route to relevant files.

Design interpretation:

- The context-folder upgrade is useful and should be preserved as a routing layer.
- The current MCP tool surface is too cluttered and makes agents choose between similar tools.
- The new agent-facing tool path should avoid forcing agents to guess between current-project tools and global/explicit-scope tools.
- Task creation/attachment behavior will be redesigned later with more input from Stig, so do not overfit the MCP redesign to the current failing path.

## Important Product Constraint

Stig wants questions and decisions framed broadly and practically. Do not ask him to choose database details, library choices, schema names, or low-level implementation structures unless the answer visibly changes the product behavior, workflow, safety, or system outcome.

When asking Stig questions:

1. State the topic briefly.
2. Explain the practical choice simply.
3. Give concrete examples when useful.
4. Ask the actual question clearly.
5. Ask one question at a time unless Stig asks for a batch.

### Integration With Managed Terminal Control

The MCP redesign and managed-terminal control package must fit together.

Bridge rules:

- `get_checks` shows project/task verification expectations. A managed-terminal work package selects its active `quality gate` from those expectations or from the same project setting/documentation source.
- `get_commands` shows useful commands, but does not decide what proof is required.
- `request_review` lets the agent say work is ready for review. For managed terminal work, this must trigger or route through the Result Finalizer before review/approval.
- Removing `submit_patch` from the normal agent-facing MCP surface is safe only if patch/changelist evidence is captured automatically or by an internal backend/sandbox mechanism.
- `add_observation` is weak/manual and must not satisfy quality gates by itself.
- final done remains controlled by Agent OS workflow, review, and Stig/admin approval rules.

## Current Status

The broad agent-facing MCP tool set is mostly decided.

Still pending from Stig/future work:

- final task creation/attachment behavior
- sandbox/work-area design
- final context-storage design
- UI visibility decisions for evidence/history
- final implementation sequencing once current source is inspected again
