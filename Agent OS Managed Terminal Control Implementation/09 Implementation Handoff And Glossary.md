# 09 Implementation Handoff And Glossary
## Source Files Merged
- `13 Implementation Handoff Checklist.md`
- `14 Glossary.md`

---

## Source 1: 13 Implementation Handoff Checklist.md

# 13 Implementation Handoff Checklist

## Purpose

This file tells a future implementation agent how to start without needing the planning conversation.

The goal is to implement a baseline where Agent OS can manage terminal-based agents through task context, evidence, quality gates, repair, and approval without removing Stig's terminal-chat workflow.

## Read First

Inside Agent OS project, read current source-of-truth docs first:

1. `/Users/codex/Documents/Agent OS/docs/context/README.md`
2. `/Users/codex/Documents/Agent OS/docs/context/Context Map.md`
3. `/Users/codex/Documents/Agent OS/docs/context/Agent OS Overview/Agent OS Overview.md`
4. runtime/pane binding docs
5. launcher docs
6. evidence/verification docs
7. linked source files/tests

Then read this implementation package in README order.

## Attention Overview Handoff

The top-tab `Attention Overview` is a redesign target. Future implementation agents should not preserve the old tab concept as the product goal.

Target role:

```text
Attention Overview = Agent OS agent-control tab
```

Use it for the workflow where Stig talks to Agent OS, and Agent OS controls the terminals/agents underneath. It should become the main UI surface for managed work when Stig does not want to chat directly in terminal panes.

## Product Baseline In One Paragraph

Stig wants to keep talking to agents in terminals, but Agent OS should wrap those terminals with task ownership, a work package, evidence capture, a quality gate, a Result Finalizer, same-agent repair, and clear approval/block/failure decisions. The system should also support Agent OS-started workers. Both paths should use the same backend truth.

## Independent Review Start Rule

Start with the MCP redesign first. After MCP redesign, start only the readiness-hardening slice for managed terminals.

Do not begin broad managed-terminal implementation until these are clear:

- current records/services are mapped to planned concepts
- managed-readiness predicate is defined
- Managed mode is separated from managed-ready/trusted result
- managed work cannot move forward until package/evidence/finalizer readiness exists
- durable work-package format/storage is defined
- Result Finalizer report and routing contract are defined
- same-agent repair packet and two-attempt state are defined
- acceptance tests cover the managed terminal lifecycle

This is not a pause on all work. It is the first implementation slice.

## First Implementation Slice

Overall first slice:

```text
Redesign MCP agent-facing tools first.
```

Then implement the smallest managed-terminal baseline that proves this flow:

```text
Task exists -> Agent OS creates work package -> terminal/run is marked managed only after package/evidence/finalizer path exists -> worker works -> evidence is captured -> Result Finalizer routes Needs repair / Needs review / Ready for approval / Blocked / Failed -> UI shows the backend result.
```

## Do Not Start With

- full Burrow/container sandboxing
- GitHub PR delivery
- preview deployment
- provider API-key storage
- broad autonomous approval
- global learning system
- deleting or resetting data

These are later layers or require separate approval.

## Current-Code Mapping Checklist

Before changing schema, map each concept to existing code and records.

| Concept needed | Check existing areas first |
| --- | --- |
| Pane/task binding | `src/services/paneRuntimeBindings.ts` and runtime context docs |
| Pane logs | `src/services/paneLogs.ts` and desktop terminal IPC |
| Work package/context | project startup context, context packets, context provenance, artifacts |
| Command proof | command runs, verification runs, evidence items, artifacts |
| Workflow state | workflow stage calculator, verification gates, closer packages, approval records |
| UI trust display | `PaneTrustHeader.tsx`, `TerminalPane.tsx`, `App.tsx`, `main.cjs` |
| Managed-readiness read model | Inspect runtime binding, workflow, evidence, finalizer/report, and UI trust sources before deciding the owning service. |
| Current-project task attach risk | Inspect current-project task creation, startup/runtime attach, and pane proof backfill before relying on the normal `create_task` path. |

## Required Backend Behaviors

1. A tracked pane is not automatically a managed worker.
2. Managed mode is selected by Stig or Agent OS; managed-ready/trusted work requires task, role, durable work package, evidence destination, quality gate/result expectation, and Result Finalizer path.
3. Late-attached terminals become managed only from the managed point forward.
4. Work package must be durable and must include worker contract, allowed scope, suggested files, suggested tests, suggested commands, quality gate, required proof, and evidence destination.
5. Suggested files are a plain list and can expand automatically.
6. Context expansion should be recorded when possible, not blocked.
7. Suggested commands and quality gate are separate.
8. Structured command runs are stronger proof than pane-log command candidates.
9. Pane-log command candidates cannot by themselves satisfy a quality gate.
10. Result Finalizer routes state; it does not bypass review/approval.
11. Same active agent receives repair first.
12. Two same-agent repair attempts are allowed for the same task/problem cycle before Stig decision.
13. Accept-with-warning requires Stig/operator decision and evidence record.
14. High-risk accept-with-warning is blocked until risk policy exists.
15. Worker supervision tracks failed start, crash, timeout, silent/stuck, cancellation, unreachable, and evidence-capture failure.

## Suggested Milestones

### Milestone 1 - Managed Label Safety

Make sure UI/backend separates selected Managed mode from managed-ready/trusted work.

Acceptance:

- Manual/Clean terminal stays Manual/Clean.
- Limited terminal stays Limited unless Stig or Agent OS changes mode.
- Managed terminal remains Managed mode, but readiness shows incomplete until package/evidence/finalizer path exists.
- Managed work cannot move forward from terminal text alone.

### Milestone 2 - Work Package Artifact

Create durable work package storage using existing records/artifacts where possible.

Acceptance:

- Package can be loaded later by finalizer.
- Package includes required fields.
- Package is tied to task/run/pane.

### Milestone 3 - Evidence Classification

Separate structured proof from weak log-only evidence.

Acceptance:

- Structured command evidence can satisfy quality gate if passed.
- Pane-log command candidate is visible but cannot satisfy quality gate alone.
- Missing proof remains visible to finalizer and UI.

### Milestone 4 - Result Finalizer

Implement finalizer report and routing.

Acceptance:

- Produces six-section summary.
- Routes Needs repair / Needs review / Ready for approval / Blocked / Failed.
- No patch plus no verification for managed Builder work becomes Blocked/no-proof.

### Milestone 5 - Repair Loop

Send repair packet to same active agent first.

Acceptance:

- Repair packet is stored.
- Same active terminal receives concise repair instruction.
- Attempt count survives pane close/reopen for the same task/problem cycle.
- After two attempts, Stig choices are shown.

### Milestone 6 - UI Decision Surface

Show terminal mode, managed-readiness status, package status, proof status, finalizer summary, and blocked/repair choices.

Acceptance:

- Stig does not need raw logs first.
- Backend rejection reasons are visible when terminal cannot be called managed.
- Accept-with-warning is explicit and recorded.

### Milestone 7 - Retention And Hardening

Add local retention/cleanup metadata and prepare later sandbox work.

Acceptance:

- Each managed run can be marked retained for review, retained for debugging, or eligible for cleanup after approval.
- No full sandbox is required for this milestone.

## Questions For Stig Only If Blocked

Ask Stig product-level questions only when the implementation cannot safely proceed without an owner choice.

Good question shape:

```text
This affects what you will see or what the system is allowed to do.
Here are the visible options and tradeoffs.
Which behavior do you want?
```

Do not ask Stig to choose tables, code structure, libraries, IPC design, or test implementation details.

---

## Source 2: 14 Glossary.md

# 14 Glossary

## Purpose

This glossary keeps Warren terms, Agent OS terms, and Stig-facing product language aligned.

## Terms

### Agent OS

The local control system around tasks, terminals, evidence, review, and approval.

### Agent OS-started work

A task starts in Agent OS first. Agent OS prepares the work package and opens/starts the worker terminal.

### Terminal-started work

Stig starts chatting in a terminal first. Later the terminal is attached to an Agent OS task and receives Agent OS control from that point forward.

### Manual/Clean terminal

A normal terminal chat chosen by Stig with no managed task package. Useful, but not trusted completion proof.

### Limited attached terminal

A terminal Stig or Agent OS treats as attached/observed but not full managed work. Agent OS may capture logs/history, but completion proof is limited.

### Managed terminal worker

A terminal Stig or Agent OS intentionally uses for managed task work. When meaningful work starts, Agent OS should create or attach a task. Managed work becomes ready/trusted only after role, durable work package, evidence destination, quality gate/result expectation, log/capture path, and Result Finalizer path are connected.

### Agent OS-launched worker

A worker process/terminal started by Agent OS from backend task/handoff state. This is the strongest first-baseline control path.

### Work package

The task packet Agent OS gives the worker. It contains task goal, role, worker contract, allowed scope, suggested files, suggested tests, suggested commands, quality gate, required proof, and evidence destination.

### Suggested files

A plain starting list of files likely relevant to the task. It is not a hard boundary.

### Suggested tests

A separate list of likely useful test files or test targets.

### Suggested commands

Commands the worker may run while working. They are helpful guidance, not proof by themselves.

### Quality gate

The main proof command or verification expectation for the task. Use this Warren-style term; do not rename it to `fast checks`.

### Evidence

Backend-readable proof tied to the task/run: command records, verification results, logs, patch/changelist, artifacts, review findings, repair attempts, and decisions.

### Structured command run

A strong command evidence record where Agent OS knows command, location, timing, result/exit code when available, and log/artifact reference.

### Pane-log command candidate

A weaker signal where a terminal log appears to show a command, but Agent OS cannot reliably prove the result.

### Result Finalizer

Agent OS service that inspects work package, patch, evidence, quality gate, logs, repair state, and risk, then routes the result.

Warren-related translation: this is Agent OS's local-terminal adaptation of Warren's result reaping idea.

### Needs repair

The same active agent should fix missing or failed proof.

### Needs review

Work has evidence but should be reviewed before approval.

### Ready for approval

The work passed the required path enough to show Stig/admin an approval package. This is not final done by itself.

### Blocked

Agent OS needs Stig/owner decision before continuing.

### Failed

The worker/run broke or could not complete safely, such as failed start, crash, timeout, silent/stuck, cancellation, unreachable terminal, or evidence-capture failure.

### Repair packet

A structured message and backend record telling the same active agent what failed or is missing, with links to proof/logs and the required next action.

### Accept with warning

A human decision to accept a result even though some proof is imperfect or missing. This is an Agent OS improvement, not a confirmed Warren feature. It must be recorded and blocked for high-risk work until a risk policy exists.

### Local retention/cleanup metadata

State that says whether run artifacts/logs/work areas are retained for review, retained for debugging, or eligible for cleanup after approval.

### Burrow sandbox

Warren's stronger worker isolation concept. Agent OS should design toward stronger sandboxing later, but the first baseline does not require full Burrow/container isolation.
