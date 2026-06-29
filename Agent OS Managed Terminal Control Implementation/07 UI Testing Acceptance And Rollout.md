# 07 UI Testing Acceptance And Rollout
## Source Files Merged
- `09 UI Product Requirements.md`
- `10 Testing Acceptance Criteria And Rollout.md`

---

## Source 1: 09 UI Product Requirements.md

# 09 UI Product Requirements

## Purpose

This file describes what Stig should see and control in Agent OS for managed terminal work.

## Attention Overview Redesign

The existing top-tab `Attention Overview` should be treated as a full redesign target.

New role:

```text
Attention Overview is the Agent OS agent-control tab.
```

This tab is where Stig talks to Agent OS when he wants Agent OS to control terminals/agents on his behalf, instead of Stig chatting directly inside terminal panes.

Expected first-version capabilities:

- start a new managed task from Agent OS
- choose or see the target managed terminal/agent
- show the selected terminal mode and readiness
- show task attachment status
- show work package status
- show suggested files, suggested tests, suggested commands, and quality gate
- show live/last known worker status
- show evidence/proof status
- always show Result Finalizer summary for managed work
- show repair attempts and same-agent repair action
- show blocked choices and approval path

Design rule:

```text
Do not preserve the old Attention Overview layout just because it exists.
Redesign it around the Agent OS-controlled workflow.
```

The terminal panes still exist. This tab is the higher-level control surface above them.

## UI Goal

The UI should make terminal trust level and task status obvious.

Stig should not need to read raw terminal output to know whether the work is ready.

## Backend Anchors For UI Implementation

Before changing UI labels, inspect current desktop surfaces such as:

- `apps/desktop/src/components/shell/PaneTrustHeader.tsx`
- `apps/desktop/src/components/shell/TerminalPane.tsx`
- `apps/desktop/src/App.tsx`
- `apps/desktop/electron/main.cjs`

UI must show backend-confirmed state. It should not infer managed completion from terminal text or pane tracking alone.

## Required UI Concepts

### Terminal mode and readiness

Every task-related terminal pane should show both:

- selected terminal mode
- backend readiness/proof state

Initial modes:

- Manual/Clean
- Limited attached
- Managed terminal worker
- Agent OS-launched worker

Labels can be refined, but the meaning must remain clear.

Backend rule:

```text
Show Managed mode when Stig or Agent OS opened that mode.
Show managed work as ready/trusted only when backend confirms task, role, work package, evidence destination, quality gate/result expectation, and Result Finalizer path for the run.
```

### Why Managed Work Is Not Ready Yet

When a Managed terminal is not ready/trusted yet, the UI should show a plain reason.

Examples:

```text
Managed setup: task not attached yet.
Managed setup: no work package yet.
Managed setup: evidence destination missing.
Managed setup: quality gate not selected.
Managed setup: Result Finalizer path missing.
Managed setup: capture health unknown.
```

This keeps Stig from guessing why a terminal is in Managed mode but the work is not ready to trust yet.

### Active task binding

A terminal pane should show:

- task title/id
- project/repo
- worker role
- selected mode and readiness
- current run/session state

### Work package status

For managed terminals, UI should show whether a work package exists.

Suggested display:

```text
Work package ready
Suggested files: 3
Suggested tests: 1
Suggested commands: 2
Quality gate: npm run verify
```

Do not show long per-file explanations in baseline.

### Evidence status

UI should show concise proof status:

- commands recorded
- quality gate passed/failed/missing
- changed files detected
- logs captured
- repair attempts used

### Result summary first

After finalizer runs, UI should show six-section summary before raw logs:

1. Outcome state
2. What changed
3. Proof
4. Problems
5. Recommended next action
6. Details/log links

## Outcome State UI

Use one primary state:

- Needs repair
- Needs review
- Ready for approval
- Blocked
- Failed

Each state should have a clear next action.

| State | UI meaning | Main action |
| --- | --- | --- |
| Needs repair | The agent needs to fix missing/failed proof. | Send repair to same agent. |
| Needs review | Work has evidence but needs review. | Start/request review. |
| Ready for approval | Work passed required path and awaits Stig. | Show approval package. |
| Blocked | Stig must decide. | Show choices. |
| Failed | Worker/run broke. | Preserve evidence and explain. |

## Repair UI

When quality gate fails:

```text
Quality gate failed.
Repair attempt 1 of 2.
Sending failure back to same active agent.
```

After repair limit:

```text
The task is not ready.
The agent tried 2 repairs.
The quality gate still fails or proof is missing.
```

Show choices:

- Let same agent continue
- Start reviewer/repair agent
- Accept with warning
- Stop and keep blocked

## Context Expansion UI

Normal context expansion should not interrupt Stig.

UI can show it in details:

```text
Additional files searched/read:
- src/databaseUpdateHelper.ts
- src/userPreferencesStore.ts
```

Do not show warnings for normal project file search/read.

## Late Attachment UI

When a terminal attaches after work already started, UI should show capture confidence.

Example:

```text
Attached to task after terminal was already running.
Earlier terminal history may be limited/backfilled.
```

This avoids pretending Agent OS captured proof it did not capture.

## Stig-Friendly Language

Avoid exposing implementation jargon as primary UI text.

Prefer:

```text
Managed
Limited
Manual
Quality gate
Proof missing
Needs repair
Ready for approval
```

Avoid:

```text
runtime binding hydration
normalized lifecycle state
adapter contract validation layer
```

## UI Acceptance Criteria

- Stig can tell if a terminal is managed or manual.
- Stig can tell which task the terminal is working on.
- Stig can see the quality gate before or during work.
- Stig can see whether proof exists without reading logs.
- Stig gets a clear decision screen when automation stops.
- Raw logs remain available but are not the first required reading path.
- UI shows backend rejection reasons when a terminal cannot be called managed.
- UI displays command/evidence confidence plainly enough that weak log-only evidence is not confused with passed quality-gate proof.

---

## Source 2: 10 Testing Acceptance Criteria And Rollout.md

# 10 Testing Acceptance Criteria And Rollout

## Purpose

This file defines how to verify the managed terminal control implementation.

## Testing Principle

Tests should prove backend behavior, not only UI text.

Use fake/local workers where possible. Do not require provider API keys or live external services for baseline tests.

## Core Acceptance Scenarios

### Scenario A - Attention Overview is the Agent OS control tab

Given Stig wants to work through Agent OS instead of directly typing into a terminal,
when he opens the top-tab Attention Overview,
then the tab should behave as the Agent OS agent-control surface.

Expected:

- Stig can start or continue managed work from the tab
- Agent OS creates or attaches a task when meaningful managed work starts
- the tab shows terminal mode/readiness, task, package, quality gate, proof, finalizer, repair, blocked, and approval status
- the old passive attention-list behavior is not treated as the target design

### Scenario 0 - Managed readiness safety is a hard gate

Given a terminal pane is in Managed mode or has runtime tracking that appears managed,
but it lacks task, role, durable work package, evidence destination, quality gate/result expectation, or Result Finalizer path,
when Agent OS displays the pane,
then it may show Managed mode but must show managed readiness as incomplete/not trusted.

Expected:

- selected mode remains visible
- UI shows why managed work is not ready yet
- task cannot move forward from terminal text alone
- backend tests cover this before broader UI work


### Scenario 1 - Manual terminal is not managed

Given a terminal with no task and no work package,
when it is shown in Agent OS,
then it must not be labeled as managed.

Expected:

- selected mode is Manual/Clean
- no finalizer path claims managed proof
- terminal text alone cannot mark task ready

### Scenario 2 - Attach terminal to task as limited

Given an existing terminal and an Agent OS task,
when Stig attaches the terminal without full managed package requirements,
then Agent OS marks it Limited attached.

Expected:

- task binding exists
- logs may be captured
- UI shows limited trust level
- finalizer does not overclaim proof from earlier uncaptured history

### Scenario 3 - Create managed terminal package

Given a task and project,
when Agent OS creates a managed terminal worker,
then a durable work package is created.

Expected package fields:

- task goal
- role
- suggested files
- suggested tests
- suggested commands
- quality gate
- worker contract
- allowed scope
- project rules used
- approved starting context/memory
- evidence destination/result expectation
- required proof

### Scenario 4 - Suggested files can expand

Given a package with suggested files,
when the agent reads/searches additional project files,
then Agent OS allows it and records expansion.

Expected:

- no warning/block for normal context expansion
- extra file history is recorded when possible
- finalizer/details can show expanded context

### Scenario 5 - Suggested tests separate from files

Given a work package,
then suggested tests are separate from suggested files.

Expected:

- `suggested_files` field exists
- `suggested_tests` field exists
- tests are not mixed into implementation file list

### Scenario 6 - Quality gate separate from suggested commands

Given a work package,
then quality gate is separate from suggested commands.

Expected:

- `suggested_commands` field exists
- `quality_gate` field exists
- no `fast checks` product label is introduced

### Scenario 7 - Agent runs command, Agent OS records evidence

Given a managed terminal worker,
when the agent runs a command,
then Agent OS records command evidence if capture is supported.

Expected:

- command linked to task/session
- command result or available evidence stored
- log/artifact reference exists when available

### Scenario 8 - Patch exists but proof is missing routes to repair

Given a managed builder changes files but no verification evidence exists,
when Result Finalizer runs,
then outcome is Needs repair.

Expected:

- finalizer does not mark ready
- repair packet is created for same active agent
- repair attempt count increments when repair starts/completes

### Scenario 9 - Failed quality gate routes to same agent

Given quality gate failed,
when Result Finalizer runs,
then same active agent receives repair first.

Expected:

- outcome Needs repair
- repair packet includes failed command/log reference
- same active agent remains owner of repair attempt

### Scenario 10 - Repair limit stops automation

Given two same-agent repair attempts have failed,
when Result Finalizer runs again,
then outcome is Blocked and Stig decision choices are shown.

Expected choices:

- let same agent continue
- start reviewer/repair agent
- accept with warning
- stop and keep blocked

### Scenario 11 - Result summary uses six sections

Given finalizer output,
then UI/API report includes six sections.

Expected:

- outcome state
- what changed
- proof
- problems
- recommended next action
- details/log links

### Scenario 12 - Worker supervision failures become Failed

Given terminal process/run fails to start, crashes, times out, becomes silent/stuck beyond limit, is cancelled, becomes unreachable, or loses evidence capture,
when finalizer or runtime close runs,
then outcome is Failed.

Expected:

- evidence preserved where possible
- plain failure reason shown
- task does not move to approval

### Scenario 13 - Done with no proof becomes Blocked

Given a managed Builder claims the task is done,
and Agent OS sees no patch/changed-file evidence and no verification proof,
when Result Finalizer runs,
then outcome is Blocked or a blocking no-proof result that requires Stig/owner decision.

Expected:

- task does not move to approval
- no repair loop starts unless there is actual work to repair
- finalizer explains that no implementation proof exists

## Suggested Test Types

### Unit tests

- work package builder
- quality gate selection
- finalizer classification
- repair attempt limit
- selected mode and readiness classification

### Integration tests

- attach pane to task
- managed package creation
- command evidence to finalizer
- repair loop state transitions
- result summary generation

### Desktop checks

- terminal mode and readiness visible
- task binding visible
- result summary visible
- decision screen visible

### Smoke tests

- fake worker claims done without proof and produces no patch/verification -> Blocked/no-proof result
- fake worker changes file and records passing quality gate -> Needs review/Ready for approval depending workflow
- fake worker fails gate twice -> Blocked with Stig choices

## Rollout Plan

### Phase 1 - Current-code mapping and managed label gating

- map current pane/runtime/task/evidence/workflow records
- define selected terminal modes and readiness states against existing records
- prevent UI from showing Managed unless backend confirms package, evidence destination, and finalizer path

### Phase 2 - Durable work package

- create work package from task/project
- store worker-facing package and backend metadata/provenance
- support Agent OS-started and terminal-attached path at basic level

### Phase 3 - Evidence and finalizer read model

- record structured command evidence where available
- store pane-log command candidates separately as weaker evidence
- implement Result Finalizer summary and outcome routing

### Phase 4 - Repair loop and decision screen

- same-agent repair packet
- repair attempt count across pane close/reopen for the same problem cycle
- stop-after-two decision screen
- operator decision record for accept-with-warning

### Phase 5 - Learning, retention, and hardening

- improve suggested files from history
- improve suggested commands from history
- track retained-for-review / retained-for-debugging / eligible-for-cleanup states
- stronger sandbox/workspace isolation
- optional PR/preview delivery later

## Verification Safety

Prefer targeted, non-destructive checks before any aggregate script.

Recommended first checks:

```text
npm run build
npm run check
focused node --test tests
npm run desktop:check when UI changes
specific smoke scripts that do not reset DB
```

Do not run aggregate scripts that call `db:reset` unless Stig explicitly approves disposable-data testing.

## Non-Destructive Verification

Do not require destructive database resets for normal verification.

If disposable-data tests are needed, require explicit approval and follow Agent OS project rules.
