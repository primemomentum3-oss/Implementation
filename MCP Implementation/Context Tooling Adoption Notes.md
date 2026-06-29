# Context Tooling Adoption Notes

## Purpose

This file reviews the downloaded context-tool skeleton and explains what should be adopted into the Agent OS MCP redesign.

Source reviewed:

```text
/Users/codex/Downloads/agent-os-context-cleanroom-setup/tools/context/
```

The reviewed package is useful, but it is explicitly a reference skeleton. It should not be copied into Agent OS as a new baseline subsystem.

## High-Level Decision

```text
Use the context-tool skeleton as inspiration.
Implement useful ideas through the agreed MCP tools and existing Agent OS backend records.
Do not create a parallel clean-room context system.
```

## Why Not Direct Import

Agent OS already has current backend records for many of the same concepts:

- `context_packets`
- `context_pack_provenance`
- `runtime_sessions`
- `runtime_runs`
- `agent_sessions`
- `file_leases`
- `task_events`
- `tool_invocations`
- `traces`
- `artifacts`
- `evidence_items`
- `command_runs`
- `patch_artifacts`
- proof history read models

The downloaded schema defines overlapping concepts such as source index runs, context packet revisions, agent sessions, file read events, context expansion requests, file leases, and audit events. Those concepts are useful, but the schema shape should not be installed directly because it collides with the current Agent OS model.

## Useful Concepts To Adopt

### Source Index

Useful for:

- `list_files`
- `search_files`
- `get_commands`
- `get_checks`
- work-package suggested files
- future context route quality checks

Adopted meaning:

Agent OS may build a helper index of source files, file kind, language, generated/runtime status, exported symbols, imports, likely tests, entrypoints, and package scripts.

Boundary:

The source index is helper/provenance data. It does not own lifecycle truth.

### File Classification

Useful classifications from the skeleton:

- source
- test
- package manifest
- database
- config/contract
- local README
- generated/runtime

Adopted meaning:

`list_files` and `search_files` should use this kind of classification to keep output useful and avoid generated/runtime noise.

### Search And Routing Reasons

Useful route evidence:

- path match
- symbol match
- direct import
- test-for-file
- config-for-file
- schema-for-file
- entrypoint-for-behavior
- package script connection
- context map route

Adopted meaning:

When Agent OS suggests or returns files, it should include a short reason and confidence when possible.

Example:

```text
src/settings.ts - path/content match for settings save task
src/settings.test.ts - likely test for selected settings file
```

### Read Files With Automatic Recording

Useful skeleton idea:

Every file read should be a recordable event tied to the task/session/packet.

Adopted meaning:

`read_files` should automatically record:

- path read
- task id if known
- runtime/session/run if known
- source of the read: attached context, search result, direct import, user requested, expansion
- timestamp
- whether the read was allowed, blocked, or partial

This should feed `get_task` continuation summaries.

### Context Expansion

Useful skeleton idea:

An agent often needs files outside the initial context packet.

Adopted meaning:

Agent OS should allow normal context expansion automatically and record it.

Important Stig decision:

```text
No block or warning is needed for normal project-file context expansion.
It should simply be noted that the agent searched/read other files.
```

Policy boundary:

Blocked/guarded paths are different from unsuggested paths. Generated/runtime folders, secrets, destructive locations, or project-external paths may still need blocking or approval.

### Packet Quality Checks

Useful checks from the skeleton:

- index/source is current
- all paths exist
- explanations are present
- read budget is respected
- implementation packet has core logic
- implementation packet has verification
- implementation packet is not docs-only unless the task is docs-only

Adopted meaning:

These should become backend warnings/quality signals, not hard blockers for normal work.

Examples:

```text
Context warning: no likely test file found.
Context warning: selected files span three areas; task may be broader than expected.
Context warning: packet has no verification suggestion.
```

### Command And Check Extraction

Useful skeleton idea:

Package scripts can be classified as test, lint, build, typecheck, or script.

Adopted meaning:

`get_commands` and `get_checks` can use package scripts as one input, together with Agent OS project settings, project docs, and command history.

Boundary:

Package scripts alone should not decide the final quality gate when Agent OS has stronger project settings or documentation.

## Concepts To Defer

### Separate Source-Index Database

Defer until the baseline MCP/managed-terminal flow proves the need.

First use artifacts/provenance or narrow records if existing tables are insufficient.

### Packet Revision Families

Useful later, but not first baseline.

Agent OS already has context packets and provenance. Add revision/family concepts only if implementation proves current records cannot support task continuation.

### Strict Access Controller

Do not adopt as-is.

The skeleton blocks reads outside the active context packet. Agent OS direction is different: allow normal context expansion and record it.

Possible future use:

- block generated/runtime/secrets/destructive paths
- prevent edits outside granted scope for implementation work
- warn or require review for risky expansion

### Clean-Room `docs/**` Exclusion

Do not adopt for Agent OS.

Agent OS currently uses `docs/context` as an active routing layer. Excluding docs would remove important source-of-truth context.

### `context:regressions` CLI

The skeleton README/package lists a regressions command, but the CLI does not implement it. Do not document or depend on it until implemented.

## Mapping To Confirmed MCP Tools

| Confirmed tool | Adopted context-tool inspiration |
| --- | --- |
| `list_files` | bounded file map, generated/runtime filtering, file kind classification |
| `search_files` | path/content/symbol/search reasons, simple source index, dependency/test hints |
| `read_files` | automatic file-read records, expansion source, partial/blocked status |
| `get_task` | attached context, previous reads/changes, expansion summary, context warnings |
| `get_commands` | package script extraction and command classification |
| `get_checks` | test/lint/build/typecheck classification, required/recommended/optional checks, quality-gate source alignment |
| `add_observation` | manual weak note only when automatic capture cannot represent useful context |
| `request_review` | should route managed-terminal work through Result Finalizer before review/approval |

## Mapping To Managed Terminal Work Packages

The context-tool ideas also help managed terminal work packages.

Useful additions:

- suggested files can come from task context plus search/index hints.
- suggested tests can come from likely test relationships.
- suggested commands can come from `get_commands` and package scripts.
- quality gate can come from `get_checks`, Agent OS settings, project docs, or fallback command history.
- context expansion during work should be recorded and included in finalizer details.

Important boundary:

The work package is a starting map, not a closed truth.

## Baseline Adoption Order

### Baseline 1 - File Tools And Recording

Implement:

- `list_files`
- `search_files`
- `read_files`
- automatic file-read recording
- expansion recording for normal project files

Reason:

This directly supports task continuation and managed-terminal context.

### Baseline 2 - Task Continuation Summary

Implement in `get_task`:

- attached context
- files read
- files changed
- checks run
- context expansion summary
- latest report/summary
- blockers/risks/next step

Reason:

This lets future agents continue without reconstructing history from raw logs.

### Baseline 3 - Commands And Checks

Implement:

- `get_commands`
- `get_checks`
- script classification
- quality-gate source alignment

Reason:

This supports proof and the managed-terminal Result Finalizer.

### Later Hardening

Implement later:

- source ownership map
- stale context warnings
- packet-size warnings
- context route quality scoring
- source index persistence
- context regression checks

Reason:

These are valuable, but they are hardening layers after the main managed-worker and MCP flow is stable.

## Implementation Guardrails

- Reuse existing Agent OS backend records first.
- Do not create a second lifecycle truth system.
- Do not treat generated helper index data as final proof.
- Do not block normal context expansion.
- Do not exclude `docs/context` from Agent OS routing.
- Do not expose low-confidence suggestions as certainty.
- Keep agent-facing outputs short and useful.
- Keep detailed raw provenance available for debugging and proof history.

## Review Conclusion

The downloaded tools are valuable because they describe the right kind of backend guardrails around context. They are not valuable as a direct copy.

Best use:

```text
Turn the good ideas into stronger behavior inside the current MCP redesign and managed-terminal work-package flow.
```
