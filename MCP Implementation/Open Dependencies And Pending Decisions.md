# Open Dependencies And Pending Decisions

## Purpose

This document lists the parts of the MCP redesign that are intentionally not finalized yet. Future agents should not treat these areas as decided unless Stig gives more direction or a later source-of-truth document supersedes this file.

## Confirmed Priority - MCP Before Managed Terminals

Status:

Decided by Stig.

Decision:

```text
Redesign the MCP tools before building the managed-terminal implementation.
```

Meaning:

- The clean 11-tool agent-facing MCP surface is the first implementation workstream.
- Managed-terminal work should later use or align with the redesigned MCP flow.
- This does not finalize sandbox/work-area design, context storage, or UI evidence visibility. Those remain separate/pending where noted.

## Pending Area 1 - Task Creation And Attachment

Status:

Pending Stig direction.

What is known:

- Stig wants stronger Agent OS-controlled task tracking.
- Current startup behavior tells agents to create/attach tasks, but Agent OS does not fully enforce it.
- Test outputs showed agents often tried to create tasks correctly.
- Current-project task creation/attach can fail in the current system.
- Live-note evidence showed a failure where current-project context had `task: null`, current-project create returned `null`, and the fallback showed PostgreSQL error `could not determine data type of parameter $2`.
- Stig said more information about future task creation behavior will be provided later.

Do not assume:

- current `agent_os_create_current_project_task` behavior is final.
- current startup instruction wording is final.
- agents should manually manage all task attachment steps.

Design direction already implied:

- Agent OS should make task creation/attachment more automatic and reliable.
- Implementation should inspect current-project task creation, runtime/pane attach, and pane runtime proof backfill before redesigning the normal `create_task` path.
- Agents should not silently continue meaningful work without a tracked task.
- Task state should be clear to agents and visible to Stig.

Question to ask later:

Task start behavior:
When an agent starts meaningful work, should Agent OS automatically create a task, attach to a matching existing task, or ask Stig when there is more than one possible match?

## Pending Area 2 - Sandbox / Work Area

Status:

Pending separate sandbox design.

What is known:

- Stig is working on sandbox/work-area behavior elsewhere.
- Current `worktree` tool language should not be treated as final.
- A possible future tool may be something like `create_sandbox`, but this is not confirmed.

Do not assume:

- final tool names.
- whether every task uses a sandbox.
- whether sandbox creation is automatic or agent-requested.
- whether patch submission remains a tool.

Design direction already implied:

- Agents should work in a controlled area when required.
- Evidence of file changes should be automatic.
- Normal agents should not need a manual `submit_patch` tool unless the sandbox design later requires a clear submit step.

Question to ask later:

Sandbox behavior:
When an agent needs to change files, should Agent OS always create a sandbox first, or only create one for implementation tasks where file changes are expected?

## Pending Area 3 - Context Storage

Status:

Partially discussed, not finalized.

What is known:

- Stig wants agent-created notes, planning, and working files kept out of the project structure when they are not actual project files.
- The project should stay clean.
- Context management is being upgraded separately.
- The current `docs/context` folder in Agent OS is useful and part of the newer context-routing model.

Do not assume:

- all agent-created notes belong in the repo.
- all context belongs outside the repo.
- current source documentation fully defines the future context-storage design.

Design direction already implied:

- Keep project source clean.
- Keep working notes/planning separate when they are not part of the product/project source.
- Preserve useful task continuation context for future agents.
- `get_task` should expose attached task context and continuation summaries.

Question to ask later:

Context storage:
Which agent-created material should live outside the project folder, and which material should remain inside the project because it is part of the product documentation or source of truth?

Examples:

- Outside project: temporary notes, planning drafts, working logs.
- Inside project: README updates, product docs, source context docs that future agents must use.

## Pending Area 4 - UI Evidence Visibility

Status:

Relevant later, not the main MCP redesign now.

What is known:

- The MCP redesign is backend/tool focused, not a UI redesign.
- Stig already has UI work in Agent OS.
- Evidence visibility matters because backend records are not useful if Stig cannot understand what happened.
- Current UI Proof & History can read proof-history bundles when a task is selected.
- Workbench summary may not clearly show every trace/evidence item.

Do not assume:

- UI needs a full redesign as part of MCP rename.
- every backend proof record should be shown directly in the main task list.

Design direction already implied:

- Stig should be able to see whether work was recorded.
- Manual observations should be distinguishable from automatic/strong evidence.
- Task history should be understandable without raw backend dumps.

Question to ask later:

Evidence visibility:
Should Stig see automatic evidence mainly as a simple task timeline, a detailed Proof & History view, or both?

## Pending Area 5 - Safety And Tool Policy Changes

Status:

Not finalized in this MCP tool-name discussion.

What is known:

- Safety/tool-policy changes should be controlled.
- Stig corrected that safety/tool policy can be within the same broad automatic upkeep process, but it needs control.
- The current task is mainly about MCP/tool flow, not final governance design.

Do not assume:

- agents can freely change safety rules.
- automatic upkeep means unrestricted policy mutation.

Design direction already implied:

- Normal agents should not manage backend/admin governance as part of everyday task work.
- Tool policy/governance should be separate from normal agent-facing tool use.

Question to ask later:

Safety changes:
When an agent finds that a tool policy or safety rule should change, should Agent OS create a controlled review item for Stig/system approval instead of letting the agent change it directly?

## Pending Area 6 - Compatibility / Old Names

Status:

Product direction clear, technical migration pending.

What is known:

- Stig does not want old names kept internally as the final design.
- Redesign means rename/restructure, not just add aliases.
- Temporary compatibility may be useful, but should not define the final model.

Do not assume:

- old `agent_os_*` names should remain as first-class internal names.
- old MCP docs should stay authoritative after redesign.

Design direction already implied:

- New names should be simple.
- Normal agent guidance should use new names only once migration is complete.
- Any compatibility aliases should be temporary and clearly marked.

Question to ask later only if needed:

Migration compatibility:
During the upgrade, should old tool names fail clearly, or should they temporarily forward to the new tools while warning that they are deprecated?

## Pending Area 7 - Final Output Shape Of `get_task`

Status:

Broad behavior decided, detailed schema pending.

What is known:

`get_task` should include:

- task details.
- attached task context.
- latest summary/report.
- previous agent work.
- files read/changed/worked on.
- checks run.
- blockers/risks/next steps.
- compressed continuation context.

Do not assume:

- raw proof-history output is the final `get_task` shape.
- every backend record belongs in the default response.

Design direction already implied:

- `get_task` should be useful to agents, not just complete as a database dump.
- Detailed proof/history can still exist elsewhere for debugging/UI.

Question to ask later if schema choices affect visible behavior:

Task continuation detail:
Should `get_task` default to a short continuation summary with optional expanded sections, or always return all available task detail at once?

## Pending Area 8 - MCP And Managed Terminal Integration

Status:

Partly decided, implementation details pending.

What is known:

- `get_checks` shows verification expectations to agents.
- Managed-terminal work packages need an active `quality gate` as the main proof expectation.
- `request_review` is a normal agent-facing signal that work is ready for review.
- Managed terminal work must go through Result Finalizer before review/approval routing.
- `submit_patch` is removed from normal agent-facing MCP, but patch/changelist proof is still required for coding work.

Do not assume:

- `request_review` can bypass Result Finalizer.
- `get_checks` and quality-gate selection are separate unrelated systems.
- removing `submit_patch` means patch proof is unnecessary.

Design direction already implied:

- `get_checks` should feed or align with work-package quality-gate selection.
- `request_review` should trigger/consult Result Finalizer for managed terminal work.
- backend or sandbox/work-area systems must capture patch/changelist evidence automatically or internally.

Question to ask later only if implementation needs a visible product choice:

Review behavior:
When an agent requests review from a managed terminal, should Stig see the Result Finalizer summary first every time, or only when proof is missing, failed, or risky?

## Pending Area 9 - Context Tooling Adoption

Status:

Direction decided, implementation details pending.

What is known:

- A downloaded context-tool skeleton exists at `/Users/codex/Downloads/agent-os-context-cleanroom-setup/tools/context/`.
- It is explicitly a reference skeleton, not a production implementation.
- It contains useful concepts: source index, router, packet builder, packet validation, expansion handling, access checks, and package-script extraction.
- It also contains schema that overlaps with existing Agent OS records.

Do not assume:

- the downloaded schema should be run in Agent OS.
- Agent OS should add a second `agent_sessions` model.
- Agent OS should add a second `file_leases` model.
- context expansion should be blocked just because a file was not in the initial package.
- `docs/**` should be excluded from Agent OS context routing.

Design direction already implied:

- Use the skeleton as inspiration for `list_files`, `search_files`, `read_files`, `get_task`, `get_commands`, and `get_checks`.
- Reuse existing Agent OS records first.
- Treat source index/routing data as helper evidence/provenance, not lifecycle truth.
- Record file reads and expansion history automatically.
- Keep context suggestions explainable with reasons and confidence.

Question to ask later only if implementation needs a visible product choice:

Context confidence:
Should Stig see low-confidence context warnings in the main task view, or should they stay in implementation/debug details unless they affect task safety?

## Summary Of What Is Already Decided

Already decided:

- Use short agent-facing tool names.
- Normal agent tool list is 11 tools.
- `get_task` replaces separate normal task-context/report tools.
- File tools are split into `list_files`, `search_files`, `read_files`.
- Agent OS should record file reads.
- Context expansion should be allowed for normal project files and recorded.
- Downloaded clean-room context tooling is inspiration only, not a direct subsystem import.
- `get_commands` and `get_checks` are separate.
- Evidence should be automatic by default.
- `add_observation` is weak/manual fallback.
- `request_review` exists.
- Agents should not directly mark final done.
- Sandbox tools are pending.

Do not reopen these unless Stig explicitly changes direction.
