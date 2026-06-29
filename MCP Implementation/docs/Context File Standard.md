# Context File Standard

## Purpose

This file defines the standard structure for Agent OS context files.

A context file is a current-state explanation of one part of Agent OS. It should help an agent understand the system area, choose the right code files, avoid wrong assumptions, and know when related context must be read.

This standard is intentionally simple. It should be strong enough to keep context professional, but not so complex that agents create bloated documentation.

## Standard Context File Sections

Every active context file should use these sections:

```md
# Agent OS [Area Name]

## Purpose

## Key Structure Overview

## Description Details

## Working Guidance

## Relationships And Dependencies

## Important Code Files

## Important Boundaries

## Links To Other Context
```

## Purpose

The `Purpose` section explains what the context file is for.

It should answer:

- what part of Agent OS the file explains
- why an agent would read this file
- what type of task should start here

Good example:

```md
## Purpose

This file explains Agent OS tool behavior: where tools are defined, how agents access them, how tool calls connect to MCP, and which related context files should be read when tool behavior touches UI, database records, or current-project routing.
```

Weak example:

```md
## Purpose

This file is about tools.
```

The weak example is too thin because it does not explain why the file matters or when an agent should use it.

## Key Structure Overview

The `Key Structure Overview` section gives the agent a fast map of the area.

It should explain the main parts of the system area before going into detail.

It may include:

- main components
- major folders
- important runtime pieces
- ownership boundaries
- how the area is divided

Good example:

```md
## Key Structure Overview

Agent OS tool behavior has four main parts:

- MCP registration exposes tools to agents.
- Tool implementation files contain the backend behavior for each tool group.
- Database records store durable lifecycle and evidence state when tools write project data.
- UI panels display tool availability and results, but do not own backend truth.
```

This section should be short enough to scan, but specific enough that an agent understands the area before reading details.

## Description Details

The `Description Details` section explains how this part of Agent OS currently works.

This section is allowed to contain sub-headings when one flat section would become hard to read.

Allowed example:

```md
## Description Details

### Tool Registration

Tool registration happens when the MCP server loads the available tool definitions and exposes them to agent clients.

### Tool Execution

Tool execution validates the request, runs backend logic, and returns structured output to the calling agent.

### Tool Results

Tool results may be informational, or they may write durable records depending on the tool type.
```

Use sub-headings when they make the file easier to scan.

Do not use sub-headings to create a long article or hide unrelated topics inside one file.

This section should describe current behavior only. It should not contain future plans, task notes, historical explanations, or old implementation ideas unless clearly marked as not current.

## Working Guidance

The `Working Guidance` section explains how an agent should safely work in this context area.

It should answer:

- where the agent should start
- what the agent should inspect first
- the first 2-4 code files the agent should usually open for normal work in this area
- what task types usually belong here
- what mistakes agents commonly make in this area
- how changes in this area should be verified
- when the agent should expand into linked context

This section should be practical, not theoretical.

Good example:

```md
## Working Guidance

Start by reading the area overview and the most specific context file for the task. Then inspect the listed important code files before changing behavior.

For narrow changes, avoid reading every linked context file immediately. Expand only if the task touches another area, such as database records, UI behavior, or MCP tool execution.

Verify changes with the tests or checks listed in this context area. If verification changes, update this context file.
```

A weak `Working Guidance` section only tells agents to be careful. A strong one tells agents where to start, what to inspect, what to avoid, how to verify, and when to expand context.

`Working Guidance` must not be generic. It must include practical instructions that a fresh agent can act on without already knowing the codebase.

It should normally name the first 2-4 code files to inspect for ordinary work in the area. It should also name the checks, tests, or verification path that usually proves the area still works.

## Relationships And Dependencies

The `Relationships And Dependencies` section explains what this area connects to.

It should answer:

- what other Agent OS areas depend on this area
- what this area depends on
- where data or control flows across boundaries
- when a task in this area becomes a cross-area task

Good example:

```md
## Relationships And Dependencies

Tool behavior depends on MCP for agent access, database records for durable state, current-project context for project-scoped operations, and UI panels for visible tool feedback. A tool task becomes a database task if it changes stored records, and it becomes a UI task if the visible tool behavior changes.
```

This section explains relationships in plain language. The actual clickable expansion links belong in `Links To Other Context`.

## Important Code Files

The `Important Code Files` section lists the real source files that usually matter for this context area.

This does not need to list every file.

It should list the files an agent is most likely to inspect or change for tasks in this area.

Good example:

```md
## Important Code Files

- `src/mcp/server.ts` - creates and runs the MCP server.
- `src/mcp/tools/index.ts` - registers available MCP tools.
- `src/mcp/tools/currentProject.ts` - exposes current-project context tools.
- `src/mcp/tools/tasks.ts` - exposes task-related MCP tools.
```

If the context file represents no code files, it must explain why.

Bad example:

```md
## Important Code Files

- many backend files
- UI files
- tests
```

That is too vague. Agents need concrete file paths.

## Important Boundaries

The `Important Boundaries` section tells agents what not to assume, where not to place information, and what must not be changed blindly.

It should include common traps and ownership rules.

Good example:

```md
## Important Boundaries

- Do not assume the UI owns task lifecycle truth.
- Do not treat archived docs as current implementation guidance.
- Do not change tool output that writes records without checking database and evidence context.
- Do not add broad context links just because files are nearby in the repository.
```

This section is important because many agent failures come from reasonable but wrong assumptions.

## Links To Other Context

Every context file must include `Links To Other Context`.

Each link must explain when the other context file should be read.

Good example:

```md
## Links To Other Context

- [Agent OS MCP](../Agent%20OS%20MCP/Agent%20OS%20MCP.md) - read when tool behavior depends on MCP registration or server execution.
- [Agent OS Database](../Agent%20OS%20Database/Agent%20OS%20Database.md) - read when the task changes stored records, migrations, or lifecycle truth.
- [Agent OS UI Tools](../Agent%20OS%20UI/Agent%20OS%20UI%20Tools.md) - read when tool behavior is shown or controlled in the app UI.
```

A context link means:

> If your task touches this relationship, read that context next.

Each link must explain the condition that makes the linked file relevant.

Good condition examples:

- read when this change reads, writes, migrates, or validates stored records
- read when this UI change changes the meaning of backend lifecycle state
- read when this task changes MCP registration, schemas, or tool execution

Links should not be random references. They should guide context expansion.

## Quality Rules

A context file must be:

- current-state focused
- specific about the system area
- specific about important code files
- clear about boundaries
- linked to related context only when useful
- dense enough to be useful
- compressed enough to avoid bloated explanation
- readable by agents with no prior project knowledge
- clear about which system owns truth for the area
- source-backed by code, migrations, tests, package scripts, or current backend behavior
- direct about partial, placeholder, read-only, disabled, hardcoded, or UI-only behavior

A context file is too weak if it only acts like an index.

A context file is too broad if a narrow task forces the agent to read many unrelated details.

## Truth Owner Rule

Each context file must identify what owns truth for the area.

Truth may be owned by one or more of:

- backend database records
- backend services or domain rules
- SQL migrations
- UI local state
- CLI output
- MCP output
- generated artifacts
- local runtime or pane state

If UI, CLI, MCP, generated reports, or terminal output only display or carry information, say that directly. Do not let agents mistake a display surface for lifecycle truth.

## Source-Backed Claim Rule

Every behavior claim should be traceable to source code, migration, test, package script, or current backend behavior.

If a claim cannot be verified from source, do not write it as fact. Say that the behavior is unclear and identify what must be inspected to confirm it.

Good:

> The UI currently displays the selected task readiness summary from backend-loaded task detail. Verify this in `TasksDetailView.tsx` and `src/services/taskReadiness.ts` before changing labels.

Bad:

> The UI controls task readiness.

## Partial Behavior Rule

If a feature is partial, placeholder, read-only, disabled, hardcoded, UI-only, or not fully wired, the context must say that directly.

Good:

> The Proof Actions surface is visible but disabled in the current UI; it must not be described as an active backend action path.

Bad:

> Proof Actions lets users export, review, and score task proof.

## Writing Style

Use:

- short paragraphs
- concrete file paths
- clear ownership statements
- direct descriptions of current behavior
- practical working guidance for future agents
- first-file inspection guidance when useful
- verification guidance inside `Working Guidance`
- explicit truth-owner language
- direct limitation wording for partial, placeholder, disabled, hardcoded, read-only, or UI-only behavior
- sub-headings inside `Description Details` when helpful
- links that explain why they matter

Avoid:

- vague strategy language
- long historical explanations
- task diary writing
- future plans
- old PRD language
- broad statements without file paths
- links without a reason

## Current-State Rule

Context must describe what Agent OS does now.

If something is not implemented, say that clearly.

Good:

> The UI currently shows memory surfaces, but candidate-to-active memory approval is not fully wired.

Bad:

> The memory system should approve candidates into active memory.

## Definition Of Done For A Context File

A context file is done when:

- it uses the standard section structure
- `Description Details` is detailed enough for agents to understand current behavior
- `Working Guidance` tells agents how to work safely in the area
- sub-headings are used if the area needs them
- important code files are concrete and current
- boundaries are explicit
- links point to useful related context
- it describes current behavior, not future plans
- a new agent can use it without reading the whole repository
