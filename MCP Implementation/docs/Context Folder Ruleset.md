# Context Folder Ruleset

## Purpose

The `docs/context/` folder is the agent-facing source of truth for the current Agent OS system.

It exists so agents can understand Agent OS through focused, current-state context instead of reading the entire repository or old planning documents.

The context folder must be scalable, maintainable, easy to route, and safe for future agents to update.

## Core Principle

Every context file must answer this question:

> If an agent has never seen Agent OS before and receives this file for a task, can it understand the relevant part of the current system well enough to work safely without reading the whole repository?

If the answer is no, the file is too weak.

If the file forces the agent to read too many unrelated source files for a narrow task, the file is too broad.

## What Belongs In `docs/context/`

Context files may contain:

- current system behavior
- current source-code organization
- important code files and what they do
- how one area connects to another area
- rules agents must follow when working in that area
- known constraints in the current implementation
- verification paths for that area
- links to other relevant context files

Context files must describe the system as it exists now.

## What Does Not Belong In `docs/context/`

Context files must not contain:

- future plans
- old implementation plans
- PRDs
- task notes
- open discussions
- historical progress reports
- temporary decisions
- vague ideas
- completed work records
- stale requirements
- archived documentation presented as current truth

Historical or planning material belongs outside active context, for example in `docs/Archived files/`.

## Required Context File Sections

Active context files should use this section structure:

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

`Description Details` may contain sub-headings when the area needs more structure.

`Working Guidance` must explain how an agent should work safely in the area, including where to start, the first 2-4 code files to inspect when useful, how to verify, and when to expand context.

Example:

```md
## Description Details

### Startup Flow

### Runtime Behavior

### Verification Behavior
```

Sub-headings should make context easier to scan. They should not turn one file into a dumping ground for unrelated topics.

## Folder Rules

Folders inside `docs/context/` should represent major Agent OS areas.

Folder names should be plain and understandable.

Preferred naming style:

- `Agent OS MCP`
- `Agent OS Tools`
- `Agent OS UI`
- `Agent OS Database`
- `Agent OS Tasks`
- `Agent OS Runtime`
- `Agent OS Evidence`
- `Agent OS Memory`
- `Agent OS Scripts`
- `Agent OS CLI`

Avoid vague folder names such as:

- `Backend`
- `Core`
- `Misc`
- `System Stuff`
- `Source Organization`

A folder should exist only if it helps agents route work.

## File Naming Rules

Context file names should describe what the file helps the agent understand.

Good examples:

- `Agent OS Tools.md`
- `Tool Governance And Invocations.md`
- `MCP Registration And Execution.md`
- `Runtime Context And Pane Binding.md`
- `Evidence Spine Capture And Artifacts.md`
- `Workflow Readiness And Fix Passes.md`

Avoid names that are:

- too vague
- too technical for no reason
- based only on code folder names
- based on old planning language
- based on one temporary task

## Split Rules

Create or split a context file when:

- one file covers more than one clear work intention
- agents often need only one part of the file
- the file mixes unrelated source-file groups
- the file becomes hard to scan
- the file causes agents to read too many unnecessary source files
- a repeated task type needs its own focused context
- links are doing too much work because the file is overloaded

Split by agent work intention, not by every technical detail.

Good split reason:

> Agents working on MCP tool registration need different context than agents working on current-project tool behavior.

Bad split reason:

> There is another code folder, so it needs its own context file.

## Merge Rules

Do not create a separate context file when:

- the file would be tiny
- agents would always need to read both files together
- the split is only based on technical naming
- the split makes navigation harder
- the information is not independently useful
- the new file duplicates another context file

More files is not automatically better.

The goal is the right-sized context for real work.

## Link Rules

Every context file must include `Links To Other Context`.

Each link must explain the condition that makes the linked file relevant.

A link means:

> If your task touches this relationship, read that context next.

Links should help agents expand context intentionally. They should not be used as a large generic reading list.

A good link says what crossing point triggers expansion, such as stored records, UI meaning, MCP registration, runtime binding, or proof capture.

## Working Guidance Rule

Every active context file must include `Working Guidance`.

A context file is not complete if it only explains what the area contains. It must also explain how an agent should work safely in that area.

`Working Guidance` should cover:

- where to start
- what to inspect first
- the first 2-4 code files to open when useful
- common task types
- common mistakes
- verification expectations
- when to expand into linked context

This prevents context files from becoming passive descriptions or file indexes.

The context folder is only scalable if future agents can use each file to decide what to read, what to avoid, and how to verify work.

## Truth Owner Rule

Each context file must identify what owns truth for the area.

Truth owners may include backend records, backend services, domain rules, migrations, UI local state, CLI output, MCP output, generated artifacts, or local runtime state.

If a surface only displays or transports truth, the context must say that. This is especially important for UI, CLI, MCP, terminal panes, generated reports, and provider output.

## Source-Backed Content Rule

Context must be based on current source code, migrations, tests, package scripts, or current backend behavior.

If context and source disagree, source wins.

If behavior cannot be verified from source, the context must say it is unclear instead of presenting a guess as fact.

## Partial Behavior Rule

If behavior is partial, placeholder, read-only, disabled, hardcoded, UI-only, or not fully wired, the context must say that directly.

Agents must not describe planned or placeholder behavior as current active behavior.

## Important Code File Rules

Each context file must list concrete important code files.

The list should include files agents are likely to inspect or change for that context area.

The list should not include every possible file just because it is nearby.

If a context file has too many important code files, check whether the context should be split by work intention.

## Archived Docs Rule

Archived files may be linked only as historical evidence.

They must not be treated as current implementation guidance unless the active context clearly explains why.

Example:

```md
Historical reference only:
[Old PRD](../../Archived%20files/...)
```

## Banned Active-Context Patterns

Avoid active-context wording like:

- `we plan to`
- `this will`
- `future implementation`
- `proposed design`
- `old approach`
- `temporary note`
- `TODO`
- `maybe`
- `should probably`

These may appear only if clearly marked as a current limitation or non-current archived reference.

## Context Review Standard

A context review should ask:

1. Can a new agent understand this area from the file?
2. Does the file reflect current code?
3. Are the important code files listed?
4. Are related context files linked with conditions for when to read them?
5. Does the file identify the truth owner for the area?
6. Are behavior claims source-backed or clearly marked unclear?
7. Are partial, placeholder, read-only, disabled, hardcoded, or UI-only behaviors stated directly?
8. Is `Working Guidance` practical rather than generic?
9. Is the file too thin?
10. Is the file too broad?
11. Are there stale paths or old concepts?
12. Does it help the agent avoid reading unnecessary files?
13. Does it explain when to expand context?
14. Would this remain useful after the next task?

## Definition Of Done For Context Folder Changes

A context folder update is done only when:

- relevant context files use the standard section structure
- `Working Guidance` explains how future agents should work in the area
- `Working Guidance` includes first-file and verification guidance when useful
- links are correct and include expansion conditions
- truth ownership is clear
- partial/current limitations are stated directly
- behavior claims are source-backed or clearly marked unclear
- important code files are listed
- boundaries are clear
- no archived file is treated as current truth
- no obvious stale behavior remains
- context routes still make sense
- navigation checks pass
- the update helps future agents work with less unnecessary reading

## Final Rule

The context folder is successful when:

> A new agent can start with a narrow task, receive the right context, understand the current system, expand only when needed, complete the work safely, and leave the context better or still correct for the next agent.
