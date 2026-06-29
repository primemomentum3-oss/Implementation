# MCP Redesign Implementation Plan

## Purpose

This plan turns the confirmed MCP redesign direction into an implementation sequence. It is intentionally structured so future agents can start safely without needing to rediscover every decision from the working session.

This is a plan, not an instruction to implement immediately. Before implementation, inspect the current source, README files, AGENTS.md, docs/context, migrations, tests, and local patterns again.

## Source Findings From Live Notes

The raw live notes were reviewed and distilled into this plan.

Current-system findings to preserve:

- Current-project task creation/attachment can fail even when broader explicit-scope task creation works.
- Observed failure shape: current project context returns `task: null`; current-project create task can return `null`; CLI fallback showed PostgreSQL error `could not determine data type of parameter $2`.
- Likely source areas to inspect before changing task-start flow: `src/services/currentProjectContext.ts`, runtime/pane attach behavior, and pane runtime proof backfill such as `src/services/paneRuntimeProof.ts`.
- Agent OS has existing recording layers: task events, tool invocations, traces, evidence items, artifacts, pane logs, command evidence, patch evidence, review evidence, and generated task reports.
- The weakness is not simply lack of recording. The weakness is that recording depends on task/session binding and on tracked actions. If an agent continues without a task or without tracked tool paths, useful history may not attach to the correct task.

## Implementation Priority Decision

Stig chose MCP redesign as the first implementation target before managed-terminal control.

This plan should therefore be treated as the front door for implementation sequencing. Managed-terminal work should wait until the agent-facing MCP flow is clean enough to support task context, file reads, checks, observations, and review requests reliably.

Do not expand this into sandboxing, PR delivery, preview deployment, or full UI redesign. The priority is the normal agent-facing tool surface.

## Implementation Goal

Create a clean agent-facing MCP tool surface for Agent OS that supports task work, context retrieval, file inspection, checks/commands guidance, automatic evidence capture, manual observations, and review requests.

The target normal tool list is:

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

## Non-Goals For First Pass

Do not implement these as part of the first pass unless Stig gives new direction:

- final sandbox/work-area tools.
- full UI redesign.
- full context storage redesign.
- safety/tool-policy process redesign.
- broad backend/admin cleanup outside MCP.
- keeping old `agent_os_*` names as the final internal design.

## Important Constraints

- Stig is redesigning, not downgrading.
- Do not preserve old names simply because they exist today.
- Do not ask Stig low-level implementation questions unless the answer affects visible behavior, workflow, safety, or system outcome.
- Current Agent OS docs/context has been upgraded and should be treated as current context-routing support, not old MCP truth.
- Future task creation/attachment behavior is pending from Stig.
- Future sandbox/work-area behavior is pending from another design thread.

## Recommended Implementation Phases

### Phase 1 - Source And Behavior Audit

Goal:

Map the current MCP source and current tool behavior before changing names.

Read first:

- project `AGENTS.md`
- root `README.md`
- `docs/context/README.md`
- `docs/context/Context Map.md`
- `docs/context/Agent OS Overview/Agent OS Overview.md`
- `docs/context/Agent OS MCP/Agent OS MCP.md`
- `docs/context/Agent OS MCP/MCP Registration And Execution.md`
- `docs/context/Agent OS Tools/Agent OS Tools.md`
- `docs/context/Agent OS Tools/Tool Governance And Invocations.md`
- relevant source-area README/context files found from the context map

Inspect source:

- MCP registry and catalog.
- current-project tools.
- task tools.
- report tools.
- patch/evidence/review tools.
- tool execution wrapper.
- proof/evidence services.
- task/report services.
- desktop bridge only if UI visibility is affected.

Output of this phase:

- current tool inventory.
- current normal-agent tools vs backend/admin/explicit-scope tools.
- current tool names to replace/merge/remove.
- current tests covering MCP/task/evidence behavior.
- current task creation/attachment failure map, especially current-project task creation and runtime/pane proof backfill.
- evidence layer map showing which actions create task events, tool invocations, traces, evidence items, artifacts, pane logs, command runs, patch artifacts, review records, and reports.

### Phase 2 - Define The New Tool Boundary

Goal:

Separate normal agent-facing tools from backend/admin tools in the code structure and catalog.

Decisions already made:

- normal agent-facing tools are the 11 confirmed tools.
- backend/admin tools should not clutter normal `list_tools` output.
- old names should not be final internal names.

Implementation direction:

- Create or adapt an agent-facing tool catalog layer.
- Keep backend/admin tools in a separate category or registry.
- Ensure `list_tools` returns a clean normal-agent view.
- Decide whether old tools are removed immediately or temporarily bridged during migration based on compatibility needs, but do not document old names as the final design.

Verification:

- unit tests for agent-facing tool catalog.
- test that backend/admin tools are not shown as normal everyday tools.
- test that task-required tools indicate missing-task state clearly.

### Phase 3 - Implement Task Tools

Target tools:

- `list_tasks`
- `create_task`
- `get_task`

`list_tasks`:

- Should return lightweight summaries only.
- Should not dump raw backend task objects.

`create_task`:

- Should create/attach task according to the final task-start design once Stig provides it.
- Until final design exists, do not overfit to current current-project task-create behavior.
- Must return clear success or clear error. No silent `null`.

`get_task`:

- Main rich continuation tool.
- Should include latest task report/summary and attached context.
- Should include previous agent work and relevant file/check history in compressed useful form.
- Should avoid raw evidence dumps unless needed.

Migration/removal:

- normal `get_task_context` should be removed or absorbed into `get_task`.
- normal `get_task_report` should be removed or absorbed into `get_task`.
- current handoff behavior should be evaluated for absorption into `get_task`.

Verification:

- tests for task summary shape.
- tests for `get_task` continuation payload.
- tests that reports/context are included in useful compressed form.
- tests that missing task returns a clear agent-facing message.

### Phase 4 - Implement File Tools

Target tools:

- `list_files`
- `search_files`
- `read_files`

`list_files`:

- lists files/folders only.
- no file contents.
- no relevance guessing.

`search_files`:

- searches names/content.
- returns paths and short match context.
- leads naturally to `read_files`.

`read_files`:

- reads selected files.
- automatically records file-read activity against task/runtime when available.

Important behavior:

- Agent OS should record which files agents read.
- File read records should later support `get_task` continuation summaries.

Verification:

- test `list_files` does not include contents.
- test `search_files` returns bounded results.
- test `read_files` records file reads when a task is attached.
- test missing/unreadable files return clear errors.

### Phase 4A - Context Tooling Integration

Goal:

Adopt useful context-tool skeleton ideas into the agreed MCP file/context tools without creating a parallel context subsystem.

Use from the downloaded skeleton:

- source/file indexing concepts.
- file kind classification.
- generated/runtime folder filtering.
- package-script extraction.
- context expansion reasons such as direct import, test-for-file, config-for-file, schema-for-file, and cross-area.
- packet/read-budget quality checks as backend warnings.

Do not use directly:

- downloaded schema as migrations.
- duplicate `agent_sessions`, `file_leases`, or audit tables.
- strict read blocking outside the initial packet.
- keyword-only router as source of truth.
- `docs/**` exclusion for Agent OS.

Implementation direction:

- Reuse existing Agent OS records first: `context_packets`, `context_pack_provenance`, artifacts, evidence items, tool invocations, runtime runs, task events, command runs, patch artifacts, and proof history.
- Add only narrow missing records if existing records cannot express file reads or context expansion cleanly.
- Treat source index data as generated/helper data, not lifecycle truth.
- Store explanation/confidence with suggestions so future agents know why a file/check was suggested.

Verification:

- test `read_files` records file-read evidence when task/runtime exists.
- test context expansion is allowed for ordinary project files and recorded.
- test generated/runtime/blocked paths are not read by default.
- test low-confidence suggestions are labeled, not presented as certain.
- test `get_task` shows attached context and expansion summary.

### Phase 5 - Implement Commands And Checks

Target tools:

- `get_commands`
- `get_checks`

`get_commands`:

- useful project commands.
- does not own completion requirements.

`get_checks`:

- verification expectations.
- shows all project checks, marking required/recommended/optional for current task.

Verification:

- tests that commands and checks are distinct.
- tests that task-relevant check recommendations appear.
- tests that checks do not run automatically through `get_checks`.
- tests or documentation proving managed-terminal quality gates are selected from the same check/project-rule source, not invented separately.

### Phase 6 - Implement Observation And Review Tools

Target tools:

- `add_observation`
- `request_review`

`add_observation`:

- weak/manual note only.
- not approval-ready proof.
- should appear in task history/continuation context.

`request_review`:

- agent says work is ready for review.
- must not mark task final done.
- should connect to existing review workflow if present.
- for managed terminal work, should trigger or consult Result Finalizer before routing to review/approval.

Remove from normal flow:

- `record_evidence`
- `submit_patch`
- `list_evidence`

Implementation caution:

These backend capabilities may still exist internally. The redesign is about the normal agent-facing MCP surface.

Verification:

- test `add_observation` labels manual/weak quality.
- test `request_review` does not mark final done.
- test normal tool list no longer encourages manual evidence management.

### Phase 7 - Automatic Evidence Integration

Goal:

Ensure the new MCP flow supports automatic evidence recording.

Automatically record:

- tool calls.
- file reads.
- commands/checks if routed through Agent OS or captured by runtime.
- file changes/patches if available from work area design.
- reports/reviews.
- runtime/session linkage.
- pane/session logs where available.
- trace/evidence/artifact records tied to the current task.

Known current-system lesson:

Current tests showed that agents often continue work after task attach/create problems. New behavior should make tracking state explicit and visible.

Implementation direction:

- If task exists, evidence should attach to it automatically.
- If task does not exist, outputs should clearly say tracking is unavailable/partial.
- Avoid silent success with no records.

Verification:

- test task-attached evidence capture.
- test missing-task behavior.
- test proof-history/read-model consumption.

### Phase 8 - Documentation And Migration

Goal:

Make future agents understand the new MCP without reading old behavior as truth.

Update only relevant docs:

- MCP overview/context.
- tool catalog/context.
- current-project/task context docs.
- README sections that mention MCP tool usage.
- migration notes if old tools remain temporarily.

Docs should explain:

- normal agent-facing tool list.
- backend/admin tool boundary.
- automatic evidence behavior.
- `get_task` as continuation source.
- file tool separation.
- sandbox pending state if still pending.

Verification:

- run context navigation check after context docs change.
- run relevant MCP tests.
- run TypeScript check if source changed.

## Migration Strategy

Recommended migration approach:

1. Add the new agent-facing tool layer.
2. Implement new tool names and behavior.
3. Move normal guidance to new names.
4. Keep backend/admin tools available only in explicit/admin contexts if still needed.
5. Add temporary compatibility only if required for existing clients.
6. Remove old names from normal docs/guidance.
7. Verify one full task flow.

Compatibility caution:

Stig said not to keep old names internally as the design goal. If compatibility aliases are used, they should be temporary and clearly marked as migration support, not final architecture.

## Suggested End-To-End Verification Flow

Use one small real task after implementation:

1. Start/create/attach task according to final task-start behavior.
2. `get_task` returns task and continuation context.
3. `list_files` shows project structure.
4. `search_files` finds target files.
5. `read_files` reads target files and records reads.
6. `get_commands` shows useful commands.
7. `get_checks` shows required/recommended checks.
8. Agent performs a tiny change or read-only investigation.
9. Agent OS records evidence automatically.
10. Agent uses `add_observation` only if needed.
11. Agent uses `request_review`.
12. `get_task` shows updated continuation summary for a future agent.

## Risks To Watch

### Risk 1 - Old Tool Surface Leaks Back In

If old backend/admin tools remain visible as normal tools, agents will keep choosing confusing paths.

Mitigation:

Make `list_tools` the clean normal surface and separate admin/explicit-scope tools.

### Risk 2 - `get_task` Becomes A Raw Dump

If `get_task` simply returns all backend records, it will be hard for agents to use.

Mitigation:

Return compressed continuation context first, with raw/debug detail only when specifically needed.

### Risk 3 - Automatic Evidence Is Incomplete Or Invisible

If Agent OS records evidence but agents/Stig cannot see it meaningfully, the system will feel unreliable.

Mitigation:

Ensure `get_task` summarizes evidence/continuation state and later ensure UI proof/history visibility is clear.

### Risk 4 - Task Creation Design Changes Later

Stig said task creation/attachment behavior will be provided later.

Mitigation:

Keep task creation implementation modular and avoid hard-coding current flawed current-project behavior as final.

### Risk 5 - Sandbox Design Conflicts

Sandbox/work-area tools are pending.

Mitigation:

Do not finalize worktree/sandbox MCP names in this pass. Leave an explicit placeholder.

## Product Owner Questions Still Pending

Ask these later only when the relevant design area is ready:

1. Task creation behavior:
When an agent starts meaningful work, should Agent OS automatically create a task, attach to an existing task, or ask Stig when there are multiple likely matches?

2. Sandbox/work area:
When an agent needs to change files, should it always work inside a sandbox first, or only for implementation tasks?

3. Evidence visibility:
Should Stig see automatic evidence as a simple task timeline, a proof/history detail view, or both?

4. Context storage:
Which agent-created notes/context should live outside the project folder, and what should still be stored in the project/repo?

## Definition Of Done For This Redesign

The MCP redesign is done when:

- the clean agent-facing tool list exists.
- normal agents are guided to those tools only.
- backend/admin tools are separated from normal flow.
- `get_task` provides useful continuation context.
- file discovery/read flow is explicit and recorded.
- commands/checks are separated clearly.
- evidence is automatic by default.
- `add_observation` exists only as weak manual fallback.
- `request_review` exists and does not mark final done.
- docs explain the new model.
- tests or smoke checks verify one realistic task flow.


### Risk 8 - MCP And Managed-Terminal Rules Drift Apart

If MCP tools define checks/review one way and managed-terminal work packages define quality gates/finalizer routing another way, agents will get conflicting instructions.

Mitigation:

- Treat `get_checks` and project quality-gate selection as the same source family.
- Treat `request_review` as an agent signal, not final approval.
- Route managed terminal review requests through Result Finalizer.
- Keep automatic patch/changelist capture even though `submit_patch` is removed from the normal agent-facing surface.


### Risk 9 - Clean-Room Context Tools Become A Parallel System

The downloaded context-tool skeleton includes its own schema for source indexes, packet revisions, agent sessions, file reads, expansion requests, file leases, and audit events. Agent OS already has overlapping backend records.

Risk:

Copying this directly would create two systems that both appear to own context/session/file access truth.

Mitigation:

- Treat the skeleton as architecture inspiration only.
- Reuse existing Agent OS records before adding any table.
- Keep backend records as lifecycle truth.
- Store source indexes and routing outputs as helper artifacts/provenance, not final truth.
- Keep normal context expansion automatic and recorded.
