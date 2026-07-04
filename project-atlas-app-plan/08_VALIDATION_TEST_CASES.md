# 08 — Validation and Test Cases

This document defines how the builder-agent should verify Project Atlas while building it.

It is not test code. It is the expected behavior framework for schema validation, graph behavior, UI behavior, canvas behavior, import behavior, and agent-maintained atlas quality.

## 1. Testing philosophy

Project Atlas is only useful if the user can trust what they see.

The app must therefore test three layers:

1. **Data correctness** — is the imported atlas bundle structurally valid?
2. **Graph correctness** — do nodes, edges, flows, docs, artifacts, and traces connect correctly?
3. **User clarity** — can a non-coder understand and navigate the project visually?

The builder-agent should not treat tests as only technical checks. Usability checks are part of correctness.

## 2. Required test fixtures

Create or use these fixtures during implementation:

| Fixture | Purpose |
|---|---|
| `07_EXAMPLE_ATLAS_BUNDLE.json` | Main happy-path demo. |
| `invalid-missing-project.json` | Missing required project metadata. |
| `invalid-duplicate-node-ids.json` | Duplicate node IDs. |
| `invalid-broken-edge.json` | Edge points to missing node. |
| `invalid-broken-flow-step-ref.json` | Flow step references missing node/artifact/document. |
| `warning-unmapped-docs.json` | Valid bundle with docs not connected to nodes. |
| `warning-low-confidence-critical-path.json` | Critical flow mostly marked `agent_inferred` or `needs_review`. |
| `large-synthetic-atlas.json` | Performance and graph-collapsing test. |
| `runtime-trace-comparison.json` | Expected flow plus observed trace mismatch. |

These fixtures may be added later as separate files. For the first build, the demo bundle is mandatory; the others can be created by the builder-agent as part of validation work.

## 3. Schema validation test cases

## 3.1 Valid demo bundle

### Input

`07_EXAMPLE_ATLAS_BUNDLE.json`

### Expected

- Validation passes.
- No blocking errors.
- Warnings may exist and should display clearly.
- Counts are displayed:
  - nodes
  - edges
  - flows
  - artifacts
  - documents
  - traces
  - warnings

### User-facing message

```text
Atlas bundle is valid. Some warnings need review.
```

## 3.2 Missing schema version

### Input

A bundle without `schemaVersion`.

### Expected

- Blocking validation error.
- App does not import as a normal project.
- Error explains that the schema version is required.

### Suggested fix shown to user/agent

```text
Add schemaVersion, for example "1.0.0", or regenerate the atlas bundle using the current Project Atlas contract.
```

## 3.3 Unsupported schema version

### Input

A bundle with `schemaVersion: "999.0.0"`.

### Expected

- Validation error or migration warning.
- App explains that the schema version is unsupported.
- If migration is impossible, import is blocked.

## 3.4 Missing project metadata

### Input

A bundle without `project.name`, `project.id`, or `project.summary`.

### Expected

- Blocking validation error.
- Error points to the missing field.
- Suggested fix asks the agent to update `atlas.project.json` or `atlas.bundle.json`.

## 3.5 Duplicate node IDs

### Input

Two nodes share the same `id`.

### Expected

- Blocking validation error.
- App lists the duplicated ID.
- App explains that graph relationships cannot be trusted with duplicate IDs.

## 3.6 Duplicate edge IDs

### Input

Two edges share the same `id`.

### Expected

- Blocking or high-severity validation error.
- App lists the duplicated edge ID.

## 3.7 Broken edge source

### Input

An edge has `sourceId` pointing to a missing node.

### Expected

- Blocking or high-severity validation error.
- App shows:
  - edge ID
  - missing source ID
  - relationship type
  - suggested fix

## 3.8 Broken edge target

### Input

An edge has `targetId` pointing to a missing node.

### Expected

Same as broken edge source.

## 3.9 Flow step references missing node

### Input

A flow step has `nodeRefs: ["node_missing"]`.

### Expected

- Validation error or high-severity warning.
- App identifies flow, step, and missing node reference.

## 3.10 Flow step references missing artifact

### Expected

- Warning or error depending on strictness.
- App can still open the flow, but the missing artifact is shown as unresolved.

## 3.11 Invalid confidence label

### Input

A node has `confidence: "probably_true"`.

### Expected

- Validation error.
- App lists allowed confidence values.

## 3.12 Secret values in environment variables

### Input

An env var node includes a real value or `valueIncluded: true`.

### Expected

- Critical validation error.
- App explains that secrets must never be stored in the atlas.
- Suggested fix: store only env var names and set `valueIncluded: false`.

## 4. Graph-engine test cases

## 4.1 Node lookup

### Input

Lookup `node_component_create_project_form` from demo bundle.

### Expected

Returns Create Project Form with:

- type: component
- area: frontend
- artifact ref: `artifact_file_create_project_form`
- connected flow: `flow_create_project`

## 4.2 Outgoing relationship traversal

### Input

Get outgoing relationships from `node_component_create_project_form`.

### Expected

Includes edge to `node_api_create_project` with type `calls`.

## 4.3 Incoming relationship traversal

### Input

Get incoming relationships to `node_api_create_project`.

### Expected

Includes edge from `node_component_create_project_form`.

## 4.4 Downstream impact

### Input

Compute impact for `node_component_create_project_form`.

### Expected

Impact report includes:

- Create Project API.
- Authentication.
- Project Service.
- Projects Table.
- Activity Logs Table.
- Create Project flow.
- Create Project Form artifact.

Risk should be at least medium because the form is part of a core flow.

## 4.5 Upstream dependencies

### Input

Get upstream dependencies for `node_table_projects`.

### Expected

Includes:

- Project Service.
- Create Project API.
- Create Project Form.

## 4.6 Flow usage index

### Input

Find flows using `node_external_email`.

### Expected

Includes `flow_create_project`.

## 4.7 Artifact usage index

### Input

Find nodes supported by `artifact_file_agents_md`.

### Expected

Includes:

- Root AGENTS.md.
- Implementation Agent.
- Agent Tooling.

## 4.8 Document usage index

### Input

Find nodes explained by `doc_agents_root`.

### Expected

Includes:

- Project root.
- Agent Tooling.
- Implementation Agent.
- Root AGENTS.md document node.

## 4.9 Lens graph: Overview

### Input

Build Overview Lens from demo bundle.

### Expected

Shows high-level systems and hides or collapses most L4/L5 details by default.

Must include:

- Demo Vibe SaaS App.
- Frontend App.
- Backend API.
- Project Database.
- Billing System.
- Agent Tooling.
- Vercel Deployment.

## 4.10 Lens graph: Agent/MCP/Hooks

### Input

Build Agent/MCP/Hooks Lens.

### Expected

Shows:

- Agent Tooling.
- Implementation Agent.
- GitHub MCP Server.
- Read Repository Tool.
- Create Pull Request Tool.
- Root AGENTS.md.
- Pre-commit Quality Hook.

## 4.11 Lens graph: Docs

### Input

Build Docs Lens.

### Expected

Shows README, AGENTS.md, and Project Atlas Agent Instructions.

Clicking AGENTS.md must be possible.

## 4.12 Lens graph: Data

### Input

Build Data Lens.

### Expected

Shows Project Database, Users Table, Projects Table, Activity Logs Table, and read/write edges from Project Service.

## 5. Canvas behavior test cases

## 5.1 Initial overview load

### Expected

When opening the demo project, the user sees a canvas-first overview, not a table or raw JSON.

The canvas should show major project areas as readable map regions/nodes.

## 5.2 Zoomed-out view

### Expected

At far zoom:

- Only major systems are visible.
- Labels are readable.
- Details like file paths are hidden.
- Graph does not look like a spaghetti hairball.

## 5.3 Medium zoom

### Expected

At medium zoom:

- Features, APIs, integrations, and data stores become visible.
- Confidence/warning badges are visible.
- Important edges appear with simple labels.

## 5.4 Close zoom

### Expected

At close zoom:

- Flow steps, scripts, docs, hooks, MCP tools, and artifacts can appear.
- File paths are visible as secondary metadata.
- The user can click detailed items.

## 5.5 Select node

### Input

User clicks `Create Project API`.

### Expected

- Node becomes selected.
- Connected edges highlight.
- Detail drawer opens.
- Drawer explains purpose, files, flows, dependencies, evidence, and confidence.

## 5.6 Select edge

### Input

User clicks edge from Create Project Form to Create Project API.

### Expected

- Edge detail opens.
- Relationship is explained as a form submitting a request to an API.
- Evidence/confidence is shown.

## 5.7 Focus mode

### Input

User focuses on Project Service.

### Expected

Canvas shows:

- Project Service.
- Upstream API/form.
- Downstream data tables/email service.
- Related flow.
- Unrelated systems dimmed or hidden.

## 5.8 Minimap

### Expected

Minimap shows current viewport and major project regions.

## 5.9 Empty lens state

### Input

Open a lens that has no matching items in a small project.

### Expected

The app explains what is missing and what the agent should add.

## 6. Detail drawer test cases

## 6.1 Node detail drawer

### Input

Click `Project Service`.

### Expected tabs/content

- Summary: explains business logic.
- Connections: shows Create Project API, Users Table, Projects Table, Activity Logs Table, Email Service.
- Flows: shows Create Project flow.
- Artifacts: shows Project Service File and Database Schema.
- Data: shows reads/writes.
- Impact: shows affected downstream systems.
- Evidence: confirms source.

## 6.2 Document drawer and viewer

### Input

Click `Root AGENTS.md`.

### Expected

- Drawer opens.
- Summary explains it is repo-wide agent instructions.
- Important notes are visible.
- Applies-to scope is visible.
- Document viewer can open Markdown preview or source reference.

## 6.3 MCP server detail

### Input

Click `GitHub MCP Server`.

### Expected

- Shows server purpose.
- Shows transport if available.
- Shows tools it exposes.
- Shows config file.
- Shows which agent uses it.

## 6.4 MCP tool detail

### Input

Click `Create Pull Request Tool`.

### Expected

- Shows tool purpose.
- Shows input/output summary.
- Shows used-by flow `Agent Creates Pull Request`.
- Confidence shows `agent_inferred`.

## 6.5 Script detail

### Input

Click `Build App`.

### Expected

- Shows command `pnpm build`.
- Shows when to use.
- Shows risk level.
- Shows CI workflow and deployment using it.

## 7. Flow lens test cases

## 7.1 Flow list

### Expected

Flow list shows:

- Create Project.
- Agent Creates Pull Request.
- Process Stripe Webhook.
- CI Build and Verification.
- Open AGENTS.md in Atlas.

## 7.2 Create Project timeline

### Expected

Timeline shows six steps in order:

1. Open dashboard.
2. Submit project form.
3. Call Create Project API.
4. Check authentication.
5. Create project records.
6. Send notification email.

Each step has connected nodes/artifacts and confidence.

## 7.3 Step detail

### Input

Click `Create project records` step.

### Expected

Shows:

- Project Service.
- Users Table read.
- Projects Table write.
- Activity Logs Table write.
- Project Service file.
- Database schema.

## 7.4 Agent Creates Pull Request flow

### Expected

Shows that the agent:

- Reads instructions.
- Uses GitHub MCP server.
- Updates `.project-atlas`.
- Creates pull request.

This is important because Project Atlas must visualize not only user-facing features but also agent/tooling systems.

## 8. Search and command palette test cases

## 8.1 Search AGENTS

### Input

Search `AGENTS`.

### Expected

Results include:

- Root AGENTS.md node.
- Root AGENTS.md document.
- Agent Creates Pull Request flow.
- Agent Tooling system.

## 8.2 Search MCP

### Input

Search `mcp`.

### Expected

Results include:

- GitHub MCP Server.
- Read Repository Tool.
- Create Pull Request Tool.
- MCP Configuration artifact.

## 8.3 Search build

### Input

Search `build`.

### Expected

Results include:

- Build App script.
- CI Build Workflow.
- Vercel Deployment.
- Package Manifest artifact.

## 8.4 Search create project

### Expected

Results include:

- Create Project flow.
- Create Project Form.
- Create Project API.
- Project Service.

## 9. Runtime trace test cases

## 9.1 Trace list

### Expected

Runtime lens shows `Create Project observed run`.

## 9.2 Trace replay

### Expected

Trace replay shows spans:

1. User submitted Create Project form.
2. Create Project API handled request.
3. Authentication checked user.
4. Project Service created records.
5. Email notification sent.

## 9.3 Expected vs observed

### Expected

The trace should map to Create Project flow steps. Any unmapped spans should be shown clearly.

## 10. Snapshot/diff test cases

## 10.1 Snapshot import

### Input

Import same demo bundle twice.

### Expected

- App stores snapshots.
- Diff shows no meaningful changes.

## 10.2 Added node diff

### Input

Modify fixture to add `Subscriptions Table`.

### Expected

- Diff shows added node.
- Data lens shows new table.
- Billing warning may disappear if fixed.

## 10.3 Changed confidence diff

### Input

Change MCP tool confidence from `agent_inferred` to `confirmed_by_config`.

### Expected

- Diff shows confidence change.
- Warning about inferred MCP tools can be reduced or removed.

## 11. Agent-maintained atlas quality tests

## 11.1 Atlas update after feature change

### Scenario

A target-project agent adds a new feature.

### Expected atlas updates

- New feature node.
- New or updated flow.
- Related files/artifacts.
- Related docs.
- Data reads/writes.
- Confidence/evidence.
- Snapshot.

## 11.2 Atlas update after docs change

### Scenario

AGENTS.md is changed.

### Expected atlas updates

- Document summary updated.
- Important notes updated.
- Evidence updated.
- Snapshot created.
- Docs Lens reflects change.

## 11.3 Atlas update after MCP config change

### Scenario

New MCP server added.

### Expected atlas updates

- MCP server node.
- MCP config artifact.
- Tool nodes if known.
- Edges from agents/clients to server.
- Agent/MCP/Hooks Lens updated.

## 12. Usability acceptance tests

These are subjective but important.

## 12.1 Five-minute understanding test

Give the app with demo bundle to a non-coder.

They should be able to answer within five minutes:

- What are the main systems?
- How does Create Project work?
- Where are AGENTS.md instructions?
- What MCP tools exist?
- What script builds the app?
- What might break if Project Service changes?

## 12.2 No-code navigation test

The user should not need to open source code to understand the major project behavior.

Source references can exist, but they should be secondary.

## 12.3 Visual overload test

At default zoom, the graph should not feel overwhelming.

If it does, the builder-agent must add grouping, filtering, or level-of-detail behavior.

## 13. Final release validation checklist

Before the builder-agent considers the first complete version ready, verify:

- Demo bundle validates.
- Invalid fixtures produce useful errors.
- Overview Lens renders.
- Flow Lens renders.
- Data Lens renders.
- Agent/MCP/Hooks Lens renders.
- Docs Lens renders.
- Folder Lens renders.
- Scripts Lens renders.
- Runtime Lens renders.
- Deployment Lens renders.
- Impact Lens renders.
- Search works.
- Command palette works.
- Node drawer works.
- Edge drawer works.
- Flow step drawer works.
- Document viewer works.
- AGENTS.md can be opened in-app.
- MCP server can be drilled into tools.
- Scripts show command/purpose/risk.
- Confidence/evidence badges are visible.
- Warnings are understandable.
- Snapshot diff works.
- Empty states guide the user/agent.
- App remains dark, readable, and visually pleasing.
