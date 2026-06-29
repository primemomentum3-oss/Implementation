# MCP Redesign Notes

## Purpose

This folder contains the working notes and structured planning documents for the Agent OS MCP redesign.

The focus is the agent-facing MCP tool surface: the tools agents use while doing real project work through Agent OS.

## Read Order

1. `MCP Redesign Decision Brief.md`
2. `Agent-Facing MCP Tool Specification.md`
3. `Context Tooling Adoption Notes.md`
4. `MCP Redesign Implementation Plan.md`
5. `Open Dependencies And Pending Decisions.md`

## Implementation Priority

Stig decided the MCP redesign should happen before the managed-terminal implementation.

Practical sequence:

```text
1. Redesign the normal agent-facing MCP tool surface.
2. Ensure task/context/check/review behavior can support managed terminals.
3. Build managed terminal readiness, work package, Result Finalizer, and repair loop on top of that cleaner tool model.
```

This does not mean MCP must solve sandboxing, PR delivery, preview deployment, or full UI redesign first. It means the agent-facing tool layer should be cleaned up before managed-terminal work relies on it.

## Main Decision

The confirmed first clean agent-facing MCP tool list is:

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

## Attention Overview Dependency

The MCP redesign happens before managed-terminal implementation, and it must support the later Attention Overview redesign.

Meaning:

```text
The MCP tool surface should make it possible for the Attention Overview tab to start/continue tasks, retrieve task context, show checks, add observations, and request review through Agent OS.
```

The MCP pass does not need to redesign the full UI by itself, but it must not produce a tool model that blocks the Attention Overview from becoming the Agent OS agent-control tab later.

## Important Boundaries

- This is not a UI redesign.
- This is not a full backend/admin cleanup.
- This is not the final sandbox/work-area design.
- This is not the final context-storage design.
- Current-system test outputs are discovery material, not direct fix requests.

## Current Planning Files

- `MCP Redesign Decision Brief.md` explains the product direction and confirmed decisions.
- `Agent-Facing MCP Tool Specification.md` defines each proposed tool.
- `Context Tooling Adoption Notes.md` reviews the downloaded context-tool skeleton and maps useful ideas into the agreed MCP model.
- `MCP Redesign Implementation Plan.md` gives a practical implementation sequence.
- `Open Dependencies And Pending Decisions.md` lists what is still waiting on Stig or another design thread.

## Source Notes Distilled

The raw live notes at `/Users/codex/Desktop/MCP Redesign/mcp-tool-review-live-notes.md` were reviewed and distilled into these structured files. Do not require future implementation agents to read the raw notes unless they need source evidence for how a decision evolved.

Key live-note details now captured here:

- `get_task_context` was discussed, narrowed, and then removed from the normal agent-facing surface.
- `get_task` is the single normal task-continuation tool.
- evidence should be automatic; `add_observation` is only weak/manual fallback.
- current-project task creation/attachment has shown failure behavior that must be checked during implementation.
- the MCP redesign must align with managed-terminal work packages, quality gates, evidence, and Result Finalizer behavior.


## Context Tooling Decision

The downloaded context-tool skeleton at `/Users/codex/Downloads/agent-os-context-cleanroom-setup/tools/context/` should be used as inspiration only.

Key decision:

```text
Adopt useful concepts into the agreed MCP tools and existing Agent OS backend records.
Do not import the clean-room tool package as a separate baseline subsystem.
```

Reasons:

- Agent OS already has context packets, context provenance, runtime runs, evidence, artifacts, tool invocations, file leases, and proof history.
- The downloaded schema overlaps/conflicts with existing Agent OS tables.
- The downloaded router is a skeleton and not reliable enough to own task context by itself.
- The downloaded access controller blocks normal expansion, while Stig decided normal context expansion should be automatic and recorded.
