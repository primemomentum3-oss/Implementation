# 01 — Product and UX Blueprint

## 1. Product definition

**Project Atlas** is a separate visual app that helps a user understand another project without needing to read the codebase directly.

It takes structured project knowledge from one or more sources:

- Agent-authored atlas files inside a target repo.
- Static scans of files, folders, package manifests, scripts, routes, configs, docs, schemas, hooks, MCP definitions, and workflows.
- Runtime traces and logs when available.
- Manual notes and explanations.

It then presents the project as an interactive, dark, zoomable, multi-lens system map.

The app is not just a diagram renderer. It is a reusable project-understanding workspace.

## 2. Primary user

The primary user is a non-coder or low-code/vibe-code builder who wants to understand:

- What systems exist in a project.
- What happens when a feature runs.
- Which parts depend on which other parts.
- Which files, scripts, tools, hooks, MCP tools, docs, and workflows support each part.
- What can break if something changes.
- Where to look when asking an agent to update or fix a feature.

## 3. Core UX principle

The user should always be able to move between these levels:

```text
Project
  -> System
    -> Feature
      -> Flow / pipeline
        -> Step
          -> Supporting artifacts
            -> File / function / script / hook / MCP tool / config / doc / trace
```

The app must never force the user into one giant graph.

Instead, the UI should use progressive disclosure:

- Start simple.
- Show only the most important objects.
- Allow drilling deeper only when the user chooses.
- Preserve breadcrumbs so the user never gets lost.
- Make all technical items readable with plain-language labels.

## 4. Visual direction

### 4.1 Overall feel

The app should feel like a premium control room / system observability app, not a plain documentation tool.

Style keywords:

- Dark.
- Calm.
- Spatial.
- High-contrast where needed.
- Soft-glow but not neon-heavy.
- Dense information, but never cluttered.
- Smooth transitions.
- Clear cards and panels.
- Focused detail panes.

### 4.2 Base theme

Use a dark theme with these design tokens:

```text
background.canvas:        #070A12
background.panel:         #0D111C
background.panelRaised:   #111827
background.card:          #151B2B
border.subtle:            rgba(255,255,255,0.08)
border.active:            rgba(125,180,255,0.55)
text.primary:             #F5F7FA
text.secondary:           #AAB4C5
text.muted:               #6F7B91
accent.primary:           #7DB4FF
accent.success:           #67E8A5
accent.warning:           #F6C85F
accent.danger:            #FF6B7A
accent.agent:             #B794F6
accent.data:              #4DD0E1
accent.runtime:           #F78FB3
```

Implementation may tune these values, but the design should stay dark and readable.

### 4.3 Typography

Use a clean sans-serif UI font and a monospace font only for file paths, commands, IDs, and code references.

Important rule:

- The main graph labels should use human names.
- File paths should appear as metadata, not as the main label.

Example:

```text
Good visible label: Payment Webhook Handler
Metadata: app/api/stripe/webhook/route.ts

Bad visible label: route.ts
```

## 5. Main app layout

The app should use a three-zone layout:

```text
+--------------------------------------------------------------------------------+
| Top Bar: Project switcher | Lens switcher | Search | Command palette | Status  |
+----------------------+---------------------------------------------------------+
| Left Sidebar         | Main Canvas / Timeline / Table / Explorer              |
| - Project map        |                                                         |
| - Lenses             |                                                         |
| - Saved views        |                                                         |
| - Flow list          |                                                         |
| - Docs list          |                                                         |
+----------------------+---------------------------------------------------------+
| Optional bottom inspector / trace playback / diff panel                         |
+--------------------------------------------------------------------------------+
```

A right-side detail drawer opens when a node, edge, flow step, artifact, or document is selected.

## 6. Navigation model

### 6.1 Top-level project home

The project home should show:

- Project name.
- Target repo/path/source.
- Last updated time.
- Current atlas version.
- Number of systems, flows, nodes, edges, docs, scripts, MCP tools, hooks, runtime traces.
- Health/status summary.
- Important warnings:
  - Low-confidence areas.
  - Missing docs.
  - Broken links.
  - Unknown dependencies.
  - Outdated scan.
- Entry cards:
  - System Map.
  - Feature Flows.
  - Data Lens.
  - Agent / MCP / Hooks Lens.
  - Docs Lens.
  - Folder Structure.
  - Scripts and Workflows.
  - Runtime Traces.
  - Change Impact.

### 6.2 Breadcrumbs

Every drill-down page should have breadcrumbs:

```text
Project Atlas > My SaaS App > Billing System > Stripe Webhook Flow > Update Subscription Step
```

Clicking any breadcrumb should go back to that level.

### 6.3 Command palette

Shortcut: `Cmd/Ctrl + K`

Should support:

- Jump to node.
- Jump to flow.
- Jump to file.
- Jump to doc.
- Change lens.
- Search all notes.
- Find scripts.
- Find MCP tools.
- Show impact of selected item.
- Open low-confidence items.
- Open recently changed items.

## 7. Lenses

A lens is a filtered and styled view over the same underlying project graph.

The same object can appear in many lenses.

Example:

`Stripe Webhook Handler` can appear in:

- System Map.
- Billing Flow.
- Runtime Trace.
- External Integrations.
- Change Impact.
- File/Folder Lens.

### 7.1 Overview Lens

Purpose:

Show the complete project at a friendly high level.

Visible node groups:

- User-facing surfaces.
- Backend/API.
- Database/data storage.
- Auth.
- External services.
- Jobs/workers.
- Agents/AI systems.
- MCP servers/tools.
- Deployment/runtime.

Avoid showing individual files unless the project is very small.

### 7.2 Flow Lens

Purpose:

Show what happens when a user action, automation, webhook, cron job, or agent operation runs.

Required visual modes:

1. **Timeline mode** — linear pipeline steps.
2. **Canvas mode** — nodes and connections.
3. **Sequence mode** — actor-to-actor message sequence.

Each step should show:

- Step number.
- Plain-language name.
- Trigger.
- Inputs.
- Outputs.
- Reads/writes.
- Supporting artifacts.
- External calls.
- Error paths.
- Evidence and confidence.

Example flow:

```text
User submits onboarding form
  -> Form validation
  -> Create account request
  -> Auth user creation
  -> Workspace record creation
  -> Default settings creation
  -> Welcome email
  -> Dashboard redirect
```

### 7.3 Data Lens

Purpose:

Show where data lives and how it moves.

Should support:

- Tables.
- Collections.
- Buckets/storage.
- Queues.
- Caches.
- Local storage.
- Config/state files.
- API payloads.
- Event payloads.

Views:

- Entity relationship view.
- Read/write overlay.
- Flow-specific data path.
- Sensitive-data marker.
- Data owner/source marker.

### 7.4 Dependency Lens

Purpose:

Show what depends on what.

Should support:

- App/module dependencies.
- Package dependencies.
- Internal service dependencies.
- Flow dependencies.
- Feature dependencies.
- External integration dependencies.
- Environment variable dependencies.

Important interaction:

Selecting a node should expose:

- Upstream dependencies: what this item needs.
- Downstream dependents: what may break if this item changes.

### 7.5 Agent / MCP / Hooks Lens

Purpose:

Make agent-related and tool-related automation visible.

Must support these object types:

- Agents.
- Agent instructions.
- AGENTS.md files.
- MCP servers.
- MCP tools.
- Tool inputs.
- Tool outputs.
- Tool permissions.
- Hooks.
- Webhooks.
- Git hooks.
- Framework hooks.
- Automation hooks.
- Background jobs.
- Scheduled jobs.
- Prompt files.
- Rules files.

Example drill-down:

```text
Project
  -> MCP Systems
    -> GitHub MCP Server
      -> Tools
        -> create_issue
        -> read_file
        -> create_pull_request
      -> Used by
        -> Planning Agent
        -> Review Agent
      -> Configured in
        -> .cursor/mcp.json
```

Hooks should be categorized because the word `hook` can mean different things:

- **Framework hook**: React hooks, lifecycle hooks, app router hooks.
- **Automation hook**: pre-commit, postinstall, deployment hooks.
- **Integration hook**: webhooks from Stripe, GitHub, Clerk, Supabase, etc.
- **Agent hook**: tool/event hooks used by coding agents.

### 7.6 Docs Lens

Purpose:

Make README files, AGENTS.md files, design docs, setup docs, API docs, and notes visible as first-class project objects.

For every document, show:

- Title.
- Path.
- Applies to which folder/system/feature.
- Summary.
- Important instructions.
- Last updated.
- Confidence/relevance.
- Broken links or outdated claims if known.

Special behavior for AGENTS.md:

- Show each AGENTS.md file as a scoped instruction source.
- Link it to the folder/subtree it applies to.
- Show instruction conflicts when possible.

### 7.7 Folder Structure Lens

Purpose:

Let the user see the target repo structure visually without reading every file.

Should support:

- Tree view.
- Treemap view.
- Graph-cluster view.
- Importance view.
- Recently changed files.
- Unmapped files.

Each folder should have a plain-language summary:

```text
app/api/billing
Purpose: Backend routes for billing and Stripe events.
Contains: API endpoints, webhook handler, subscription helpers.
Connected to: Stripe, users table, subscriptions table.
```

### 7.8 Scripts and Workflows Lens

Purpose:

Show commands and automation.

Must support:

- package.json scripts.
- pnpm/yarn/npm/bun scripts.
- Python scripts.
- shell scripts.
- Makefile tasks.
- Docker commands.
- GitHub Actions workflows.
- CI/CD pipelines.
- cron jobs.
- deployment commands.

For each script, show:

- Command.
- Plain-language purpose.
- Inputs.
- Outputs.
- Files touched.
- Systems affected.
- When to use it.
- Risk level.

### 7.9 Runtime Trace Lens

Purpose:

Show what actually happened during a real request/run.

Should support:

- Imported OpenTelemetry-like traces.
- Manual trace JSON.
- Agent-captured logs converted into trace steps.
- Flow replay.
- Timing and error markers.

Trace view should show:

```text
Trigger
  -> frontend event
  -> API endpoint
  -> auth check
  -> service function
  -> database write
  -> external API call
  -> response
```

The user should be able to compare:

- Expected flow.
- Observed flow.
- Missing step.
- Extra unexpected step.
- Failed step.

### 7.10 Deployment Lens

Purpose:

Show where the project runs.

Must support:

- Hosting provider.
- Backend runtime.
- Database.
- Storage.
- Queues.
- Cron jobs.
- Environment variables.
- Secrets names only, never values.
- Domains.
- Webhook URLs.
- CI/CD pipeline.

### 7.11 Change Impact Lens

Purpose:

Before changing something, show what might be affected.

When a user selects a node/file/system, show:

- Direct dependents.
- Indirect dependents.
- Flows containing this item.
- Tests/scripts related to this item.
- Docs that mention this item.
- Runtime traces that include this item.
- Risk level.
- Suggested verification checklist for an agent.

## 8. Core interaction patterns

### 8.1 Node detail drawer

Clicking any node opens a drawer with tabs:

1. **Summary**
2. **Flow usage**
3. **Dependencies**
4. **Artifacts**
5. **Data**
6. **Docs**
7. **Runtime traces**
8. **Agent notes**
9. **History**

Every node summary should include:

```text
Name
Type
Purpose
What it does
Why it exists
Inputs
Outputs
Reads
Writes
Triggers
Triggered by
Depends on
Used by
Related files
Related docs
Related scripts
Related MCP tools
Related hooks
Confidence
Evidence
Last updated
```

### 8.2 Edge detail drawer

Clicking an edge should explain the relationship.

Example:

```text
Dashboard Page -> Create Project API
Relationship: calls
When: user submits project form
Payload: project name, selected template, user token
Evidence: app/dashboard/new-project/page.tsx, app/api/projects/route.ts
Confidence: confirmed_by_code
```

### 8.3 Progressive zoom

Zoom levels should change what is visible:

- Zoomed out: only systems and major flows.
- Medium: components, integrations, data stores.
- Zoomed in: steps, files, scripts, docs, functions, config.

### 8.4 Focus mode

Selecting a node and pressing `F` should enter focus mode.

Focus mode shows:

- The selected node.
- Direct dependencies.
- Direct dependents.
- Flows involving the node.
- Docs and artifacts supporting it.

Everything else becomes dimmed or hidden.

### 8.5 Path mode

The user should be able to ask:

```text
Show path from User Signup Form to Database User Record
```

The app should show the shortest/most relevant path through the graph.

### 8.6 Compare mode

Compare two atlas snapshots:

- Before feature implementation.
- After feature implementation.

Show:

- Added nodes.
- Removed nodes.
- Changed nodes.
- Added edges.
- Removed edges.
- Flow changes.
- New risks.

## 9. Explanation notes

The app should include explanation notes, but notes should be attached to things rather than existing as one giant explanation document.

Every important object can have notes:

- Plain-language explanation.
- Technical explanation.
- Agent instruction.
- Risk note.
- Troubleshooting note.
- Business meaning.
- Example run.

Notes should support levels:

```text
explanation_level: simple | normal | detailed | technical
```

The UI should default to `normal` for the main user.

## 10. Search design

Search must search across:

- Node names.
- Flow names.
- Step names.
- File paths.
- Script names.
- Tool names.
- MCP servers.
- Hook names.
- Docs.
- Notes.
- Edge labels.
- Tags.
- Evidence sources.

Search result cards should show:

- Name.
- Type.
- Why it matched.
- Project area.
- Confidence.
- Quick actions:
  - Open.
  - Focus.
  - Show impact.
  - Show in flow.

## 11. Empty states

The app must handle incomplete atlas data well.

Examples:

- No runtime traces yet.
- No MCPs found.
- No docs mapped.
- No flows defined.
- Unknown dependencies.

Empty states should tell the agent/user exactly what to add next.

Example:

```text
No MCP tools mapped yet.
Ask an agent to inspect MCP configuration files and add MCP server/tool nodes to .project-atlas/atlas.bundle.json.
```

## 12. Confidence and evidence UI

Every generated item should show its source quality.

Confidence labels:

- `confirmed_by_code`
- `confirmed_by_config`
- `confirmed_by_schema`
- `confirmed_by_docs`
- `observed_runtime`
- `agent_inferred`
- `user_confirmed`
- `needs_review`

UI treatment:

- Green/blue for confirmed.
- Purple for agent-authored.
- Pink for runtime observed.
- Yellow for needs review.
- Red for broken/missing.

Evidence examples:

```text
Source: package.json
Lines: scripts.build
Reason: defines build command

Source: app/api/stripe/webhook/route.ts
Lines: 1-160
Reason: handles Stripe webhook event
```

## 13. Required visual components

Build these components early:

- `ProjectShell`
- `TopNavigation`
- `LensSwitcher`
- `CommandPalette`
- `GraphCanvas`
- `FlowTimeline`
- `NodeCard`
- `NodeDetailDrawer`
- `EdgeDetailDrawer`
- `EvidenceBadge`
- `ConfidenceBadge`
- `TagPill`
- `MiniMap`
- `BreadcrumbTrail`
- `FilterBar`
- `DocsViewer`
- `FolderTree`
- `ScriptList`
- `TraceReplay`
- `SnapshotDiffPanel`

## 14. User journeys

### Journey A: Understand a project for the first time

1. User imports/selects a target project.
2. App shows project home.
3. User opens Overview Lens.
4. User sees main systems.
5. User clicks a system.
6. Detail drawer explains purpose, dependencies, files, docs, flows.
7. User opens a flow.
8. Flow shows steps and supporting artifacts.
9. User opens a step and sees exactly what supports it.

### Journey B: Understand an MCP setup

1. User opens Agent / MCP / Hooks Lens.
2. App shows MCP servers.
3. User clicks one MCP server.
4. App shows tools, configs, consuming agents, permissions, and flows.
5. User clicks one tool.
6. Drawer shows inputs, outputs, consumers, and examples.

### Journey C: Understand what a script does

1. User opens Scripts Lens.
2. User searches `build`.
3. App shows package script, workflow usage, and affected systems.
4. User clicks script.
5. Drawer shows command, purpose, inputs, outputs, related files, and risks.

### Journey D: See what breaks if something changes

1. User selects a node or file.
2. User clicks `Show Impact`.
3. App highlights downstream dependents.
4. App lists flows, tests, scripts, docs, and runtime traces that include the item.
5. App creates a verification checklist that an implementation agent can follow.

### Journey E: Review project after an agent update

1. Agent updates the target project.
2. Agent also updates `.project-atlas/atlas.bundle.json`.
3. User re-imports or refreshes the atlas.
4. App shows diff from previous snapshot.
5. User sees new/changed nodes and flows.
6. User reviews low-confidence changes.

## 15. UX acceptance criteria

The first production-quality version is acceptable when:

- A non-coder can understand the app structure without opening source code.
- The graph is readable at high level and drillable at lower levels.
- Every major object has purpose, relationships, artifacts, evidence, and confidence.
- The app supports multiple lenses over the same data model.
- Agents can update the project atlas without changing the UI code.
- The app can handle missing/incomplete data gracefully.
- The dark UI feels intentional, polished, and easy to navigate.
