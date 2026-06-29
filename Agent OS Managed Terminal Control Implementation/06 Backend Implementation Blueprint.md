# 06 Backend Implementation Blueprint
## Source Files Merged
- `08 Backend Implementation Blueprint.md`

---

## Source 1: 08 Backend Implementation Blueprint.md

# 08 Backend Implementation Blueprint

## Purpose

This file gives implementation agents a backend-oriented blueprint. It is intentionally implementation-facing, but it avoids final schema/API decisions until the existing code is inspected.

## Implementation Strategy

Build the smallest useful baseline around existing Agent OS systems.

Recommended first milestone:

```text
Managed terminal worker baseline
```

Dependency order for managed-terminal implementation after MCP redesign:

1. Map current records/services before adding schema.
2. Separate user-selected Managed mode from backend managed-readiness/trusted result.
3. Create durable work package artifact/metadata.
4. Add finalizer report/read model.
5. Add repair packet storage/delivery and attempt counting.
6. Add UI decision surfaces backed by backend state.
7. Harden learning, cleanup, and sandbox direction after the baseline works.

The milestone should connect:

- pane runtime binding
- task attachment
- work package generation/storage
- command/evidence capture
- quality gate selection
- Result Finalizer
- repair loop state
- UI result summary

## Existing Areas To Inspect First

Before editing, inspect project docs in required order:

1. `docs/context/README.md`
2. `docs/context/Context Map.md`
3. `docs/context/Agent OS Overview/Agent OS Overview.md`
4. runtime/pane docs
5. launcher docs
6. evidence/verification docs
7. linked source files/tests

Likely source areas:

- runtime pane binding
- pane proof/log capture
- project startup context
- launch profiles and launch processes
- agent run contracts
- verification/evidence services
- workflow/review/approval services
- desktop pane UI

## Current Code Anchors

Implementation agents should inspect these current areas before designing migrations or UI state:

- `src/services/paneRuntimeBindings.ts` for pane/task/runtime tracking and current tracking levels.
- `src/services/paneLogs.ts` for terminal log capture and command-like log evidence.
- workflow stage calculation services, especially stage calculator behavior.
- existing records named or represented as `runtime_runs`, `project_startup_contexts`, `context_pack_provenance`, `artifacts`, `evidence_items`, `command_runs`, and `verification_runs`.
- desktop components such as `apps/desktop/src/components/shell/PaneTrustHeader.tsx`, `apps/desktop/src/components/shell/TerminalPane.tsx`, `apps/desktop/src/App.tsx`, and `apps/desktop/electron/main.cjs` for visible pane state and IPC wiring.

Important:

```text
Do not create a new schema just because this document names a concept.
First map the concept to existing Agent OS records, then add only the smallest missing storage needed.
```

## Independent Review Adjustment: Readiness-Hardening First

Before broad managed-terminal implementation, complete the MCP redesign first. Then do one narrow readiness-hardening slice.

Required order:

1. MCP agent-facing tool redesign.
2. Read-only current-code mapping.
3. Managed-label safety.
4. Durable work-package contract.
5. Result Finalizer contract.
6. Same-agent repair-loop state.
7. Acceptance test plan for the managed terminal lifecycle.

Hard rule:

```text
No UI or backend read model should present managed work as ready/trusted until the strict managed-readiness predicate passes. A terminal may still display Managed mode if Stig or Agent OS opened it that way.
```

This prevents the current lower-level runtime tracking concept from being mistaken for the product-level managed worker.

## Managed-Readiness Predicate Contract

Managed mode is selected by Stig or Agent OS. Managed-readiness/trusted result requires all of these to be connected to the same task/run/session:

- task binding
- worker role
- durable work package
- evidence destination
- log/capture path
- quality gate or result expectation
- Result Finalizer path/report destination
- same-run linkage between pane, runtime run, task, package, evidence, and finalizer

If any required part is missing in a Managed terminal, keep Managed mode visible but show the missing readiness reason.

Do not treat a value such as `tracking_level=managed` as sufficient by itself. Runtime tracking is an input to readiness, not proof that managed work is ready/trusted.

## Current-Code Mapping Targets

The independent review highlighted these targets to inspect before coding:

- pane binding/readiness calculation in runtime binding services
- startup context and current-project task creation/attachment
- pane runtime proof backfill and task proof linkage
- workflow stage calculation and existing reliability gates
- command runs, verification runs, evidence items, artifacts, patch artifacts, reviews, approvals, and operator decisions
- desktop trust/header surfaces that display terminal mode and readiness

The goal is to reuse existing records and add a narrow read model/service where needed, not scatter readiness logic across UI, pane binding, workflow, proof history, and MCP.

## Proposed Backend Concepts

Use existing tables/services where possible. If new records are needed, design them around these concepts.

### Managed terminal session

Represents a terminal pane used for task work.

Fields/concepts:

```text
id
task_id
pane_id / runtime_run_id
project_id
repo_id
control_level: manual | limited | managed | launched
role: builder | reviewer | verifier | closer | coordinator
status: created | running | awaiting_result | needs_repair | blocked | failed | completed
work_package_id
started_at
attached_at
closed_at
failure_reason
```

### Work package

Durable package given to worker.

Fields/concepts:

```text
id
task_id
managed_terminal_session_id
role
task_goal
suggested_files[]
suggested_tests[]
suggested_commands[]
quality_gate
result_expectation
evidence_destination
created_at
source_version/hash
```

### Context expansion record

Records extra files searched/read beyond suggested package.

Fields/concepts:

```text
session_id
file_path or area
operation: searched | read | inspected | changed
source: terminal_log | tool_event | explicit_agent_report | backend_observed
confidence
created_at
```

### Command evidence record

Records commands run by terminal worker.

Fields/concepts:

```text
session_id
task_id
command
cwd
started_at
ended_at
exit_code/result
classification: suggested | quality_gate | other
log_artifact_id
```

### Result finalizer report

Stores finalizer output.

Fields/concepts:

```text
session_id
task_id
outcome_state
what_changed
proof
problems
recommended_next_action
details_links
repair_attempt_count
quality_gate_status
created_at
```

## Reuse Versus New Storage

| Concept in this package | First implementation preference |
| --- | --- |
| Managed terminal session | Reuse runtime run / pane binding / task binding where possible. Add metadata only if current records cannot express package/evidence/finalizer readiness. |
| Work package | Prefer durable artifact plus metadata/provenance linked to task/run. Add a narrow record only if artifacts are not queryable enough. |
| Context expansion record | Reuse context provenance, pane logs, tool events, or task timeline if available. Store confidence/source. |
| Command evidence | Reuse structured `command_runs` where available. Keep pane-log command candidates separate and lower confidence. |
| Quality gate result | Reuse verification runs/evidence items where possible. The finalizer reads them; it should not invent proof. |
| Result finalizer report | Store as backend-readable report/artifact linked to task/run, plus any state needed for workflow routing. |
| Repair packet | Reuse context packets, handoffs, runtime run metadata, task timeline, or artifacts before adding new storage. |
| Operator decision | Reuse approval/decision/evidence records where possible. Needed for accept-with-warning. |

## Work Package Builder

Responsibilities:

1. Read task and project context.
2. Select role.
3. Suggest files.
4. Suggest tests separately.
5. Suggest commands separately.
6. Select quality gate.
7. Store package.
8. Make package available to terminal agent.

Quality gate order:

```text
Agent OS project setting -> project docs -> package scripts/history fallback
```

## Terminal Attachment Flow

### Attach existing terminal

```text
User selects terminal.
User attaches task or Agent OS infers task.
Backend creates/updates runtime binding.
Backend records selected mode and readiness state.
If Managed mode is requested and meaningful work starts, backend creates or attaches a task, then creates work package and evidence/finalizer path.
UI may show Managed mode, but displays readiness as incomplete until backend confirms package, evidence destination, and finalizer path.
Earlier uncaptured history stays limited/backfilled as evidence, even if the terminal mode is Managed now.
```

### Start from Agent OS

```text
User creates task.
Agent OS creates managed session.
Agent OS creates work package.
Agent OS opens/starts terminal worker.
Terminal receives package.
Backend starts evidence capture.
```

## Work Package Delivery To Terminal

Possible mechanisms, to choose after inspecting current terminal implementation:

- write package to file and print path in terminal
- inject package text as initial terminal prompt
- set environment variables pointing to package/result paths
- expose via MCP/current-project tool
- combine file path plus short terminal instruction

Recommended baseline:

```text
Use a durable work package file/artifact plus a short terminal instruction.
```

Reason:

- terminal text can be lost or edited
- file/artifact gives Result Finalizer stable source
- shorter prompt avoids noisy terminal startup

## Evidence Capture

Evidence capture should combine:

- pane logs
- command tracking if available
- terminal/tool events if available
- explicit worker result file/report if available
- patch/changelist inspection
- verification records

Implementation should not overclaim precision.

If command exit code cannot be reliably captured from a chat terminal, store a weaker evidence type and mark it as lower confidence until stronger capture is implemented.

## Result Finalizer Service

Inputs:

- task/session
- work package
- logs/artifacts
- commands, separated by evidence confidence class
- changed files/patch
- verification/quality gate
- repair state
- active supervision state such as failed start, crash, timeout, silent/stuck, cancellation, or capture failure

Output:

- outcome state
- six-section summary
- recommended next action
- repair packet if needed

Pseudo-flow:

```text
load session and work package
collect changed files and patch evidence
collect command and quality gate evidence
classify proof status
classify problems
if run/system broke -> Failed
else if repair limit exhausted or decision needed -> Blocked
else if proof missing/failed and same agent available -> Needs repair
else if review required -> Needs review
else if approval required -> Ready for approval
store finalizer report
route workflow action
```

## Repair Loop State

Track:

```text
repair_attempt_count
last_repair_packet_id
last_quality_gate_failure
repair_limit = 2
```

After repair limit:

```text
outcome = Blocked
show Stig decision choices
```

## Role Contracts

Start with roles:

- Builder
- Reviewer
- Verifier
- Closer
- Coordinator

Baseline role rules:

### Builder

Can edit source within task scope. Must produce patch and verification evidence.

### Reviewer

Inspects patch/evidence. Should not edit source by default.

### Verifier

Runs or checks verification evidence. Should not make broad code changes.

### Closer

Prepares closeout/approval package. Cannot override failed gates.

### Coordinator

Orients task and routes work. Should not be sole proof of completion.

## Migration/Rollout Advice

Do not add broad schema churn before checking current records.

Prefer:

1. Reuse existing runtime/pane/task/evidence records where possible.
2. Add narrow tables/artifacts only for missing concepts.
3. Keep first milestone testable with fake/local workers.
4. Do not require live provider/API keys for tests.
