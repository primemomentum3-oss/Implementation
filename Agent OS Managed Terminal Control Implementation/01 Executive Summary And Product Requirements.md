# 01 Executive Summary And Product Requirements
## Source Files Merged
- `00 Executive Summary And Scope.md`
- `01 Product Requirements.md`

---

## Source 1: 00 Executive Summary And Scope.md

# 00 Executive Summary And Scope

## Short Summary

Agent OS should become the control layer around terminal-based AI agents.

Stig currently works by talking to agents through CLI terminals. That workflow should remain possible. The improvement is that Agent OS should be able to attach a task, context package, command/evidence tracking, quality gate, repair loop, and approval flow around those terminals.

Plainly:

```text
The terminal remains the place where the agent works.
Agent OS becomes the system that knows what the work is, what proof is needed, and when the work is ready.
```

## Why This Matters

Today, terminal-based agent work can become vague:

```text
The agent says it is done.
The terminal has a lot of text.
Stig has to judge whether the result is actually safe.
```

The target system should make this more controlled:

```text
Agent OS knows the task.
Agent OS knows the worker role.
Agent OS knows what files/tests/commands were suggested.
Agent OS records what files were searched/read.
Agent OS records which commands ran and whether they passed.
Agent OS checks whether enough proof exists.
Agent OS routes repair, review, approval, or blocked decisions.
```

## Main Product Decision

Stig wants both of these workflows:

### Workflow A - Agent OS Starts The Work

```text
Stig creates or approves a task in Agent OS.
Agent OS prepares the work package.
Agent OS opens/starts a managed terminal worker.
The agent receives the task context.
The agent works in the terminal.
Agent OS captures evidence and finalizes the result.
```

### Workflow B - Stig Starts In The Terminal

```text
Stig begins chatting with an agent in a terminal.
The terminal is attached to an Agent OS task.
Agent OS adds task context, suggested files/tests/commands, and quality gate.
The agent continues working in the terminal.
Agent OS captures evidence and finalizes the result.
```

The second workflow is critical because it matches Stig's current working style.

## Attention Overview Product Decision

The current top-tab `Attention Overview` in the Agent OS UI should be completely removed/redesigned as its current experience.

New product purpose:

```text
Attention Overview becomes the Agent OS agent-control tab.
```

This is the place Stig uses when he does not want to chat directly inside terminals. Instead, Stig talks to Agent OS from this tab, and Agent OS routes the work into managed terminals, tasks, work packages, checks, Result Finalizer, repair, review, and approval.

Practical meaning:

- Stig can start work from Agent OS instead of typing directly into a terminal.
- Agent OS creates or attaches the task.
- Agent OS prepares context/work package/check expectations.
- Agent OS sends the work to the correct managed terminal/agent.
- Stig sees status, proof, finalizer summaries, repair choices, and approval choices in the tab.
- The tab should not be a passive alert/attention list anymore. It becomes the command/control surface for Agent OS-managed agent work.

## Baseline Scope

The first implementation should focus on managed terminal control, not full Warren-style sandboxing.

Baseline should include:

- Manual/Clean / Limited / Managed terminal modes plus readiness states
- task attachment for terminal panes
- managed work package generation
- suggested files, suggested tests, suggested commands, quality gate
- automatic normal context expansion tracking
- command evidence capture
- Result Finalizer
- same-agent repair loop
- result summary and decision states
- clear UI indicators
- local retention/cleanup metadata for managed run artifacts/logs

## Out Of Scope For First Baseline

Keep these for later unless the existing code already supports them cleanly:

- full Burrow/bwrap/container sandbox implementation
- automatic GitHub PR creation
- preview deployment
- cross-project global learning
- fully autonomous approval
- changing secrets, deployment, billing, database reset, or paid external systems
- replacing the terminal workflow entirely

## Long-Term Direction

Long term, Agent OS can move toward Warren-style worker isolation:

```text
managed terminal -> managed worktree -> managed process -> sandboxed worker
```

But the first useful product baseline should make terminal-attached managed work reliable.

---

## Source 2: 01 Product Requirements.md

# 01 Product Requirements

## Product Goal

Enable Agent OS to control terminal-based agents without removing Stig's ability to chat with agents in terminals.

The product should make managed terminal work understandable, evidence-backed, repairable, and approval-centered.

## Primary Users

### Stig

Stig is the product owner and a non-coder. He should not have to understand database schemas, launch profiles, adapter internals, or test architecture to use the system.

Stig needs to see:

- what the agent is doing
- whether the terminal is managed or manual
- what task the terminal belongs to
- what proof exists
- what failed or is missing
- what Agent OS recommends next
- when his decision is required

### AI Worker Agent

The terminal agent needs:

- a clear task goal
- a role
- a work package
- suggested files
- suggested tests
- suggested commands
- quality gate
- result/evidence expectations
- repair instructions when needed

### Implementation Agent

The implementation agent needs to preserve current Agent OS principles:

- backend records are source of truth
- terminal output is not lifecycle truth by itself
- approval is separate from worker completion
- evidence must be stored and tied to the task
- UI should display backend truth

## Product Requirements

### PR-001: Preserve Terminal Chat Workflow

Agent OS must continue to allow Stig to chat inside terminal panes.

Requirement:

```text
Terminal chatting remains a supported workflow.
Managed terminal control adds structure around it.
```

Acceptance:

- Stig can start a terminal and chat with the agent.
- That terminal can be attached to an Agent OS task.
- Stig or Agent OS can open a terminal as Manual/Clean, Limited, or Managed.
- Backend records the selected mode and the readiness/proof state separately.
- The UI may show Managed mode when that is what Stig selected, but it must not treat the work as trusted/ready until backend proof requirements are satisfied.

### PR-002: Support Agent OS-Started And Terminal-Started Work

Agent OS must support both start paths.

Requirement:

```text
A managed task can start from Agent OS or from an existing terminal session.
```

Acceptance:

- Agent OS can create a task and prepare a managed terminal worker.
- A Managed terminal should always have a task once meaningful work starts.
- Agent OS should automatically create or attach that task when work starts, unless Stig has already selected an existing task.
- Late attachment is allowed, but earlier unverified history remains limited/backfilled unless Agent OS already captured it.

### PR-003: Show Terminal Control Level

Every terminal pane involved in task work must show its selected mode and backend readiness/proof state.

Initial modes:

- Manual
- Limited attached
- Managed terminal worker
- Agent OS-launched worker

Acceptance:

- Stig can tell whether the terminal was opened as Managed, Limited, or Manual/Clean.
- Stig can also tell whether Managed work is ready/trusted, still being prepared, or missing proof.
- A current code field such as `tracking_level=managed` is not enough by itself to mark managed work as ready/trusted.

### PR-004: Managed Work Requires A Work Package

Managed work is ready/trusted only when Agent OS has prepared or attached a structured work package.

The work package must include:

- task goal
- role
- suggested files
- suggested tests
- suggested commands
- quality gate
- evidence destination/result expectation
- worker contract
- allowed scope
- scope reminders / unsafe action warnings
- project rules used to build the package
- approved starting context/memory
- required proof
- definition of ready/done for that worker

Acceptance:

- Managed mode can be opened before meaningful work starts.
- When meaningful work starts in Managed mode, Agent OS must create or attach a task and prepare the managed-work package path.
- Work package is stored as a durable task/run artifact.
- Result Finalizer knows which package the worker received.

### PR-005: Suggested Files Are A Starting Map, Not A Boundary

Agent OS should suggest files, but not assume its first guess is perfect.

Requirement:

```text
Normal context expansion is automatic and recorded.
```

Acceptance:

- Agent can read/search extra project files without warning/blocking.
- Agent OS records extra files/areas searched/read when possible.
- Future suggestions can learn from the history.

### PR-006: Suggested Tests Are Separate From Suggested Files

Work package must separate suggested implementation/context files from suggested tests.

Example:

```text
Suggested files:
- src/settings.ts

Suggested tests:
- src/settings.test.ts
```

Acceptance:

- Work package has separate fields/sections.
- Result Finalizer does not treat suggested tests as proof by themselves.

### PR-007: Suggested Commands Are Separate From Quality Gate

Work package must include suggested commands and a separate quality gate.

Example:

```text
Suggested commands:
- npm test -- settings.test.ts
- npm run typecheck

Quality gate:
- npm run verify
```

Acceptance:

- Do not introduce `fast checks` terminology.
- Use Warren-style language: suggested commands and quality gate.
- Quality gate is the main proof expectation.

### PR-008: Agent Runs Checks, Agent OS Records Evidence

The terminal agent runs commands. Agent OS records command evidence and checks whether proof is enough.

Acceptance:

- Command history is linked to task/run.
- Command result includes at minimum command text, timing, exit/result if available, and log/artifact reference.
- Result Finalizer can see whether quality gate passed, failed, was missing, or had a justified alternative.

### PR-009: Result Finalizer Runs After Managed Work

After a managed terminal worker claims completion, exits, or reaches a review point, Agent OS must run a Result Finalizer.

The finalizer produces one primary state:

- Needs repair
- Needs review
- Ready for approval
- Blocked
- Failed

Acceptance:

- Terminal text alone cannot mark work ready.
- Finalizer output includes six summary sections.
- Backend workflow uses the finalizer outcome to route next steps.

### PR-010: Same Active Agent Repairs First

If proof is missing or the quality gate fails, Agent OS should send repair feedback to the same active agent first.

Acceptance:

- Same active agent gets up to two repair attempts.
- Repair packet includes exact failure/missing-proof evidence.
- After the limit, Agent OS stops and asks Stig.

### PR-011: Stig Gets Clear Choices After Automation Cannot Finish

When repair attempts are exhausted, Agent OS must present a clear decision point.

Suggested choices:

- Let same agent continue
- Start reviewer/repair agent
- Accept with warning
- Stop and keep as blocked

Acceptance:

- Agent OS does not silently continue infinite repair loops.
- Stig sees the failed quality gate, repair attempts, and current reason.

### PR-012: Risk-Based Accept With Warning

Agent OS may support `accept with warning` as an Agent OS-specific improvement.

Important:

```text
This is not a confirmed Warren feature.
It is an Agent OS improvement inspired by Warren-style failure handling.
```

Acceptance:

- Accept-with-warning is human-controlled.
- Missing proof/warnings are recorded.
- Higher-risk tasks require stronger approval or remain blocked.

### PR-013: Local-First First

First baseline remains local-first and approval-centered.

Acceptance:

- No required GitHub PR delivery in baseline.
- No preview deployment required in baseline.
- Branch/PR/preview delivery can be optional later.

### PR-014: No Provider API Keys Required For Baseline

Agent OS should not require Stig to provide provider API keys for the managed-terminal baseline.

Requirement:

```text
The terminal/CLI may handle its own authentication, but Agent OS should control task context, evidence, repair, and approval around the terminal without needing to own model API keys.
```

Acceptance:

- Baseline tests use fake/local workers or existing terminal processes.
- Agent OS can manage a terminal even when the provider login lives inside the CLI tool.
- Provider API-key storage is not a prerequisite for managed terminal control.

### PR-015: Active Worker Supervision

Agent OS should supervise managed workers while they are running, not only after they stop.

Acceptance:

- Backend records latest useful activity, heartbeat/status, and capture health when available.
- Failed start, crash, timeout, silent/stuck, cancellation, and evidence-capture failure have clear reasons.
- Stig sees a plain explanation when the run fails before a normal final result exists.
