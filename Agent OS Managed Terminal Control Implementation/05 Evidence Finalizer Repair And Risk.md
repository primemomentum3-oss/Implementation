# 05 Evidence Finalizer Repair And Risk
## Source Files Merged
- `05 Evidence Quality Gate And Result Finalizer.md`
- `06 Repair Approval And Risk Decisions.md`

---

## Source 1: 05 Evidence Quality Gate And Result Finalizer.md

# 05 Evidence Quality Gate And Result Finalizer

## Purpose

This file defines how Agent OS should know whether terminal-agent work is actually ready.

Core rule:

```text
The agent can say it is done, but Agent OS decides whether the evidence is enough.
```

## Evidence Model

Managed terminal work should produce evidence connected to the task/run.

Important evidence types:

- work package artifact
- terminal/provider log artifact
- command run records
- command output/log references
- files searched/read beyond suggested files
- patch/changelist/changed files
- verification results
- quality gate result
- repair attempts
- finalizer report
- review findings when applicable
- Stig approval/decision when applicable

## Evidence Confidence Classes

Agent OS should separate evidence strength so the finalizer does not overclaim proof.

| Evidence class | Meaning | Can satisfy quality gate by itself? |
| --- | --- | --- |
| Structured command run | Agent OS has a command record with command, cwd, timing, result/exit code, and log/artifact reference. | Yes, if it matches the gate and passed. |
| Pane-log command candidate | Agent OS sees command-like text in terminal logs, but cannot prove result reliably. | No. It can support investigation only. |
| Agent-reported/manual command | The agent or Stig says a command ran, but Agent OS does not have structured result proof. | No, unless Stig explicitly accepts the risk. |

Baseline rule:

```text
Pane-log candidates and manual reports are not enough to mark a quality gate passed.
```

## Command Evidence

For each captured command, store as much as possible:

```text
command text
working directory
start time
end time
exit code or observed result
stdout/stderr artifact or log reference
associated task/run/pane
whether command was suggested, quality gate, or other
```

The exact capture mechanism may depend on current terminal integration. If exit code is not reliably available from an interactive provider pane, record the strongest available evidence and mark confidence appropriately.

## Quality Gate Evidence

Quality gate is the main completion proof command/expectation.

Result Finalizer should classify quality gate as:

- passed
- failed
- missing
- unavailable
- not relevant with justification
- replaced by accepted alternative

Baseline should focus on passed, failed, and missing.

## Result Finalizer

Result Finalizer is the Agent OS equivalent of Warren's `reap` idea, adapted to local terminal workflows.

It should run after:

- worker claims completion
- worker exits
- pane closes while task-bound
- repair attempt completes
- Stig requests inspection

## Finalizer Inputs

Result Finalizer should inspect:

- task record
- managed terminal/run record
- work package received by worker
- terminal log artifact
- command evidence
- quality gate result
- patch/changelist/changed files
- suggested vs expanded context
- repair attempt count
- review state if any
- risk flags if any

## Finalizer Outputs

Result Finalizer must produce one primary outcome state:

```text
Needs repair
Needs review
Ready for approval
Blocked
Failed
```

It should also produce the standard six-section summary:

1. Outcome state
2. What changed
3. Proof
4. Problems
5. Recommended next action
6. Details/log links

## Outcome State Definitions

### Needs repair

Use when work may be fixable by the same active agent.

Examples:

- quality gate failed
- proof missing
- verification failed
- result file incomplete
- command evidence missing but agent is still active
- patch exists but verification proof is missing

Default next action:

```text
Send exact repair packet to same active agent.
```

### Needs review

Use when implementation evidence exists but review is required.

Examples:

- patch exists
- verification exists
- work touches meaningful code
- reviewer has not checked it yet

Default next action:

```text
Move to reviewer/review workflow.
```

### Ready for approval

Use when work has passed required evidence/review path and awaits Stig/admin approval.

Default next action:

```text
Show Stig approval package.
```

Important:

```text
Ready for approval is not final done.
```

### Blocked

Use when Agent OS needs Stig's decision before continuing.

Examples:

- repair attempts exhausted
- high-risk accept-with-warning decision needed
- scope/risk question needs owner decision
- quality gate unavailable and no safe fallback
- managed Builder claims done but produced no patch and no verification proof

Default next action:

```text
Ask Stig for a decision.
```

### Failed

Use when the worker/run broke or could not complete safely.

Examples:

- failed start
- crashed
- timed out
- stuck/silent beyond limit
- evidence capture unusable
- terminal lost before useful evidence

Default next action:

```text
Preserve evidence and explain the failure plainly.
```

## Six-Section Summary Detail

### 1. Outcome state

Plain state first. Do not make Stig read logs to understand the result.

### 2. What changed

Summarize patch/changelist/changed files.

If no files changed, state that plainly.

### 3. Proof

Summarize command evidence, quality gate, tests, verification, review records.

### 4. Problems

Summarize missing proof, failed checks, stuck/crash, risky scope, repair limit, or blocked decision.

### 5. Recommended next action

Use the default action mapped to the outcome state.

### 6. Details/log links

Link to terminal logs, command logs, patch artifacts, verification records, and full finalizer report.

## Finalizer Rules

### Rule 1: Do not trust terminal text alone

A message like `done` is not enough.

### Rule 2: Coding changes need patch and verification evidence

Managed builder work requires changed-file/patch evidence and verification evidence.

### Rule 3: Failed quality gate routes to repair

If quality gate failed, route to same active agent first unless repair limit is reached.

### Rule 4: Missing quality gate is not ready

If no acceptable verification evidence exists, do not move to approval.

### Rule 5: Result Finalizer produces backend state for routing, not final approval

UI should display finalizer output. The terminal/provider output does not own lifecycle truth.

The finalizer may route work to repair, review, approval, blocked, or failed paths. It must not silently mark important work final-done without the required review/approval path.

### Rule 6: No patch plus no verification is blocked for managed Builder work

If a managed Builder says the task is done but Agent OS sees no patch/changed-file evidence and no verification proof, treat the result as Blocked or a blocking no-proof result.

Use Needs repair when there is actual work to repair, such as a patch with missing or failed verification.

## Current Workflow Integration

Result Finalizer should integrate with existing Agent OS workflow truth instead of bypassing it.

Expected routing meaning:

| Finalizer outcome | Backend workflow meaning |
| --- | --- |
| Needs repair | Create/store repair packet and route it to the same active agent when available. |
| Needs review | Request or prepare reviewer/review workflow. |
| Ready for approval | Prepare approval package; this is not final done. |
| Blocked | Record owner decision needed and show Stig choices. |
| Failed | Record runtime/worker failure and preserve evidence. |

Implementation anchors to inspect before coding:

- workflow stage calculation
- verification gates
- closer packages
- approval/decision records
- runtime run state
- evidence/artifact records

## Example Finalizer Report

```text
Outcome state:
Needs repair

What changed:
- src/settings.ts changed
- src/settings.test.ts changed

Proof:
- npm test -- settings.test.ts failed
- npm run typecheck not run
- quality gate npm run verify not run

Problems:
- Settings save test failed.
- Quality gate missing.

Recommended next action:
Send repair packet to same active agent.

Details/log links:
- Terminal log artifact
- Command log for npm test -- settings.test.ts
- Patch artifact
```

---

## Source 2: 06 Repair Approval And Risk Decisions.md

# 06 Repair Approval And Risk Decisions

## Purpose

This file defines what happens when a managed terminal worker cannot prove the work is ready.

## Main Repair Rule

```text
The same active agent repairs first.
```

If proof is missing or verification fails, Agent OS should send structured repair feedback back to the agent that already has the task context.

## Why Same-Agent Repair First

The same agent:

- knows the original task
- knows what it changed
- has current context loaded
- can often repair faster than a new worker

A separate repair agent should be an escalation path, not the default first response.

## Repair Packet Requirements

A repair packet should include:

- original task goal
- worker role
- repair attempt number
- repair attempt limit
- failed command or missing proof
- relevant log excerpt or artifact link
- quality gate status
- current changed files/patch reference
- required next action
- scope reminder

Example:

```text
Repair needed.

Quality gate failed:
npm run verify

Problem:
settings.test.ts failed.

Required action:
Fix the failed behavior or explain why the gate cannot pass.
Rerun the relevant check and the quality gate if possible.

Attempt:
1 of 2 repair attempts.
```

## Repair Packet Storage And Delivery

The repair packet should be stored as backend-readable task/run data before it is sent to the terminal.

Implementation should first inspect existing Agent OS records such as context packets, agent handoffs, runtime run metadata, evidence/artifacts, and task timeline events. Reuse those where they fit before adding a new table.

Delivery goal:

```text
The same active terminal receives a concise repair instruction, and Agent OS keeps the full repair packet as durable evidence.
```

The repair packet should remain attached to the task even if the pane closes and reopens.

## Repair Attempt Limit

Stig approved this rule:

```text
The same active agent gets up to two repair attempts for the same task and same finalizer problem cycle.
After that, Agent OS stops and asks Stig.
```

## After Repair Limit

When repair attempts are used up, Agent OS should show Stig choices:

1. Let the same agent continue
2. Start a reviewer/repair agent
3. Accept with warning
4. Stop and keep as blocked

The task remains not-ready until Stig chooses a path.

## Accept With Warning

Accept-with-warning is an Agent OS improvement, not a confirmed Warren feature.

It means:

```text
Stig may accept a result even though some proof is missing or imperfect, but the warning and risk must remain recorded.
```

This should never be automatic.

Baseline safety rule:

```text
Accept-with-warning must create an operator decision/evidence record, and the task should not become final-done unless the normal approval rules allow it.
```

## Risk-Aware Acceptance

The system should eventually treat task risk differently.

Lower-risk examples:

- docs update
- small copy/text change
- local UI wording
- non-critical cleanup

Higher-risk examples:

- secrets
- auth/security
- database changes
- data deletion
- deployment
- billing/payment
- destructive commands

Recommended baseline:

```text
Record risk signals and require Stig decision for accept-with-warning.
Do not fully automate risk classification in the first baseline unless existing project services already support it.
Block or disable accept-with-warning for secrets, auth/security, database, data deletion, deployment, billing/payment, and destructive work until a risk policy exists.
```

## Blocked Versus Failed

### Blocked

Blocked means Agent OS needs a decision from Stig.

Examples:

- repair attempts exhausted
- accept-with-warning decision needed
- high-risk area touched
- quality gate unavailable and no safe alternative

### Failed

Failed means the worker/run broke or could not safely complete.

Examples:

- failed start
- crashed
- timed out
- terminal/process lost
- evidence capture unusable

## Approval Boundary

Workers cannot approve their own work.

Agent OS can move work toward:

```text
Needs review
Ready for approval
Blocked
Failed
```

But important final done requires the required evidence/review/approval path.

## Stig-Facing Decision Screen

After repair limit, show:

```text
The task is not ready.
The agent tried 2 repairs.
The quality gate still fails or proof is still missing.

Choices:
- Let same agent continue
- Start reviewer/repair agent
- Accept with warning
- Stop and keep blocked
```

Keep wording plain and product-focused.
