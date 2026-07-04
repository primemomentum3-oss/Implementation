# 06 — Component Specification

This document defines the major product components the builder-agent should create.

It is not implementation code. It is a component framework that explains responsibilities, expected behavior, data needs, and user-facing intent.

## 1. Component design philosophy

Project Atlas must feel like an interactive map of a project.

The UI should not feel like:

- A file explorer with extra icons.
- A static documentation site.
- A raw dependency graph.
- A developer-only architecture tool.

It should feel like:

- A dark 2D world map.
- A control-room dashboard.
- A zoomable knowledge canvas.
- A visual project operating system.

The user should be able to move from broad understanding to deep detail through clicking, zooming, filtering, and focusing.

## 2. Component hierarchy

Suggested high-level hierarchy:

```text
AppRoot
  ProjectLibrary
  ProjectWorkspace
    WorkspaceTopBar
    WorkspaceSidebar
    AtlasCanvasViewport
      GraphCanvas
      CanvasControls
      CanvasMiniMap
      ZoomLevelController
      ContextualOverlays
    DetailDrawer
    BottomInspector
    CommandPalette
    ImportValidationModal
```

The builder-agent may choose a different exact internal structure, but the user-facing responsibilities must remain.

## 3. Core layout components

## 3.1 AppRoot

### Purpose

Owns global app state, project storage, theme, routing, and error boundaries.

### Must provide

- Dark theme root.
- Project state provider.
- Atlas graph provider.
- Toast/notification provider.
- Error boundary.
- Keyboard shortcut provider.

### User-facing expectation

The app should always feel stable. If an imported atlas has errors, the app should explain the issue instead of crashing.

## 3.2 ProjectLibrary

### Purpose

Shows all imported project atlases.

### Must show

- Project name.
- Summary.
- Source/repo/path.
- Last imported/generated date.
- Number of nodes, edges, flows, docs, scripts, MCP tools, hooks, traces.
- Warning count.
- Confidence summary.
- Open project action.
- Import project action.

### Empty state

If no projects exist, show:

```text
Add your first Project Atlas bundle.
Ask an agent to create .project-atlas/atlas.bundle.json in your target project, then import it here.
```

## 3.3 ProjectWorkspace

### Purpose

Main shell for exploring one project.

### Layout

```text
Top bar
Left sidebar
Main canvas/work area
Right detail drawer
Optional bottom inspector
```

### Must coordinate

- Active project.
- Active lens.
- Selected node/edge/flow/step/document/artifact.
- Search query.
- Focus mode.
- Canvas state.
- Drawer state.

## 3.4 WorkspaceTopBar

### Purpose

Global controls while inside a project.

### Must include

- Project switcher.
- Active project name.
- Active lens switcher.
- Search/command palette trigger.
- Import/refresh action.
- Snapshot/diff action.
- Validation status indicator.
- Last updated indicator.

### Visual behavior

The top bar should be compact, glassy/dark, and persistent.

## 3.5 WorkspaceSidebar

### Purpose

Navigation and contextual list for the current project.

### Must include

- Lens list.
- Flow list.
- Saved views.
- Important systems.
- Warning/needs-review shortcuts.
- Recent items.

### Behavior

The sidebar should not be the main experience. It is a navigation aid for the canvas.

## 4. Canvas components

## 4.1 AtlasCanvasViewport

### Purpose

The container for the world-map exploration experience.

### Must include

- Graph canvas.
- Minimap.
- Zoom controls.
- Lens filters.
- Fit-to-view button.
- Focus-mode controls.
- Legend.
- Empty state overlay.

### User-facing expectation

The canvas should be the first thing the user naturally looks at.

## 4.2 GraphCanvas

### Purpose

Render atlas nodes and edges as a zoomable, pannable 2D world map.

### Must support

- Pan.
- Zoom.
- Node click.
- Edge click.
- Node hover preview.
- Edge hover preview.
- Connected-node highlighting.
- Dim unrelated nodes.
- Grouped/collapsed nodes.
- Lens-specific rendering.
- Zoom-level detail changes.
- Stable layouts.

### Required node states

- Default.
- Hovered.
- Selected.
- Focused.
- Dimmed.
- Warning.
- Low confidence.
- Runtime observed.
- Expanded group.
- Collapsed group.

### Required edge states

- Default.
- Hovered.
- Selected.
- Highlighted path.
- Dimmed.
- Warning.
- Runtime observed.

### Visual language

Use dark cards, soft borders, subtle glow, clear icons, and readable type.

Avoid noisy neon lines.

## 4.3 WorldMapLevelOfDetail

### Purpose

Controls what appears at each zoom level.

### Required behavior

At far zoom:

- Show only major systems/domains.
- Hide file paths.
- Hide most edge labels.
- Show group badges/counts.

At medium zoom:

- Show systems, features, APIs, data stores, external services.
- Show key edges.
- Show confidence/warning badges.

At close zoom:

- Show flow steps, artifacts, scripts, hooks, MCP tools, docs.
- Show summaries on selected/hovered nodes.
- Show more edge labels.

At deep zoom:

- Show metadata, file paths, document previews, command snippets, input/output summaries.

### Key rule

Zoom should reveal meaning, not just make text bigger.

## 4.4 CanvasMiniMap

### Purpose

Help the user understand location in the project world.

### Must show

- Full graph bounds.
- Current viewport rectangle.
- Major groups/areas.
- Selected node location.

### Behavior

Clicking/dragging the minimap should move the viewport.

## 4.5 CanvasControls

### Purpose

Quick actions on the canvas.

### Required controls

- Zoom in.
- Zoom out.
- Fit view.
- Reset layout.
- Toggle focus mode.
- Toggle labels.
- Toggle confidence overlays.
- Toggle evidence overlays.
- Open legend.

## 4.6 CanvasLegend

### Purpose

Explain colors, badges, node icons, edge types, and confidence states.

### Must explain

- Node types.
- Edge types.
- Confidence badges.
- Warning badges.
- Runtime observed marker.
- Agent inferred marker.
- Needs review marker.

## 5. Node components

## 5.1 AtlasNodeCard

### Purpose

Base visual card for all graph nodes.

### Must show

- Icon.
- Label.
- Type badge.
- Optional area/group.
- Confidence marker.
- Warning marker.
- Small metric badges when useful.

### Should optionally show

- Summary.
- Artifact count.
- Flow count.
- Doc count.
- Script count.
- MCP tool count.
- Hook count.

### Required interaction

- Click opens detail drawer.
- Hover highlights connected edges.
- Double-click or expand action drills deeper.

## 5.2 SystemNode

Represents major project areas.

Examples:

- Frontend App.
- Backend API.
- Database.
- Authentication.
- Agent Tooling.
- Deployment.

Must feel like a large map region.

## 5.3 FeatureNode

Represents a feature/domain inside a system.

Examples:

- Project Creation.
- Billing.
- User Management.
- Agent Review Flow.

Should show connected flows count.

## 5.4 FlowNode

Represents a user/API/automation/runtime process.

Should show:

- Flow type.
- Number of steps.
- Trigger.
- Runtime trace availability.

Click should open Flow Detail.

## 5.5 FlowStepNode

Represents one step in a flow.

Should show:

- Step number.
- Plain label.
- Inputs/outputs indicator.
- Reads/writes indicator.
- Error path marker if present.

## 5.6 ArtifactNode

Represents concrete project assets:

- Files.
- Folders.
- Configs.
- Scripts.
- Workflows.
- Schemas.
- Prompt files.
- Rule files.

Must show path as secondary metadata.

## 5.7 DocumentNode

Represents README, AGENTS.md, docs, notes, architecture docs.

Must show:

- Document type.
- Applies-to scope.
- Important notes count.
- Outdated/conflict warning if present.

Click should open document viewer inside the app.

## 5.8 McpServerNode

Represents an MCP server.

Must show:

- Server name.
- Transport if known.
- Tool count.
- Config file.
- Consuming agents/clients.

Double-click should reveal MCP tools.

## 5.9 McpToolNode

Represents an individual MCP tool.

Must show:

- Tool name.
- Purpose.
- Inputs/outputs summary.
- Used-by relationships.

## 5.10 HookNode

Represents framework hooks, webhooks, git hooks, package lifecycle hooks, agent hooks, and automation hooks.

Must show hook category clearly.

## 5.11 ScriptNode

Represents build/dev/test/lint/deploy/db/maintenance scripts.

Must show:

- Script name.
- Command summary.
- Risk level.
- Affected systems.

## 5.12 DataNode

Represents tables, collections, schemas, queues, caches, buckets, models.

Must show:

- Data type.
- Sensitive marker if relevant.
- Read count.
- Write count.

## 6. Edge components

## 6.1 AtlasEdge

### Purpose

Shows a meaningful relationship between two items.

### Required properties

- Direction.
- Type.
- Label if visible.
- Confidence state.
- Evidence state.

### Common edge meanings

- contains.
- calls.
- reads.
- writes.
- triggers.
- depends_on.
- documents.
- configured_by.
- exposes_tool.
- uses_mcp_tool.
- handles_webhook.
- deploys_to.

### Interaction

Clicking an edge opens an edge detail drawer explaining the relationship in plain language.

## 6.2 EdgeDetailPanel

Must show:

- Source item.
- Target item.
- Relationship type.
- Plain-language explanation.
- Evidence.
- Confidence.
- Related flows.

Example:

```text
Create Project Form submits request to Create Project API.
This relationship is confirmed by code and supports the Create Project flow.
```

## 7. Detail drawer components

## 7.1 DetailDrawer

### Purpose

Explain the selected item.

### Must support selected types

- Node.
- Edge.
- Flow.
- Flow step.
- Artifact.
- Document.
- Runtime trace.
- Warning.

### Must include

- Header.
- Type badge.
- Confidence badge.
- Evidence summary.
- Tabs.
- Related actions.

### Required actions

- Focus on canvas.
- Show impact.
- Open related flow.
- Open related docs.
- Copy agent instruction for this item.
- Show source reference.

## 7.2 SummaryTab

Must answer:

- What is this?
- Why does it exist?
- What area does it belong to?
- What does it connect to?
- What should the user know first?

## 7.3 ConnectionsTab

Must show:

- Upstream dependencies.
- Downstream dependents.
- Incoming edges.
- Outgoing edges.
- Relationship explanations.

## 7.4 FlowsTab

Must show:

- Flows containing this item.
- Flow steps containing this item.
- Trigger context.
- Role in each flow.

## 7.5 ArtifactsTab

Must show:

- Related files.
- Related folders.
- Related configs.
- Related scripts.
- Related workflows.

Each artifact should show path, summary, and confidence.

## 7.6 DocsTab

Must show:

- Related README files.
- AGENTS.md files.
- Architecture docs.
- Notes.
- Setup docs.

Clicking a document opens `DocumentViewer`.

## 7.7 DataTab

Must show:

- Reads.
- Writes.
- Creates.
- Updates.
- Deletes.
- Sensitive data markers.

## 7.8 McpToolsTab

Must show:

- MCP servers related to this item.
- MCP tools related to this item.
- Inputs/outputs.
- Which agents use them.

## 7.9 HooksTab

Must show:

- Hook category.
- Trigger.
- Handler.
- Event source.
- Related flow.

## 7.10 RuntimeTab

Must show:

- Runtime traces containing this item.
- Observed spans.
- Duration/status.
- Errors if present.

## 7.11 ImpactTab

Must show:

- Risk level.
- Direct dependents.
- Indirect dependents.
- Affected flows.
- Affected docs.
- Affected scripts/tests.
- Verification checklist.

## 8. Flow components

## 8.1 FlowList

Shows all flows with filters.

Must show:

- Flow name.
- Type.
- Area.
- Trigger.
- Step count.
- Confidence.
- Runtime trace availability.

## 8.2 FlowDetail

Shows one flow in multiple representations.

Modes:

1. Timeline.
2. Canvas.
3. Sequence.

## 8.3 FlowTimeline

### Purpose

The clearest non-coder representation of a flow.

### Must show

- Step number.
- Step name.
- Summary.
- Inputs.
- Outputs.
- Reads/writes.
- Supporting artifacts.
- Tools/hooks/scripts.
- Error paths.
- Confidence/evidence.

## 8.4 FlowCanvas

Shows flow steps spatially with supporting items around each step.

Example:

```text
[Submit Form]
  supported by: CreateProjectForm.tsx, Dashboard Page
  writes/calls: Create Project API

[Create Project API]
  supported by: route.ts
  reads: users
  writes: projects, activity_logs
```

## 8.5 FlowStepDetail

Clicking a step must explain:

- What happens.
- Who/what performs it.
- What comes in.
- What goes out.
- What data is read/written.
- What can fail.
- What files/tools/docs support it.

## 9. Lens components

## 9.1 LensSwitcher

Must make lens changes obvious and fast.

Each lens should have:

- Icon.
- Name.
- Short purpose.
- Count of visible items or relevant warnings.

## 9.2 LensFilterBar

Must support filters:

- Area.
- Node type.
- Edge type.
- Confidence.
- Tags.
- Levels.
- Show/hide inferred.
- Show/hide needs-review.
- Show only runtime-observed.

## 9.3 LensEmptyState

Every lens needs a useful empty state.

Example:

```text
No MCP tools are mapped yet.
This does not mean the project has no MCP tools. It means the atlas bundle does not currently include them.
Ask an agent to inspect MCP config files and update .project-atlas.
```

## 10. Document components

## 10.1 DocumentViewer

### Purpose

Open mapped docs inside the app.

### Must support

- Markdown rendering.
- Document summary.
- Path.
- Applies-to scope.
- Important notes.
- Related nodes/flows/artifacts.
- Confidence/evidence.
- Open source reference if available.

### Special handling: AGENTS.md

AGENTS.md files should show:

- Scope.
- Main rules.
- Commands.
- Validation requirements.
- Applies-to folder path.
- Conflicts with other agent docs if known.

## 10.2 MarkdownPreviewCard

Used inside drawers and search results.

Shows:

- Title.
- Short summary.
- Path.
- Applies-to area.
- Important notes count.

## 11. Folder and artifact components

## 11.1 FolderTree

Must show project structure in a readable tree.

Each folder can have:

- Purpose summary.
- Item count.
- Mapped/unmapped status.
- Related systems.

## 11.2 FolderTreemap

Optional but useful for visual mode.

Shows folders as map regions sized by number/importance of files.

## 11.3 ArtifactList

Must list files/configs/scripts/workflows/schemas with filters.

## 12. Search and command components

## 12.1 GlobalSearchIndex

Not a visible component, but essential.

Should index:

- Nodes.
- Edges.
- Flows.
- Steps.
- Artifacts.
- Documents.
- Notes.
- Traces.
- Warnings.

## 12.2 CommandPalette

Must support:

- Search.
- Navigate.
- Focus.
- Show impact.
- Switch lens.
- Open docs.
- Open flows.
- Filter by confidence.

## 12.3 SearchResultCard

Must show:

- Name.
- Type.
- Match reason.
- Area.
- Confidence.
- Quick actions.

## 13. Import and validation components

## 13.1 ImportProjectScreen

Must support importing/selecting:

- `atlas.bundle.json`.
- Later: `.project-atlas` folder.
- Later: zip export.

## 13.2 ValidationReport

Must show:

- Valid/invalid status.
- Node/edge/flow counts.
- Errors.
- Warnings.
- Suggested fixes.
- Open anyway option only when safe.

## 13.3 ValidationIssueCard

Each issue should be understandable to a non-coder.

Must show:

- Human-readable problem.
- Technical reference.
- Why it matters.
- Suggested fix for an agent.

## 14. Runtime components

## 14.1 TraceList

Shows imported/observed runtime traces.

## 14.2 TraceReplay

Shows actual execution as:

- Timeline.
- Span tree.
- Connected atlas nodes.
- Status markers.
- Duration markers.

## 14.3 ExpectedVsObservedFlow

Shows differences between the expected atlas flow and a runtime trace.

Must identify:

- Missing expected steps.
- Extra observed steps.
- Failed steps.
- Slow steps.
- Unmapped runtime spans.

## 15. Snapshot/diff components

## 15.1 SnapshotList

Shows imported snapshots.

## 15.2 SnapshotDiffView

Shows changed atlas data between versions.

Must show:

- Added/removed/changed nodes.
- Added/removed/changed edges.
- Added/removed/changed flows.
- Confidence changes.
- Evidence changes.

## 15.3 CanvasDiffOverlay

Optional but valuable.

Shows changes visually on the canvas.

## 16. Status and trust components

## 16.1 ConfidenceBadge

Must show trust level.

Labels:

- Confirmed by code.
- Confirmed by config.
- Confirmed by schema.
- Confirmed by docs.
- Observed runtime.
- Agent inferred.
- User confirmed.
- Needs review.

## 16.2 EvidenceBadge

Must indicate whether evidence exists.

Clicking opens evidence details.

## 16.3 WarningBadge

Shows issue severity.

## 16.4 HealthSummary

Shows project atlas health:

- Total warnings.
- Low-confidence items.
- Missing evidence.
- Broken references.
- Unmapped docs/artifacts.
- Runtime trace coverage.

## 17. Agent-helper components

## 17.1 AgentInstructionPanel

Shows copyable instructions for maintaining the atlas.

Must include:

- Create initial atlas prompt.
- Update after feature prompt.
- Review atlas quality prompt.
- Fix validation errors prompt.

## 17.2 AgentChecklistPanel

Shows context-specific checklist.

Example when viewing Billing System:

```text
If an agent changes billing, it should update:
- Billing system node.
- Stripe webhook flow.
- Payment data nodes.
- Related scripts/workflows.
- Related docs.
- Runtime traces if available.
```

## 18. Visual component quality rules

Every visible component should satisfy:

- Clear title.
- Clear type.
- Clear relationship to current context.
- Plain-language summary where possible.
- Confidence/evidence if it represents generated knowledge.
- Action to go deeper.
- Empty state if no data exists.

## 19. Accessibility and usability rules

- Text must be readable on dark backgrounds.
- Important actions must not rely on color alone.
- Badges should include labels, not only icons.
- Keyboard navigation should exist for search, command palette, drawer close, and canvas focus.
- Hover-only information should also be accessible by click.

## 20. Component build order

Recommended order:

1. AppRoot.
2. ProjectWorkspace shell.
3. Import/validation screen.
4. GraphCanvas basic rendering.
5. AtlasNodeCard.
6. AtlasEdge.
7. DetailDrawer.
8. FlowTimeline.
9. LensSwitcher and filters.
10. Docs viewer.
11. Folder tree.
12. Scripts list.
13. Agent/MCP/hooks lens.
14. Search/command palette.
15. Impact view.
16. Runtime trace view.
17. Snapshot diff.
18. Polish.

## 21. Final component standard

The component system is successful when the user can click almost anything and answer:

```text
What is this?
Where is it in the project world?
What does it connect to?
What supports it?
What uses it?
What evidence says this is true?
What should an agent update if this changes?
```
