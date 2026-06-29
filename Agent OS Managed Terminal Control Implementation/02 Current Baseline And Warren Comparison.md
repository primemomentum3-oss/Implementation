# 02 Current Baseline And Warren Comparison
## Source Files Merged
- `02 Current Agent OS Baseline.md`
- `07 Warren Comparison And Adopted Patterns.md`

---

## Independent Review: Current Repo Reality Checks

Implementation agents must keep these current-code realities separate from the target design:

- Current pane/runtime tracking can use `managed` language more loosely than the planned managed-readiness/trusted-result state. The product model now separates selected Managed mode from backend readiness.
- Current startup context is useful orientation, but it is not the planned managed-worker work package.
- Current workflow stages and reliability gates are useful inputs, but they are not the planned Result Finalizer contract by themselves.
- Current MCP still exposes older broad `agent_os_*` tools; the clean 11-tool MCP surface is future direction, not current repo truth.
- Desktop role panes may prepare provider sessions, but backend-controlled Agent OS-launched builder/reviewer/coordinator work still requires backend handoff support.

Treat this file as planning context. Confirm current behavior from project docs and source before coding.

## Source 1: 02 Current Agent OS Baseline.md

# 02 Current Agent OS Baseline

## Purpose

This file explains the current Agent OS foundation that implementation agents should build on.

Do not treat this package as a request to replace the existing architecture. The correct approach is to extend the current backend/runtime/evidence foundations.

## Current Foundation From Project Context

Agent OS already has important building blocks:

- backend task records
- task events/reports
- runtime session/run tracking
- desktop terminal panes
- pane runtime binding
- taskless-to-task attachment
- pane log capture
- launcher records
- controlled process profiles
- agent-run contracts
- structured input/result packet pattern
- evidence and verification records
- review and approval boundaries
- automatic executor foundations

## Important Existing Project Rules

Implementation agents must preserve these rules:

```text
Backend records are the source of truth.
UI displays backend truth.
Terminal output is evidence only when captured and linked.
Provider output is not lifecycle truth by itself.
Workers cannot approve their own work.
Important work is final done only after required evidence/review/approval path.
```

## Current Terminal/Panes Baseline

Current Agent OS docs say desktop panes can have runtime context and task binding.

Existing capabilities include:

- panes can have runtime session/run/startup context
- panes can start taskless and attach to a task later
- one active task can be owned by only one active terminal pane at a time
- pane logs can be captured as provider-log artifacts in managed/limited contexts
- closing runtime-bound panes records backend close state

These are the foundation for managed terminal control.

## Current Launcher Baseline

Current Agent OS docs say launchers support:

- launch profiles
- launch records
- scout/builder/reviewer session/handoff preparation
- controlled process launches
- local worker/adapter command execution
- structured agent-run results
- automatic execution foundations

Important current boundary:

```text
Desktop Codex/Claude panes can continue a task through pane binding.
They are not automatically the same as launcher-owned builder/reviewer process launches unless a backend launcher handoff exists and is bound.
```

## Main Gap

The key missing layer is not simply terminals, launchers, or logs by themselves.

The missing layer is:

```text
Managed terminal worker control
```

That means one coherent path where Agent OS can:

1. attach or start a terminal worker
2. bind it to a task and role
3. provide a work package
4. capture context expansion and commands
5. record evidence
6. run Result Finalizer
7. route repair/review/approval/blocked

## Recommended Implementation Approach

Use existing services first.

Implementation agents should inspect these areas before changing code:

- `docs/context/README.md`
- `docs/context/Context Map.md`
- `docs/context/Agent OS Overview/Agent OS Overview.md`
- `docs/context/Agent OS Runtime/Runtime Context And Pane Binding.md`
- `docs/context/Agent OS Launchers/Agent OS Launchers.md`
- `docs/context/Agent OS Launchers/Launch Profiles And Controlled Processes.md`
- `docs/context/Agent OS Evidence/Agent OS Evidence.md`
- `docs/context/Agent OS Review/Agent OS Review.md`

Likely important code areas:

- `src/services/paneRuntimeBindings.ts`
- `src/services/paneRuntimeProof.ts`
- `src/services/paneLogs.ts`
- `src/services/projectStartupContext.ts`
- `src/services/launcher.ts`
- `src/services/launchProcesses.ts`
- `src/services/launch/executeLaunchProcess.ts`
- `src/services/launch/agentRunResults.ts`
- `src/services/launch/resultValidation.ts`
- `src/services/automaticExecutor.ts`
- `src/services/trustedRunner.ts`
- `src/domain/launchProfilePolicy.ts`
- `apps/desktop/electron/main.cjs`
- `apps/desktop/src/App.tsx`
- `apps/desktop/src/components/shell/TerminalPane.tsx`
- `apps/desktop/src/components/shell/PaneTrustHeader.tsx`

## Baseline Reality Check

Current Agent OS is partially ready.

```text
Possible: yes
Already fully implemented: no
Needs product/backend design: yes
```

Do not claim the current system already provides full managed terminal worker behavior. It has the foundation but not the complete product behavior described here.

---

## Source 2: 07 Warren Comparison And Adopted Patterns.md

# 07 Warren Comparison And Adopted Patterns

## Purpose

This file explains which Warren ideas are being adopted, adapted, postponed, or not copied.

## Warren In Plain Terms

Warren is a self-hostable control plane for ephemeral coding agents.

In simplified form:

```text
Warren creates a run.
Warren provisions a Burrow sandbox.
Warren seeds context into the worker.
The agent works inside the sandbox.
The agent runs checks and commits.
Warren reaps the result.
Warren pushes branch / opens PR / handles preview or cleanup.
```

## Important Difference

Warren is more API/sandbox/PR-oriented.

Agent OS is local-first, terminal-based, task/evidence/approval-oriented.

Therefore:

```text
Do not copy Warren directly.
Adapt the useful control patterns to Agent OS terminal workflows.
```

## Adopted Warren-Inspired Patterns

### 1. Worker boundary

Warren has Burrow sandbox isolation.

Agent OS target:

```text
Managed terminal/worktree/process boundary now.
Stronger sandbox boundary later.
```

### 2. Context seeding

Warren seeds worker context.

Agent OS target:

```text
Prepare a work package with task, role, suggested files/tests/commands, quality gate, evidence expectations.
```

Important Agent OS adaptation:

```text
The work package is a starting map, not a closed truth.
```

### 3. Agent runs project checks

Warren expects agents to run quality gates inside the sandbox.

Agent OS target:

```text
Terminal agent runs suggested/chosen commands.
Agent OS records command evidence.
Result Finalizer checks whether proof is enough.
```

### 4. Reap/result finalization

Warren runs `reap` after terminal state.

Agent OS target:

```text
Run Result Finalizer after managed terminal work.
```

Finalizer should inspect patch, proof, quality gate, logs, repair state, and route next action.

### 5. Failure reasons

Warren distinguishes failure shapes such as never_started, crashed, timed_out, dropped_commit.

Agent OS target:

```text
Use plain outcome states plus detailed failure reasons underneath.
```

### 6. Command analytics

Warren mines command behavior after runs.

Agent OS target:

```text
Use command history to improve future suggested commands.
```

## Postponed Warren Patterns

### Branch/PR delivery

Warren pushes branches and can open PRs.

Agent OS baseline:

```text
Local-first and approval-centered.
PR/branch/preview delivery is optional later.
```

### Preview environments

Warren can launch per-run previews.

Agent OS baseline:

```text
Not required for first managed terminal control baseline.
```

### Full Burrow-style sandbox

Warren uses Burrow.

Agent OS baseline:

```text
Design toward stronger sandboxing but start with current worktree/process/runtime foundations.
```

## Agent OS Improvements Beyond Warren

### Risk-based accept-with-warning

Warren has warning/failure events, but no confirmed product workflow for risk-based accept-with-warning.

Agent OS can add:

```text
Human-controlled accept-with-warning, recorded with risk and missing proof.
```

### Local retention before full sandbox cleanup

Warren can clean up sandbox/run resources after reaping.

Agent OS baseline should at least record local retention intent:

```text
retained for review
retained for debugging
eligible for cleanup after approval
```

This keeps the first version practical without requiring Burrow/container isolation.

### Terminal-started managed work

Warren is dispatch-first.

Agent OS needs:

```text
Stig can start in a terminal, then Agent OS attaches and manages the terminal.
```

This is a key Agent OS-specific product need.

## What Not To Import Blindly

Do not require these for first baseline:

- provider API keys stored in Agent OS
- GitHub token as required path
- PR as required delivery
- preview as required proof
- container sandbox as prerequisite for managed terminal control

## Summary Table

| Warren concept | Agent OS implementation meaning |
| --- | --- |
| Run row | Managed terminal/run record tied to task and role. |
| Burrow sandbox | Long-term worker isolation target. |
| Seed worker context | Work package. |
| Stream events | Terminal logs, command evidence, runtime/pane events. |
| Reap result | Result Finalizer. |
| Quality gate | Separate work-package quality gate field. |
| Command mining | Future suggested command learning. |
| PR/preview | Later optional delivery layer. |
