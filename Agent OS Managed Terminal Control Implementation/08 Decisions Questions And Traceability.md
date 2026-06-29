# 08 Decisions Questions And Traceability
## Source Files Merged
- `11 Decision Log And Open Questions.md`
- `12 Traceability Matrix.md`

---

## Source 1: 11 Decision Log And Open Questions.md

# 11 Decision Log And Open Questions

## Purpose

This file records cleared decisions and remaining questions for implementation agents.

## Cleared Decisions

### D001 - Preserve terminal chat

Stig wants to keep chatting inside terminal panes. Agent OS should manage terminals, not remove them.

### D002 - Support both work entry paths

Agent OS should support:

- Agent OS starts task/work
- Stig starts in terminal and attaches task later

### D003 - Managed requires work package

A worker should be called managed only when Agent OS has task context, role, work package, evidence destination, and finalizer path.

### D004 - Suggested files are plain list

Suggested files should start as a plain file list, no per-file explanations.

### D005 - Suggested files are expandable

The agent can search/read extra files automatically. Agent OS records it.

### D006 - Suggested tests separate from files

Suggested tests should be their own work-package section.

### D007 - Suggested commands separate from quality gate

Do not use `fast checks` terminology. Use suggested commands and quality gate.

### D008 - Quality gate selection order

Use:

1. Agent OS project setting
2. Project docs
3. sensible fallback

### D009 - Agent runs commands, Agent OS records evidence

The terminal agent runs checks. Agent OS records and evaluates evidence.

### D010 - Result Finalizer is required

Managed terminal work must go through Result Finalizer before review/approval.

### D011 - Outcome states

Use:

- Needs repair
- Needs review
- Ready for approval
- Blocked
- Failed

### D012 - Six-section result summary

Use:

1. Outcome state
2. What changed
3. Proof
4. Problems
5. Recommended next action
6. Details/log links

### D013 - Same active agent repairs first

Failed/missing proof goes back to same active agent first.

### D014 - Two repair attempts

Same active agent gets up to two repair attempts before Stig decision.

### D015 - Stig choices after repair limit

Show:

- let same agent continue
- start reviewer/repair agent
- accept with warning
- stop and keep blocked

### D016 - Risk-based accept-with-warning

Keep as Agent OS improvement, not Warren feature. It is human-controlled and evidence-recorded.

### D017 - Branch/PR/preview later

First baseline remains local-first. Warren-style branch/PR/preview delivery is later optional layer.

### D018 - Sandbox is long-term target

Move toward Warren-style sandbox/worker isolation, but first baseline can build on managed terminal/worktree/process foundations.

### D019 - Readiness-hardening first

Implementation should start with current-code mapping and managed-label safety before broad feature build-out.

### D020 - MCP redesign comes first

Stig decided the MCP tools should be redesigned before building the managed-terminal implementation. The clean 11-tool MCP surface is now the first implementation target, and managed-terminal control should build on or align with that redesigned tool surface.

### D021 - Show why managed work is not ready

When a Managed terminal is not ready/trusted yet, the UI/read model should expose the missing readiness reason in plain language.

### D022 - Current startup context is not the work package

Startup context can inform the work package, but it does not satisfy the planned durable managed-worker package by itself.


### D023 - Result Finalizer always visible

Stig confirmed the Result Finalizer summary should always be shown for managed terminal work. It should not appear only for problem cases.

### D024 - Managed terminals get a task when work starts

For any Managed terminal, Agent OS should create or attach a task automatically when meaningful work starts. The agent should not continue meaningful managed work with no task.

### D025 - Terminal mode is selected, readiness is proven

Stig chooses/open modes such as Managed, Limited, or Manual/Clean. Backend should not arbitrarily relabel that choice. Backend should track whether managed readiness/proof is complete enough to trust the work.

### D026 - Attention Overview becomes Agent OS control tab

Stig decided the current top-tab `Attention Overview` should be completely removed/redesigned as its current experience.

The new purpose is: Stig talks to Agent OS in this tab, and Agent OS routes work to managed terminals/agents. The tab should show task, terminal mode/readiness, work package, checks/quality gate, evidence, Result Finalizer summary, repair choices, blocked decisions, and approval state.

This is the non-terminal interaction path for managed agent work.

## Open Questions For Implementation Planning

These questions should be resolved when they become necessary, not before baseline design starts.

### Q001 - UI labels

What exact user-facing labels should distinguish Manual/Clean, Limited, Managed, Agent OS-launched, and managed-readiness states?

Recommendation:

```text
Start with plain labels and tooltips.
Do not invent technical labels.
```

### Q002 - Work package delivery method

How should Agent OS deliver work package to terminal?

Options:

- package file + terminal instruction
- direct terminal prompt injection
- environment variable pointing to package
- MCP/current-project tool
- hybrid

Recommendation:

```text
Use durable package artifact/file plus short terminal instruction for baseline.
```

### Q003 - Command evidence fidelity

Can current terminal panes reliably capture command exit codes?

If not, baseline should record weaker evidence and avoid overclaiming.

### Q004 - Late attachment trust

How much of terminal history before task attachment can be trusted?

Recommendation:

```text
Mark pre-attachment history as limited/backfilled unless Agent OS captured it directly.
```

### Q005 - Risk categories

Which task categories are high risk for accept-with-warning?

Initial high-risk examples:

- secrets
- auth/security
- database/data deletion
- deployment
- billing/payment
- destructive commands

### Q006 - Should Agent OS ever run final verification itself?

Current cleared model says terminal agent runs commands and Agent OS records evidence. Later, Agent OS may also run final verification itself for stronger proof.

Recommendation:

```text
Keep agent-run evidence as baseline. Add Agent OS-run verification later if needed.
```

### Q007 - Learning scope

Should learned file/command suggestions stay project-specific first?

Recommendation:

```text
Yes. Cross-project learning should be later and explicit.
```

### Q008 - Sandbox milestone

What is the first sandbox milestone?

Recommendation:

```text
Managed worktree/process boundary first. Full Warren-style sandbox later.
```

### Q009 - MCP dependency choice

Resolved: redesign MCP tools first. Managed-terminal implementation should come after or build directly on the new 11-tool MCP surface.

Implementation meaning:

```text
Do MCP tool redesign first, then build managed terminal readiness, work package, finalizer, and repair loop on top of the cleaned tool model.
```

### Q010 - Result Finalizer visibility timing

Resolved: show the Result Finalizer summary every time for managed terminal work. It teaches trust and makes the lifecycle visible.


## Implementation Caution

Do not let open questions block the first baseline unless they directly affect safety or data integrity.

The baseline can be implemented with conservative defaults and clear labels.

---

## Source 2: 12 Traceability Matrix.md

# 12 Traceability Matrix

## Purpose

This file maps Stig-cleared planning decisions and Warren/Agent OS review findings to implementation requirements.

Use it to check that implementation work does not drift back into vague terminal tracking without real managed-worker control.

## Source Material Used

Planning source folder:

```text
/Users/codex/Desktop/AGENT OS Improvement notes/
```

Implementation package:

```text
/Users/codex/Desktop/Implementation/Agent OS Managed Terminal Control Implementation/
```

Warren reference:

```text
https://github.com/jayminwest/warren
local review clone used earlier: /private/tmp/warren-review
```

Current Agent OS source of truth remains the project docs and code under:

```text
/Users/codex/Documents/Agent OS/
```

## Product Direction Decisions

| Decision | Meaning for implementation | Covered in package |
| --- | --- | --- |
| S001 | Planning and UI must be broad, visual, and product-understandable for Stig. | `01`, `03`, `09`, `11` |
| S002 | Compare Warren to current Agent OS; do not only reconfirm current Agent OS. | `07`, this file |
| S003 | Translate Warren terms into Agent OS meanings. | `07`, `14` |
| C011 | Warren-style sandbox/worker isolation is a target capability. | `03`, `07`, `10` |
| C015 | Branch/PR/preview delivery is later optional; first baseline is local-first approval. | `00`, `07`, `10` |
| C031 | Preserve terminal chat and add Agent OS control around it. | `00`, `01`, `03`, `09` |

## Managed Worker Decisions

| Decision | Meaning for implementation | Covered in package |
| --- | --- | --- |
| C001 | Managed worker requires prepared work package before managed work starts. | `01`, `03`, `04`, `08`, `10` |
| C012/C016 | Work package is a suggested starting map, not closed truth. | `04`, `10` |
| C013 | Workers are actively supervised while running. | `01`, `03`, `05`, `10` |
| C014 | Use a standard Result Finalizer/Inspector. | `05`, `08`, `10` |
| C017 | Normal context expansion is automatic and recorded. | `04`, `09`, `10` |
| C018 | Context history improves future suggestions. | `04`, `10` |
| C019 | Suggested tests are separate from suggested files. | `01`, `04`, `10` |
| C020 | Start with only suggested files, no per-file explanation. | `04` |

## Evidence And Quality Decisions

| Decision | Meaning for implementation | Covered in package |
| --- | --- | --- |
| C002 | Builder work needs patch/changelist and verification evidence. | `05`, `10` |
| C021 | Agent runs verification; Agent OS records/checks evidence. | `01`, `05`, `08`, `10` |
| C022 | Suggested commands are separate from quality gate. | `01`, `04`, `10` |
| C023 | Command history improves future suggestions. | `04`, `10` |
| C024 | Do not introduce the term `fast checks`; use Warren-style language. | `01`, `04`, `10`, `14` |
| C025 | Quality gate is separate and primary proof expectation. | `04`, `05`, `10` |
| C026 | Quality gate selection order: Agent OS setting, project docs, fallback. | `04`, `08` |

## Repair And Decision Decisions

| Decision | Meaning for implementation | Covered in package |
| --- | --- | --- |
| C003 | Same active agent receives repair feedback first. | `06`, `10` |
| C008/C027/C028 | Same agent gets two repair attempts before Stig decision. | `06`, `10` |
| C004/C006 | Result summary is first view and uses six sections. | `05`, `09`, `10` |
| C005/C007 | Use outcome states and default next actions. | `05`, `09` |
| C009 | Blocked means Stig/owner decision; Failed means worker/run broke. | `05`, `06`, `10` |
| C029 | After two failed repairs, show Stig choices. | `06`, `09`, `10` |
| C030 | Risk-based accept-with-warning is Agent OS improvement, not Warren feature. | `06`, `07`, `10` |

## Starter Role Decisions

| Decision | Meaning for implementation | Covered in package |
| --- | --- | --- |
| C010 | Starter roles are Builder, Reviewer, Verifier, Closer, Coordinator. | `01`, `03`, `04`, `08`, `14` |

## Warren Feature Mapping

| Warren pattern | Agent OS baseline meaning | Status |
| --- | --- | --- |
| Run row | Runtime/task/session record for managed terminal work. | Adapt |
| Burrow sandbox | Future stronger worker isolation; first baseline uses managed terminal/worktree/process boundaries. | Later, design-compatible |
| Seed worker context | Durable work package. | Adopt/adapt |
| Agent runs checks | Terminal agent runs commands; Agent OS records evidence. | Adopt/adapt |
| Reap result | Result Finalizer. | Adopt/adapt |
| Branch/PR/preview | Optional later delivery path. | Later |
| Cleanup/destroy | Baseline retention/cleanup metadata first. | Adapt |
| API-key dispatch | Not required for terminal-first baseline. | Do not require |

## Sub-Agent Review Fixes Applied

| Review issue | Fix added |
| --- | --- |
| No-proof result contradicted source. | `05` and `10` now state no patch plus no verification for managed Builder work becomes Blocked/no-proof, not ordinary Needs repair. |
| Late attach wording too loose. | `01`, `03`, and `08` now separate selected Managed mode from trusted managed-readiness; earlier uncaptured history stays limited/backfilled as evidence. |
| Work package fields too thin. | `01` and `04` now include worker contract, allowed scope, project rules, approved context/memory, and required proof. |
| Active supervision under-specified. | `01`, `03`, `05`, and `10` now include failed start, crash, timeout, silent/stuck, cancellation, unreachable, and capture failure. |
| Cleanup/retention missing from baseline. | `00`, `03`, `07`, and `10` now include local retention/cleanup metadata without requiring full sandboxing. |
| Broken context doc path. | `02` and `08` now point to `docs/context/Agent OS Overview/Agent OS Overview.md`. |
| Managed label unsafe. | `README`, `01`, `03`, `08`, and `09` now separate Managed mode from managed-readiness/trusted result; backend-confirmed package/evidence/finalizer readiness is required before work moves forward. |
| Schema concepts too specific. | `08` now includes a reuse-versus-new-storage table. |
| Command evidence confidence unclear. | `05` now separates structured command runs, pane-log candidates, and manual/agent-reported commands. |
| Finalizer workflow integration unclear. | `05` now maps finalizer outcomes to workflow routing and states finalizer does not own final approval. |
| Repair sequencing underspecified. | `06` now describes repair packet storage/delivery and attempt counting. |
| UI lacks code anchors. | `09` now lists desktop implementation anchors. |
| Rollout order unsafe. | `08` and `10` now start with current-code mapping and managed-label gating before UI claims. |
| Accept-with-warning too loose. | `06` now requires operator decision/evidence record and blocks high-risk domains until policy exists. |
