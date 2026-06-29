# Agent OS Managed Terminal Control Implementation

Date: 2026-06-28
Status: implementation planning package, compressed to 10 files
Owner: Stig
Audience: implementation agents who did not participate in the planning conversation

## Purpose

This folder translates the Agent OS / Warren planning discussion into implementation-ready product and technical notes.

The main product direction is:

```text
Agent OS should let Stig keep chatting in terminals, while turning those terminals into managed task workers when attached to Agent OS.
```

This is not a request to remove terminal workflows. It is a request to make terminal workflows safer and more controlled by Agent OS.

## Compression Note

This package was compressed from 16 Markdown files to 10 Markdown files without intentionally removing information.

Merged documents preserve their original content under `Source Files Merged` sections. The old split is represented in the mapping below so future agents can still trace where information came from.

## Read Order

Read these files in order:

1. `01 Executive Summary And Product Requirements.md`
2. `02 Current Baseline And Warren Comparison.md`
3. `03 Managed Terminal Worker Model.md`
4. `04 Work Package And Context System.md`
5. `05 Evidence Finalizer Repair And Risk.md`
6. `06 Backend Implementation Blueprint.md`
7. `07 UI Testing Acceptance And Rollout.md`
8. `08 Decisions Questions And Traceability.md`
9. `09 Implementation Handoff And Glossary.md`

## Former File Mapping

| Former file | Current file |
| --- | --- |
| `00 Executive Summary And Scope.md` | `01 Executive Summary And Product Requirements.md` |
| `01 Product Requirements.md` | `01 Executive Summary And Product Requirements.md` |
| `02 Current Agent OS Baseline.md` | `02 Current Baseline And Warren Comparison.md` |
| `07 Warren Comparison And Adopted Patterns.md` | `02 Current Baseline And Warren Comparison.md` |
| `03 Managed Terminal Worker Model.md` | `03 Managed Terminal Worker Model.md` |
| `04 Work Package And Context System.md` | `04 Work Package And Context System.md` |
| `05 Evidence Quality Gate And Result Finalizer.md` | `05 Evidence Finalizer Repair And Risk.md` |
| `06 Repair Approval And Risk Decisions.md` | `05 Evidence Finalizer Repair And Risk.md` |
| `08 Backend Implementation Blueprint.md` | `06 Backend Implementation Blueprint.md` |
| `09 UI Product Requirements.md` | `07 UI Testing Acceptance And Rollout.md` |
| `10 Testing Acceptance Criteria And Rollout.md` | `07 UI Testing Acceptance And Rollout.md` |
| `11 Decision Log And Open Questions.md` | `08 Decisions Questions And Traceability.md` |
| `12 Traceability Matrix.md` | `08 Decisions Questions And Traceability.md` |
| `13 Implementation Handoff Checklist.md` | `09 Implementation Handoff And Glossary.md` |
| `14 Glossary.md` | `09 Implementation Handoff And Glossary.md` |

## Independent Review Adjustment

An independent review confirmed the direction is strong, but implementation should not start as a broad build.

The first overall implementation target is MCP redesign. After the MCP agent-facing tool surface is redesigned, the first managed-terminal build target is a readiness-hardening slice:

```text
MCP redesign -> current-code mapping -> managed-label safety -> durable work package contract -> Result Finalizer contract -> repair-loop state -> acceptance test plan
```

Do not let current runtime tracking alone become the product meaning of managed work. Stig or Agent OS may open a terminal in Managed mode, but backend readiness decides whether the managed work is trusted, complete enough, and allowed to move forward.

## Implementation Principle

Do not start by rebuilding Agent OS.

The implementation should harden and extend the existing Agent OS foundations:

- task records
- runtime/pane binding
- launcher records
- agent-run contracts
- evidence/artifacts
- verification gates
- review/approval flow
- backend source-of-truth rules

The first implementation should make the smallest useful baseline of managed terminal control work end to end.

## What This Package Is

This package is:

- product requirements
- behavior specification
- implementation blueprint
- acceptance criteria
- risk list
- open-question list

This package is not:

- final database schema
- final UI design
- final API contract
- permission to skip project docs
- permission to change unrelated Agent OS behavior

## Key Outcome

After the baseline implementation, Stig should be able to use Agent OS in two ways:

```text
Path A: Start from Agent OS
Agent OS creates task -> prepares work package -> opens/starts managed terminal -> records evidence -> finalizes result.

Path B: Start from terminal
Stig chats in a terminal -> terminal attaches to Agent OS task -> Agent OS adds control/evidence/finalizer behavior.
```

Both paths should lead to the same backend-managed task lifecycle.

Important implementation rule:

```text
A terminal can be in Managed mode because Stig or Agent OS started it that way.
When meaningful work starts in a Managed terminal, Agent OS should create or attach a task automatically.
Managed work is trusted/ready only after Agent OS has task, role, durable work package, evidence destination, quality gate/result expectation, and Result Finalizer path for that run.
```
