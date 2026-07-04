# Firstmate Capability Coverage Review For Agent OS Supervisor Runtime

## Purpose

This document verifies that the Agent OS Supervisor Runtime plan captures the functional essence of Firstmate.

The assumption for this review is: Firstmate has the best current workflow. Agent OS should implement the same workflow power with Agent OS-native names, records, safety rules, and UI.

This is not a code review of Firstmate. It is a feature-coverage review: does the Agent OS plan include an equivalent for each important Firstmate capability?

---

## Coverage summary

| Firstmate capability | Agent OS equivalent in plan | Coverage |
| --- | --- | --- |
| One main liaison | Coordinator | Covered |
| Many autonomous workers | Worker Runtime Runs | Covered |
| Visible worker sessions | Worker Endpoints: Desktop PTY / backend PTY / process | Covered |
| Separate git worktrees | Agent OS task worktrees | Covered |
| Written task briefs | Work Packages / context packets | Covered |
| Status-file reporting | Worker Signals | Covered |
| Automatic watcher | Supervisor Poller | Covered |
| Durable wake queue | Supervisor Inbox | Covered |
| Drain wakes at supervision turn | `supervisor drain` and Coordinator drain loop | Covered |
| Peek worker | Inspect Worker | Covered |
| Send instruction | Deliver Instruction / control messages | Covered |
| Ship vs scout tasks | Builder vs Scout roles, implementation vs read-only/planning work classes | Covered |
| PR/review/approval flow | Agent OS verification/review/closer/approval workflow | Covered with Agent OS stronger gates |
| Stale worker detection | Supervisor Scan + managedWorkerSupervision | Covered |
| Restart recovery | DB-backed runtime records, wake queue, logs, transitions | Covered |
| AFK/background supervision | `supervisor watch` and future notifier | Partially covered / later phase |
| Secondmates | Domain Coordinators | Deferred intentionally |
| Dispatch profiles | Dispatch profiles | Later phase |
| X/public mention mode | External ingress / notifier / future integration | Out of scope for core |
| `/stow` memory sweep | Agent OS memory promotion and retrospectives | Partially covered by existing system |
| Self-update | Not directly relevant to Agent OS supervisor | Not required |

---

## 1. One liaison / one main controlling agent

### Firstmate behavior

The user talks to one main agent. That main agent routes work, supervises workers, escalates only real decisions, and reports outcomes.

### Agent OS plan

Use `Coordinator` as the main controlling role.

Coordinator responsibilities:

- receive user request.
- create or attach backend task.
- create Worker assignments.
- create Work Packages.
- drain Supervisor Inbox.
- inspect Workers.
- deliver instructions.
- report decisions/approval needs to user.
- never treat Worker output as final approval.

### Coverage result

Covered.

### Implementation notes

Do not expose raw Worker coordination complexity to the user as the default experience. The UI can show it, but the main interaction should still feel like one Coordinator is responsible.

---

## 2. Worker creation and launch

### Firstmate behavior

Firstmate uses `fm-spawn.sh` to create a terminal endpoint, get a worktree, install hooks, record metadata, and launch a worker agent with a brief.

### Agent OS plan

Equivalent pieces:

- Worker Dispatch service.
- Runtime session/run records.
- Work Package context packet.
- Worker Endpoint type.
- Agent OS worktree service.
- Desktop PTY or process launch path.
- Worker startup context.

### Coverage result

Covered.

### Implementation notes

Agent OS should not use a single spawn script. It should use services:

```text
workerDispatch.ts
projectStartupContext.ts
paneRuntimeBindings.ts
worktrees.ts
launchProcesses.ts
```

The implementation must distinguish:

```text
interactive PTY Worker
contract process Worker
```

because only interactive Workers can be live-steered like Firstmate crewmates.

---

## 3. Isolated worktrees

### Firstmate behavior

Each coding worker operates in an isolated git worktree. Firstmate refuses to spawn if the worker is in the primary checkout.

### Agent OS plan

Use existing Agent OS worktrees:

- implementation tasks require worktrees.
- Builder Workers must use task worktrees.
- command evidence for quality gates should run in the worktree.
- Work Package must include worktree path and scope.
- file leases protect edit ownership.

### Coverage result

Covered.

### Implementation notes

Add explicit guard tests:

- Builder cannot start without required worktree.
- Builder cannot use primary repository path as edit cwd.
- quality-gate command for worktree task must use worktree cwd.
- Worker Work Package must state worktree path and branch.

---

## 4. Worker instructions / brief contract

### Firstmate behavior

Each worker receives a Markdown brief telling it:

- what task it owns.
- how to set up.
- how to verify isolation.
- what status states to report.
- when to stop.
- what not to do.
- what completion means.

### Agent OS plan

Use Work Packages as structured context packets.

Work Package must include:

- objective.
- role.
- task id.
- runtime run id.
- worktree/scope.
- allowed/protected paths.
- status protocol.
- required proof.
- completion contract.
- MCP and CLI fallback instructions.

### Coverage result

Covered.

### Implementation notes

Work Package should be visible through:

- `get_task`.
- runtime context packet CLI.
- startup context artifact.

Workers should not need hidden chat state to know what to do.

---

## 5. Worker status protocol

### Firstmate behavior

Workers append status lines to `state/<id>.status`:

```text
working:
blocked:
needs-decision:
done:
failed:
```

The watcher reads these lines and wakes Firstmate for important states.

### Agent OS plan

Use Worker Signals:

```text
working
blocked
needs_decision
ready_for_review
ready_for_approval
completion_claimed
failed
heartbeat
repair_completed
```

Signals should be created by:

- MCP `report_status`.
- CLI `agent-os worker signal` fallback.
- process adapters.
- internal services.

### Coverage result

Covered.

### Implementation notes

Important semantic difference:

`completion_claimed` is not final `done`. It is the Worker equivalent of `done:`. The Coordinator/workflow must inspect and validate before task completion.

---

## 6. Automatic watcher

### Firstmate behavior

`fm-watch.sh` watches status files, turn-end files, check scripts, terminal panes, stale conditions, and heartbeats. It absorbs benign wakes and queues actionable ones.

### Agent OS plan

Use Supervisor Poller:

- scans active runtime runs.
- reads Worker Signals.
- reads runtime transitions.
- reads task events.
- reads tool invocations.
- reads command/evidence/review/verification records.
- reads pane/process logs.
- uses `evaluateManagedWorkerSupervision` for active/idle/stale/stuck/capture health.
- writes actionable wakes to Supervisor Inbox.

### Coverage result

Covered.

### Implementation notes

Start as explicit command:

```bash
agent-os supervisor scan
```

Then add watch mode:

```bash
agent-os supervisor watch --interval 15
```

Use DB advisory locks to avoid two pollers for the same project/installation.

---

## 7. Durable wake queue

### Firstmate behavior

Actionable wakes are appended to `state/.wake-queue`. Draining is lock-protected and restore-on-failure.

### Agent OS plan

Use `supervisor_wakes` table.

Key properties:

- dedupe key.
- statuses: queued, drained, handling, handled, dismissed, superseded.
- transaction drain with `FOR UPDATE SKIP LOCKED`.
- stale handling requeue.
- severity.
- linked task/runtime run.

### Coverage result

Covered.

### Implementation notes

Do not build this as an in-memory queue. Durability is central to the Firstmate workflow.

---

## 8. Drain wakes and resume supervision

### Firstmate behavior

At the start of a supervision turn, Firstmate drains wake queue, reads records, handles them, and re-arms watcher.

### Agent OS plan

Coordinator drain loop:

```text
supervisor drain
for each wake:
  inspect context
  decide action
  execute action or ask user
  mark handled/dismissed/superseded
```

CLI:

```bash
agent-os supervisor drain --limit 20
agent-os supervisor handled <wake-id>
```

### Coverage result

Covered.

### Implementation notes

A wake that was drained but not handled must be recoverable. Add stale handling requeue.

---

## 9. Peek worker / inspect worker

### Firstmate behavior

`fm-peek.sh` captures last N lines from a worker's terminal endpoint.

### Agent OS plan

Use Inspect Worker.

For desktop PTYs:

- Electron owns live buffer.
- add IPC to return last N lines by runtime run / pane ref.
- fallback to provider-log artifacts.

For backend process Workers:

- read process logs and result artifacts.

Inspection returns:

- runtime run/session.
- task.
- endpoint status.
- latest Worker Signal.
- workflow stage.
- terminal tail/log tail.
- evidence summary.
- command runs.
- tool invocations.
- suggested Coordinator action.

### Coverage result

Covered.

### Implementation notes

This is better than Firstmate peek because it combines terminal tail with backend proof.

---

## 10. Send instruction / steer worker

### Firstmate behavior

`fm-send.sh` types a line into the worker terminal.

### Agent OS plan

Use Deliver Instruction:

1. persist `worker_control_messages` record.
2. deliver through endpoint if live.
3. record delivery result.
4. expose pending message through Worker pull path if live delivery fails.

### Coverage result

Covered.

### Implementation notes

Do not send unrecorded text to Worker terminals. Every instruction must be auditable.

This is stronger than Firstmate while preserving its behavior.

---

## 11. Ship vs scout task shapes

### Firstmate behavior

Ship tasks change code. Scout tasks investigate and write reports.

### Agent OS plan

Use work classes and Worker roles:

- Scout role for planning/read-only/investigation.
- Builder role for implementation.
- Reviewer role for review.
- Verifier role for independent verification.
- Closer role for ready package.

### Coverage result

Covered.

### Implementation notes

For Scout outputs:

- final report / evidence.
- `ready_for_decision` when sufficient.

For Builder outputs:

- patch artifact.
- command evidence.
- verification.
- review.
- closer package.
- `ready_for_approval`, not final done.

---

## 12. Delivery modes and approval boundary

### Firstmate behavior

Projects can ship via `no-mistakes`, `direct-PR`, or `local-only`, with optional yolo behavior. Firstmate must not merge without explicit approval unless authorized.

### Agent OS plan

Agent OS should map this to workflow/approval policies, not copy modes literally.

Equivalent concepts:

| Firstmate mode | Agent OS equivalent |
| --- | --- |
| no-mistakes | validation contract + verification + review + closer + approval |
| direct-PR | future lighter delivery profile with review/approval retained |
| local-only | local worktree/branch delivery profile, final user approval before merge |
| +yolo | explicit governance/approval policy, not default |

### Coverage result

Partially covered now; plan covers policy path.

### Implementation notes

Do not implement `+yolo` early. First implement explicit approval and ready states.

---

## 13. Stale and stuck detection

### Firstmate behavior

Watcher detects stale panes and distinguishes still-working vs not-working.

### Agent OS plan

Use `managedWorkerSupervision` plus Supervisor Scan.

States already available conceptually:

- active.
- idle.
- stale.
- stuck.
- capture_failed.
- disconnected.

### Coverage result

Covered.

### Implementation notes

The current read model should become an input into `supervisorScan`, not remain only a UI/detail calculation.

---

## 14. Turn-end signals

### Firstmate behavior

Firstmate installs harness-specific hooks to touch `state/<id>.turn-ended` when a worker finishes a turn.

### Agent OS plan

Do not rely on provider-specific turn-end hooks in V1. Use stronger signals:

- MCP tool calls.
- Worker Signals.
- runtime transitions.
- tool invocation activity.
- provider log flush timestamps.
- process exit.
- inactivity/stale thresholds.

### Coverage result

Covered differently.

### Implementation notes

Provider-specific turn hooks can be added later if needed, but the Agent OS version should prefer backend records and log/evidence activity over fragile UI text hooks.

---

## 15. Check scripts and slow polling

### Firstmate behavior

`state/<id>.check.sh` can poll for PR merge, X mentions, or other slow checks.

### Agent OS plan

Use Supervisor Scan inputs and workflow records:

- verification runs.
- review status.
- closer package status.
- task workflow stage.
- process/launch exit status.
- future external adapters.

### Coverage result

Covered.

### Implementation notes

Add future `supervisor_checks` table only if needed. Do not start with arbitrary shell check scripts; Agent OS already has structured backend records.

---

## 16. Restart-proof behavior

### Firstmate behavior

State lives on disk and in terminal backend. Restart drains wake queue and reconciles live endpoints.

### Agent OS plan

State lives in Postgres and artifacts:

- tasks.
- runtime sessions/runs/transitions.
- Worker Signals.
- Supervisor Inbox.
- context packets.
- startup contexts.
- artifacts/logs.
- command/evidence/review/workflow records.

Desktop live PTYs are not fully restart-proof, but runtime close/reopen/continuation records allow task continuation.

### Coverage result

Covered, with stronger backend truth.

### Implementation notes

Add `supervisor requeue-stale-handling` and endpoint disconnected wakes for recovery.

---

## 17. AFK / background supervision

### Firstmate behavior

`/afk` starts a daemon that handles routine wakes and escalates only important events.

### Agent OS plan

Phase 1:

- `agent-os supervisor watch --interval 15`.
- Supervisor Inbox shows important events.

Later:

- desktop notification.
- OS notification.
- webhook/Telegram/Slack notifier.
- quiet hours.
- notification policy by wake severity.

### Coverage result

Partially covered / later phase.

### Implementation notes

Do not block core supervisor on notification features. First make wakes reliable.

---

## 18. Persistent domain supervisors

### Firstmate behavior

Secondmates are persistent domain supervisors with their own homes, state, backlog, and projects.

### Agent OS plan

Use Domain Coordinators later.

Possible model:

```text
domain_coordinators
  id
  project_scope
  responsibility_scope
  default_dispatch_profile
  active_runtime_session_id
  status
```

They should own a queue of domain-scoped tasks but still use Agent OS backend truth.

### Coverage result

Deferred intentionally.

### Implementation notes

Do not implement Domain Coordinators before Worker supervision, wake queue, and control messages are stable.

---

## 19. Project memory and stow-like behavior

### Firstmate behavior

Firstmate routes durable knowledge to project AGENTS.md, captain data, learnings, backlog, or reports. `/stow` sweeps session knowledge.

### Agent OS plan

Agent OS already has memory candidates, project memory, retrospectives, evals, and memory promotion foundations. The Supervisor plan should integrate later:

- Worker completion can propose memory candidates.
- Supervisor wakes can include `memory_candidate_ready`.
- Coordinator can route accepted learnings to memory promotion.

### Coverage result

Partially covered by existing Agent OS memory system.

### Implementation notes

Do not make Worker Signals write project memory directly. Use memory candidate and approval/promotion flows.

---

## 20. External/public mention mode

### Firstmate behavior

Optional X mode lets public mentions become work requests and follow-ups.

### Agent OS plan

Out of scope for core Supervisor Runtime.

Future equivalent:

```text
external_ingress_requests
external_followups
supervisor_wakes for external requests
notification adapters
```

### Coverage result

Out of scope for current implementation.

### Implementation notes

Do not mix public ingress with local multi-worker supervision until the local trust model is stable.

---

## 21. Self-update

### Firstmate behavior

Firstmate can self-update its own repo and secondmate homes.

### Agent OS plan

Not required for Supervisor Runtime.

Agent OS can handle self-update separately if desired, but it is not core to multi-worker orchestration.

### Coverage result

Not required.

---

## 22. Toolchain/bootstrap checks

### Firstmate behavior

Bootstrap checks required tools, GitHub auth, backend availability, and watcher liveness.

### Agent OS plan

Add Supervisor readiness checks:

```bash
agent-os supervisor doctor
```

Checks:

- DB connectivity.
- migrations current.
- worktree root exists/writable.
- MCP binary built.
- desktop PTY bridge available if inspecting live panes.
- active poller lock status.
- active queued/handling wakes.
- provider CLIs available if launching Workers.
- task/worktree constraints healthy.

### Coverage result

Should be added after Phase 1 or 2.

---

## 23. Guarded teardown / cleanup

### Firstmate behavior

Teardown refuses dirty or unlanded worktrees.

### Agent OS plan

Agent OS should add Worker cleanup policy:

- never remove task worktree while task has unsubmitted patch or unrecorded changes.
- use patch artifact and worktree status checks.
- retain worktree for review/debug until approval.
- mark runtime local retention state.
- only clean after approval or explicit discard decision.

### Coverage result

Partially covered by current worktree/retention concepts; needs explicit cleanup policy in Supervisor feature.

### Implementation notes

Add tests before any automatic cleanup.

---

## 24. Full Firstmate-equivalent workflow acceptance test

The final system should pass this scenario:

```text
1. User asks Coordinator to investigate and fix a bug.
2. Coordinator creates backend task.
3. Coordinator creates Scout Worker.
4. Scout reports reproduction via Worker Signal.
5. Supervisor creates wake.
6. Coordinator drains wake and decides Builder is needed.
7. Coordinator creates Builder Worker with task worktree.
8. Builder records patch, command proof, and completion_claimed.
9. Supervisor creates wake.
10. Coordinator inspects Builder.
11. Coordinator routes verification.
12. Verification failure creates wake and repair instruction.
13. Builder repairs and reports repair_completed.
14. Verification passes.
15. Reviewer Worker runs and approves.
16. Closer package created.
17. Workflow becomes ready_for_approval.
18. Supervisor creates user-facing wake.
19. Coordinator reports ready package to user.
20. User approves through Agent OS approval path.
21. Task becomes done through existing approval boundary.
22. Worktree retention/cleanup state updates safely.
```

If this scenario works, Agent OS has captured the core Firstmate workflow.

---

## 25. Final coverage judgment

The implementation plan captures the Firstmate technical essence if the following are implemented:

- Supervisor Inbox.
- Supervisor Poller.
- Worker Signals.
- Work Packages.
- Worker Endpoints.
- Inspect Worker.
- Deliver Instruction.
- task worktree guard.
- workflow/approval boundary.
- restart/drain recovery.
- stale/disconnected detection.
- multi-worker dispatch.

The only intentionally deferred Firstmate features are:

- Domain Coordinators / secondmates.
- external public mention mode.
- AFK/out-of-band notification.
- dispatch profile sophistication.
- self-update.

Those are valuable, but not required for the first professional implementation of the core multi-agent supervision system.
