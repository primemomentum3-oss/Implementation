# 04 Work Package And Context System
## Source Files Merged
- `04 Work Package And Context System.md`

---

## Source 1: 04 Work Package And Context System.md

# 04 Work Package And Context System

## Purpose

The work package is the structured task packet Agent OS gives to a managed terminal worker.

It should help the agent start correctly without pretending that Agent OS knows everything from the beginning.

## Core Rule

```text
The work package is a strong starting map, not a closed truth.
```

## Required Work Package Fields

Baseline work package should include:

```text
task_goal
worker_role
worker_contract
allowed_scope
scope_reminders_or_unsafe_action_warnings
project_rules_used
approved_starting_context_or_memory
suggested_files
suggested_tests
suggested_commands
quality_gate
evidence_destination
required_proof
result_expectation
definition_of_ready
repair_context, when applicable
```

## Worker Contract And Scope

The work package should tell the agent how the result will be judged in plain terms.

Example:

```text
Worker contract:
You are the Builder. Make the code change needed for this task. Keep work inside the task scope. Record what you changed and run the quality gate if possible.

Allowed scope:
- settings save behavior
- related tests
- direct documentation if needed

Required proof:
- changed-file/patch evidence
- relevant test evidence
- quality gate result or accepted exception
```

This is not a replacement for project instructions. It is the task-specific contract for the worker run.

## Suggested Files

Suggested files should be a plain list only.

Example:

```text
Suggested files:
- src/settings.ts
- src/databaseUpdateHelper.ts
```

Do not include per-file explanations in the first version.

Reason:

```text
Stig explicitly preferred a plain list. More explanation can bloat the package and create confusion.
```

## Suggested Tests

Suggested tests should be separate from suggested files.

Example:

```text
Suggested tests:
- src/settings.test.ts
```

Reason:

```text
The agent should be able to see likely implementation/context files separately from likely verification files.
```

## Suggested Commands

Suggested commands should be a separate simple section.

Example:

```text
Suggested commands:
- npm test -- settings.test.ts
- npm run typecheck
```

Suggested commands are guidance, not proof.

Actual proof comes from commands that were recorded as run and interpreted by Result Finalizer.

## Quality Gate

Quality gate should be separate from suggested commands.

Example:

```text
Quality gate:
- npm run verify
```

Do not introduce `fast checks` as a product term. Use Warren-style language:

```text
suggested commands
quality gate
```

## Quality Gate Selection Order

Agent OS should select quality gate using Warren-style priority adapted to Agent OS:

1. Explicit Agent OS project quality-gate setting.
2. Project docs such as `AGENTS.md`, `CLAUDE.md`, README, or equivalent.
3. Sensible fallback commands based on project scripts/history.

Implementation note:

```text
Avoid asking Stig to choose command details unless the system cannot infer a safe option.
```

## Normal Context Expansion

The agent may search/read extra project files automatically.

Stig decision:

```text
Always allow normal context expansion automatically.
No block or warning is needed for normal context expansion.
Agent OS should note that the agent searched/read other files.
```

Implementation behavior:

- suggested files are not a hard boundary
- extra searched/read files should be recorded when possible
- context expansion should help future agents understand what happened
- normal context expansion should not interrupt the worker

Scope of this decision:

```text
This applies to normal searching, reading, and inspecting project files.
It does not decide risky actions such as delete, reset, secrets, deployment, or broad edits.
```

## Learning From Context History

Agent OS should use recorded context expansion history to improve future suggested files.

Example:

```text
If settings tasks often start in src/settings.ts and then read src/databaseUpdateHelper.ts, Agent OS may suggest both earlier next time.
```

Do not treat history as permanent truth by itself.

Suggested confidence signals:

- repeated use in similar tasks
- file changed in successful task
- test file connected to successful verification
- reviewer referenced file
- Result Finalizer marked file relevant

## Learning From Command History

Agent OS should use previous command history to improve suggested commands.

Example:

```text
If settings tasks usually prove with npm test -- settings.test.ts, suggest it earlier.
```

Command learning should consider:

- successful runs
- failed runs
- command exit results
- finalizer outcome
- task type
- changed files
- suggested tests

Important:

```text
Actual command evidence remains more important than suggested commands.
```

## Work Package Storage

Implementation should store both:

- worker-facing package
- backend metadata used to generate it

Reason:

```text
Result Finalizer must know exactly what the worker received.
Future agents must understand whether context changed during the run.
```

## Example Work Package

```text
Task goal:
Fix settings save bug.

Worker role:
Builder

Suggested files:
- src/settings.ts
- src/databaseUpdateHelper.ts

Suggested tests:
- src/settings.test.ts

Suggested commands:
- npm test -- settings.test.ts
- npm run typecheck

Quality gate:
- npm run verify

Worker contract:
You are the Builder for this task. Make the required change, stay in scope, record evidence, and report blockers clearly.

Allowed scope:
- settings save behavior
- directly related tests

Required proof:
- patch/changed files
- relevant test evidence
- quality gate result or accepted exception

Evidence destination:
Agent OS task/run evidence records.

Definition of ready:
Patch exists, relevant verification is recorded, quality gate passes or has accepted exception, Result Finalizer returns Ready for approval or Needs review.
```
