# Context Maintenance Workflow

## Purpose

This workflow explains how agents keep `docs/context/` scalable, accurate, and useful over time.

The goal is not only to write better context once. The goal is to make context stay correct as Agent OS changes.

## Task Start Flow

When an agent starts a task, it should not read the whole context folder.

The starting packet should usually include:

1. `docs/context/README.md`
2. `docs/context/Context Map.md`
3. `Agent OS Overview.md`
4. one relevant area overview
5. one to three focused context files for the actual task

A normal packet should aim for:

- 4-6 context files
- 5-15 source files

If a packet requires 30+ source files for a narrow task, the context structure is probably too broad.

## Context Packet Rule

The first packet should be narrow enough to avoid waste, but complete enough to begin safely.

The packet should answer:

- what area is being changed
- which context files explain that area
- which code files are likely relevant
- which linked context files may need expansion
- which tests or checks matter

The agent should expand only when the task proves more context is needed.

## Context File Creation Flow

When an agent creates or rewrites a context file, it should use this structure:

```md
## Purpose

## Key Structure Overview

## Description Details

## Working Guidance

## Relationships And Dependencies

## Important Code Files

## Important Boundaries

## Links To Other Context
```

`Description Details` may use sub-headings when needed.

`Working Guidance` must explain where a future agent should start, the first 2-4 code files to inspect when useful, what mistakes to avoid, how to verify, and when to expand context.

Use sub-headings for real structure, such as:

- startup flow
- runtime behavior
- UI behavior
- database behavior
- verification behavior
- failure behavior

Do not use sub-headings to mix unrelated work areas into one oversized file.

## Context Expansion Rules

If an agent needs more context while working, it must expand intentionally.

The agent should explain:

- what is missing
- why it is needed
- which file or context area revealed the gap
- what additional context should be read

Example:

> I need to expand context because the tool file calls the current-project resolver, and the current-project behavior controls which files the tool can see.

The agent must not silently wander through unrelated files.

## Context Expansion Learning

When a context expansion happens, the system should later improve one of these:

- context map route
- links between context files
- important code file list
- context packet rule
- missing focused context file
- unclear context explanation

Expansion is not a failure by itself.

Repeated expansion for the same reason means the context structure needs improvement.

## Context Update Triggers

A context file must be updated when:

- current behavior changes
- important code files are added, removed, renamed, or moved
- a new tool, command, UI panel, workflow, database table, or script is added
- an existing flow changes
- tests or checks for the area change
- agents discover that the context was incomplete or misleading
- context links become incorrect
- source ownership changes
- the task changes how agents should work in that area

## When Context Usually Does Not Need Updating

Context usually does not need updating for:

- formatting-only changes
- internal refactors that do not change behavior
- small cleanup that does not affect agent understanding
- temporary experiments
- test-output changes with no behavior change
- generated files
- dependency lockfile changes unless they affect agent workflow

If unsure, the agent should state whether context was checked and why no update was needed.

## Task Closeout Context Check

Before closing a meaningful task, the agent must check:

1. Did I change behavior?
2. Did I change files covered by a context file?
3. Did I add, remove, rename, or move important files?
4. Did I discover missing or wrong context?
5. Did I change tests, commands, scripts, workflows, or UI behavior?
6. Does any context file now mislead future agents?
7. Do links between context files still point to the right places?

If yes to any relevant item, update context before calling the task complete.

## Working Guidance Closeout Check

When creating or updating a context file, the agent must check whether `Working Guidance` is present and still accurate.

Before closing a context-changing task, ask:

1. Does the context file explain where a future agent should start?
2. Does it explain what code files or related context should be inspected first?
3. Does it name the first 2-4 code files to open when useful?
4. Does it describe the common task types that belong in this area?
5. Does it warn about common mistakes or wrong assumptions?
6. Does it explain how changes in this area should be verified?
7. Does it explain when the agent should expand into linked context?

If the answer is no, update `Working Guidance` before closing the task.

This keeps context useful for real work, not only for documentation structure.

## Truth Owner And Source-Backed Closeout Check

When creating or updating a context file, the agent must check whether the written content is backed by source truth.

Before closing a context-changing task, ask:

1. Does the context identify what owns truth for this area?
2. Are behavior claims traceable to code, migrations, tests, package scripts, or current backend behavior?
3. If something could not be verified, is it marked as unclear instead of written as fact?
4. Are UI, CLI, MCP, reports, terminal output, or generated artifacts described as display/carrier surfaces when they do not own truth?
5. Are partial, placeholder, read-only, disabled, hardcoded, UI-only, or not-fully-wired behaviors stated directly?
6. Do links explain the condition that makes each linked context relevant?

If the answer is no, update the context before closing the task.

## Context Source Ownership

Each context file should have clear source ownership.

The project should maintain a source ownership map that connects:

- context files
- code files
- tests
- migrations
- related context files

This allows the system to warn when source code changes but matching context was not reviewed.

Example ownership relationship:

```md
Agent OS Tools.md owns:
- `src/mcp/tools/index.ts`
- `src/mcp/tools/*.ts`
- related tool tests
- tool-related migrations
```

## Stale Context Warning

If source files change but related context files do not, Agent OS should warn before closeout.

Example warning:

> You changed files in Agent OS MCP, but no Agent OS MCP context file changed. Confirm whether context still matches the current system.

The warning should not always block the task.

It should force the agent to check and explain.

## Automated Guardrails

Automated checks should detect:

- broken context links
- missing `Links To Other Context`
- missing `Important Code Files`
- missing `Working Guidance`
- generic `Working Guidance` that does not say where to start, what to inspect, how to verify, or when to expand
- links without clear expansion conditions
- missing truth-owner language
- behavior claims that are not source-backed or marked unclear
- partial, placeholder, read-only, disabled, hardcoded, or UI-only behavior described as active behavior
- missing standard sections
- context files that are too thin
- context files that are too large
- duplicate source ownership without explanation
- references to archived docs as current truth
- planning language in active context
- source files changed without related context being reviewed
- context files with no clear boundaries
- old paths that no longer exist

## Packet Size Guardrail

The system should warn when a context route gives too much context for a narrow task.

Example:

> This task appears to be about one MCP tool, but the packet includes 42 source files. Consider using a narrower context route.

The goal is not to minimize context at all costs.

The goal is to give enough context without forcing agents to read unrelated files.

## Context Maintenance Review

A maintenance review should happen when:

- a major area changes
- a new folder or system is added
- several agents expand context for the same reason
- context files become too large
- context files become too thin
- agents repeatedly read many unnecessary files
- verification finds broken links or stale paths

The review should answer:

1. Are the current folders still the right major areas?
2. Are files split by work intention?
3. Are important code file lists accurate?
4. Are links useful or noisy?
5. Are agents getting too much or too little context?
6. Did recent code changes make any context stale?

## Implementation Order

Build the maintenance system in this order:

1. Define the context file standard.
2. Define the context folder ruleset.
3. Add or update context files to match the standard.
4. Add link checking.
5. Add required-section checking.
6. Add source ownership mapping.
7. Add stale-context warnings.
8. Add packet-size warnings.
9. Add recorded context expansion.
10. Use expansion records to improve routes over time.

## Definition Of Done For A Task That Changes Context

A context-changing task is done when:

- the relevant context files are updated
- `Working Guidance` is present, accurate, practical, and includes verification guidance when useful
- context links resolve and include expansion conditions
- truth ownership is clear
- behavior claims are source-backed or clearly marked unclear
- partial/current limitations are stated directly
- related context is linked only where useful
- important code file lists are still accurate
- the context map still routes tasks correctly
- any new split or merge follows the ruleset
- no historical notes are presented as current truth
- automated context checks pass
- the final summary says whether context was updated or checked and left unchanged

## Final Rule

The context system is maintainable when any future agent can answer these questions without guessing:

- where should this information live?
- should this context file be split?
- should this context file be merged?
- what code files does this context represent?
- what linked context should be read next?
- did this task make context stale?
- what must be checked before closing the task?
