# 02 — Technical Implementation Plan

## 1. System summary

Project Atlas is a standalone project-visualization app that reads structured atlas data about one or more target projects and renders it through interactive lenses.

It should be built so the target project does not need to be rewritten. The target project only needs one of these:

1. A generated `.project-atlas/atlas.bundle.json` file.
2. A `.project-atlas/` folder containing split atlas files that can be bundled.
3. A local scanner/agent process that outputs the bundle.
4. Optional runtime trace files.

The Project Atlas app consumes the bundle and visualizes it.

```text
Target Project Repo
  -> agent/scanner creates .project-atlas/*
  -> Project Atlas imports atlas bundle
  -> app renders visual lenses, flows, docs, details, traces, and impact maps
```

## 2. Product architecture

Use a modular architecture with clear boundaries:

```text
project-atlas/
  apps/
    web/                         # Main visual app
  packages/
    schema/                      # AtlasBundle schema, validators, migrations
    graph-engine/                # Graph normalization, traversal, impact analysis
    importers/                   # Import atlas files, local JSON, GitHub export, zip
    scanners/                    # Optional static scanners for common stacks
    ui/                          # Shared dark UI components
    agent-kit/                   # Agent prompts, setup templates, validation helpers
    renderers/                   # Graph, flow, docs, trace, folder renderers
  examples/
    demo-target-project/          # Example target project atlas
    demo-atlas-bundles/           # Example JSON bundles
```

If the implementation repo is not meant to contain the actual app yet, create this structure inside the implementation branch/folder chosen by the builder. The plan is independent of exact repo placement.

## 3. Recommended stack

### 3.1 Web app

Recommended default:

```text
TypeScript
Next.js or Vite React
React
Tailwind CSS
shadcn-style component primitives or custom primitives
React Flow / XYFlow-style graph canvas
Zustand or equivalent lightweight client state
TanStack Query or equivalent async state
Zod or equivalent schema validation
```

Reasoning:

- TypeScript makes the graph contract safer.
- A React-based UI is well suited for interactive panels, drawers, filters, and graph canvases.
- The atlas data should be validated before rendering.
- The graph engine should be framework-independent so it can be tested separately.

### 3.2 Storage options

Support three storage modes:

#### Mode A — Local bundle import MVP

User imports or points the app at:

```text
.project-atlas/atlas.bundle.json
```

The app loads it into browser/local app state.

This is the fastest MVP.

#### Mode B — Local workspace storage

The app stores imported projects locally:

```text
IndexedDB / SQLite / local app storage
```

This enables:

- Multiple projects.
- Snapshot history.
- Diffs.
- Saved views.
- User notes.

#### Mode C — Server-backed workspace

Optional later mode:

```text
Postgres / SQLite server / hosted database
```

Useful if multiple people share the same atlas.

### 3.3 Local project access

A normal web app cannot freely scan arbitrary local folders. Therefore, use one of these approaches:

#### Preferred first approach

Have agents generate atlas files inside the target repo. The Project Atlas app imports the generated JSON.

#### Preferred second approach

Build a small local CLI:

```bash
atlas scan /path/to/target-project --out /path/to/target-project/.project-atlas
atlas bundle /path/to/target-project/.project-atlas --out atlas.bundle.json
atlas validate atlas.bundle.json
```

#### Optional later approach

Package as a desktop app with local file access.

Possible wrappers:

- Tauri.
- Electron.
- Native local helper process.

Do not require desktop packaging for MVP.

## 4. Core data flow

```text
Target repo
  -> agent/scanner inspects repo
  -> writes .project-atlas split files
  -> bundle builder validates and merges files
  -> creates atlas.bundle.json
  -> Project Atlas imports bundle
  -> graph engine normalizes nodes/edges
  -> lenses render filtered graph views
  -> user selects items
  -> detail drawer shows explanations, artifacts, docs, evidence, traces
```

## 5. Target project folder contract

Agents/scanners should create this folder in target projects:

```text
.project-atlas/
  atlas.project.json             # Project metadata
  atlas.nodes.json               # Nodes
  atlas.edges.json               # Edges
  atlas.flows.json               # Flows and flow steps
  atlas.documents.json           # README, AGENTS.md, docs, notes
  atlas.artifacts.json           # Files, scripts, configs, routes, workflows
  atlas.traces.json              # Optional runtime traces
  atlas.views.json               # Saved lens views and layout hints
  atlas.bundle.json              # Built single-file export consumed by the app
  notes/
    *.md                         # Human/agent-written detailed notes
  snapshots/
    YYYY-MM-DDTHH-mm-ss.json     # Historical bundles for diffing
  agent-instructions.md          # Instructions for maintaining the atlas
```

For large projects, split files by area:

```text
.project-atlas/
  areas/
    billing.nodes.json
    billing.edges.json
    billing.flows.json
    onboarding.nodes.json
    onboarding.edges.json
    onboarding.flows.json
```

The app should support both single-file and split-file imports.

## 6. Atlas data model concept

The project is represented as a typed property graph.

### 6.1 Node

A node is any meaningful object:

- Project.
- System.
- Feature.
- Flow.
- Flow step.
- Folder.
- File.
- Function.
- Component.
- API route.
- Page.
- Database table.
- Storage bucket.
- Script.
- Hook.
- Webhook.
- MCP server.
- MCP tool.
- Agent.
- Prompt.
- Config.
- Environment variable.
- External service.
- Deployment target.
- Runtime span.
- Document.

### 6.2 Edge

An edge is a relationship between two nodes:

- contains
- calls
- reads
- writes
- creates
- updates
- deletes
- triggers
- triggered_by
- emits
- listens_to
- depends_on
- uses
- configured_by
- documents
- runs
- deploys_to
- handles_webhook
- exposes_tool
- uses_mcp_tool
- authenticates_with
- stores_in
- renders
- imports
- observed_as

### 6.3 Flow

A flow is an ordered or semi-ordered process:

- User action.
- API call.
- Webhook.
- Cron job.
- Agent action.
- CI/CD workflow.
- Data pipeline.
- Background job.

A flow contains steps. Each step can link to nodes, files, scripts, MCP tools, hooks, docs, and evidence.

### 6.4 Evidence

Evidence says why the app believes something is true.

Examples:

```text
Source file: package.json
Evidence type: config
Reason: script is defined in package.json scripts

Source file: AGENTS.md
Evidence type: docs
Reason: instruction applies to repo root

Source trace: trace-2026-07-04-create-project.json
Evidence type: runtime
Reason: span observed API call during real run
```

### 6.5 Confidence

Every node, edge, and flow should have confidence:

```text
confirmed_by_code
confirmed_by_config
confirmed_by_schema
confirmed_by_docs
observed_runtime
agent_inferred
user_confirmed
needs_review
```

## 7. Graph levels

Define fixed conceptual levels, but allow nodes to override.

```text
L0 — World / context
L1 — Major systems
L2 — Features / domains / bounded areas
L3 — Flows / pipelines
L4 — Steps / components / data stores / tools
L5 — Files / functions / scripts / configs / docs
```

Rendering rules:

- Overview lens defaults to L0-L2.
- Flow lens defaults to L3-L4.
- Deep inspection reveals L5.
- Folder lens can show L5 earlier because it is specifically for file navigation.

## 8. App routes / pages

Suggested app routes:

```text
/
  Landing or project selector

/projects
  List imported projects

/projects/:projectId
  Project home dashboard

/projects/:projectId/overview
  Overview lens

/projects/:projectId/flows
  Flow list

/projects/:projectId/flows/:flowId
  Flow detail timeline/canvas/sequence

/projects/:projectId/data
  Data lens

/projects/:projectId/dependencies
  Dependency lens

/projects/:projectId/agents
  Agent/MCP/hooks lens

/projects/:projectId/docs
  Docs lens

/projects/:projectId/folders
  Folder structure lens

/projects/:projectId/scripts
  Scripts and workflows lens

/projects/:projectId/runtime
  Runtime traces lens

/projects/:projectId/impact/:nodeId
  Change impact view

/projects/:projectId/snapshots
  Snapshot history and diff

/import
  Import atlas bundle/folder

/settings
  App settings
```

For a first MVP, route count can be smaller, but the internal model should anticipate all lenses.

## 9. Core packages

### 9.1 `packages/schema`

Responsibilities:

- Export TypeScript types.
- Export JSON Schema.
- Validate atlas bundles.
- Normalize legacy versions.
- Migrate bundle versions.
- Provide constants for node types, edge types, confidence labels, and lens names.

Functions:

```ts
validateAtlasBundle(input: unknown): ValidationResult<AtlasBundle>
normalizeAtlasBundle(bundle: AtlasBundle): NormalizedAtlasBundle
migrateAtlasBundle(bundle: unknown): AtlasBundle
assertNodeIdsUnique(bundle: AtlasBundle): ValidationIssue[]
assertEdgesReferToExistingNodes(bundle: AtlasBundle): ValidationIssue[]
assertFlowStepsReferToExistingNodes(bundle: AtlasBundle): ValidationIssue[]
```

### 9.2 `packages/graph-engine`

Responsibilities:

- Build adjacency indexes.
- Query upstream/downstream dependencies.
- Find paths.
- Build lens-specific subgraphs.
- Collapse/expand groups.
- Compute impact maps.
- Compute graph health warnings.
- Compute suggested layout groups.

Core APIs:

```ts
createGraph(bundle: NormalizedAtlasBundle): AtlasGraph
getNode(graph: AtlasGraph, nodeId: string): AtlasNode | undefined
getNeighbors(graph: AtlasGraph, nodeId: string, options?: NeighborOptions): GraphNeighborhood
getUpstream(graph: AtlasGraph, nodeId: string, depth?: number): AtlasNode[]
getDownstream(graph: AtlasGraph, nodeId: string, depth?: number): AtlasNode[]
findPaths(graph: AtlasGraph, fromId: string, toId: string): AtlasPath[]
buildLensGraph(graph: AtlasGraph, lens: LensId, filters: LensFilters): RenderGraph
computeImpact(graph: AtlasGraph, nodeId: string): ImpactReport
computeHealth(graph: AtlasGraph): HealthReport
```

### 9.3 `packages/importers`

Responsibilities:

- Import single bundle JSON.
- Import split `.project-atlas/` folder.
- Import zip export.
- Import remote raw JSON URL later.
- Merge split files.
- Validate and report errors.

Core APIs:

```ts
importAtlasBundle(file: File | string): Promise<AtlasBundle>
importAtlasFolder(files: FileList | LocalFolderRef): Promise<AtlasBundle>
mergeAtlasParts(parts: AtlasPart[]): AtlasBundle
```

### 9.4 `packages/scanners`

Optional helper package. Do not make MVP depend on perfect scanner output.

Scanners should produce partial atlas data that agents can improve.

Initial scanner targets:

- File tree scanner.
- README/docs scanner.
- AGENTS.md scanner.
- package.json scripts scanner.
- dependency manifest scanner.
- GitHub Actions workflow scanner.
- env example scanner.
- route scanner for common web frameworks.
- database schema scanner for obvious schema files.
- MCP config scanner.
- hook scanner.

Output type:

```ts
ScannerResult = {
  nodes: AtlasNode[];
  edges: AtlasEdge[];
  artifacts: AtlasArtifact[];
  documents: AtlasDocument[];
  warnings: ScannerWarning[];
}
```

### 9.5 `packages/agent-kit`

Responsibilities:

- Provide templates for agent setup.
- Provide validation checklist.
- Provide prompt files.
- Provide examples.
- Provide schema helper docs.
- Provide update workflow instructions.

Files:

```text
packages/agent-kit/
  templates/
    .project-atlas/
      atlas.project.json
      atlas.nodes.json
      atlas.edges.json
      atlas.flows.json
      atlas.documents.json
      atlas.artifacts.json
      atlas.views.json
      agent-instructions.md
  prompts/
    create-initial-atlas.md
    update-atlas-after-feature.md
    review-atlas-quality.md
    add-runtime-trace.md
  checklists/
    initial-map-checklist.md
    update-checklist.md
    confidence-checklist.md
```

### 9.6 `packages/ui`

Responsibilities:

- Design system tokens.
- Layout primitives.
- Cards.
- Badges.
- Drawers.
- Tabs.
- Search input.
- Command palette.
- Empty states.
- Markdown renderer.
- Dark theme.

### 9.7 `packages/renderers`

Responsibilities:

- Graph canvas rendering.
- Flow timeline rendering.
- Sequence rendering.
- Entity/data rendering.
- Folder tree rendering.
- Docs rendering.
- Runtime trace rendering.

## 10. UI state model

Use a central project session state:

```ts
type AtlasSessionState = {
  activeProjectId: string | null;
  activeLens: LensId;
  selectedNodeId: string | null;
  selectedEdgeId: string | null;
  selectedFlowId: string | null;
  selectedStepId: string | null;
  filters: LensFilters;
  zoomLevel: number;
  focusNodeId: string | null;
  expandedGroupIds: string[];
  collapsedGroupIds: string[];
  detailDrawerOpen: boolean;
  commandPaletteOpen: boolean;
  searchQuery: string;
  compareSnapshotId: string | null;
}
```

Keep graph data immutable after load. UI state should reference IDs, not duplicate graph objects.

## 11. Graph rendering strategy

### 11.1 Render graph creation

Do not render the raw graph directly.

Pipeline:

```text
AtlasBundle
  -> normalized graph
  -> lens filter
  -> level filter
  -> grouping/collapse
  -> layout hints
  -> render graph
  -> visual canvas
```

### 11.2 Node visual types

Define visual components by node type:

```text
ProjectNode
SystemNode
FeatureNode
FlowNode
StepNode
FileNode
FolderNode
ScriptNode
DocumentNode
DataNode
ExternalServiceNode
McpServerNode
McpToolNode
HookNode
AgentNode
RuntimeSpanNode
DeploymentNode
UnknownNode
```

Every node card should support:

- Icon.
- Label.
- Type badge.
- Confidence badge.
- Warning badge.
- Mini metrics.
- Expand/collapse affordance.

### 11.3 Edge visual types

Style edges by relation:

- `calls` — directional connection.
- `reads` — data read connection.
- `writes` — data write connection.
- `depends_on` — dependency connection.
- `triggers` — event/automation connection.
- `contains` — hierarchy connection.
- `documents` — dashed documentation connection.
- `observed_as` — runtime trace connection.

Edges should have labels, but labels may hide when zoomed out.

### 11.4 Layout hints

The schema should allow layout hints, but the app must still auto-layout if hints are missing.

Example:

```json
{
  "layout": {
    "group": "billing",
    "rank": 3,
    "preferredColumn": "backend",
    "pinned": false
  }
}
```

Suggested default columns for overview:

```text
User / Client
Frontend
API / Backend
Business Logic
Data
External Services
Automation / Jobs
Deployment
```

## 12. Lens engine

Each lens is a function that takes the normalized graph and returns a render model.

```ts
type LensDefinition = {
  id: LensId;
  label: string;
  description: string;
  defaultLevels: GraphLevel[];
  includedNodeTypes: AtlasNodeType[];
  includedEdgeTypes: AtlasEdgeType[];
  groupBy?: GroupingRule;
  defaultFilters?: LensFilters;
  build: (graph: AtlasGraph, filters: LensFilters) => RenderGraph;
}
```

Initial lenses:

```text
overview
data
flows
dependencies
agent_mcp_hooks
docs
folders
scripts
runtime
deployment
impact
```

## 13. Details drawer data composition

The drawer should not depend on the renderer. It should call a graph query API.

```ts
getNodeDetail(graph, nodeId) -> NodeDetailViewModel
```

Node detail view model:

```ts
type NodeDetailViewModel = {
  node: AtlasNode;
  summary: string;
  upstream: AtlasRelationSummary[];
  downstream: AtlasRelationSummary[];
  flows: FlowUsage[];
  artifacts: AtlasArtifact[];
  documents: AtlasDocument[];
  scripts: AtlasArtifact[];
  hooks: AtlasNode[];
  mcpTools: AtlasNode[];
  dataReads: AtlasNode[];
  dataWrites: AtlasNode[];
  runtimeSpans: AtlasTraceSpan[];
  evidence: Evidence[];
  warnings: AtlasWarning[];
  suggestedAgentChecklist: string[];
}
```

## 14. Import and validation UX

When importing a bundle, show a validation screen before opening the project.

Validation should check:

- Bundle schema version exists.
- Project metadata exists.
- Node IDs are unique.
- Edge source/target IDs exist.
- Flow step node references exist.
- Artifact references exist where possible.
- Confidence labels are valid.
- Evidence sources are valid enough.
- Required top-level fields are present.

Validation UI:

```text
Valid: 842 nodes, 1,940 edges, 31 flows, 128 docs/artifacts
Warnings: 14 low-confidence nodes, 9 missing evidence references
Errors: 0 blocking errors
```

If errors exist, allow opening in degraded mode only if graph can still render safely.

## 15. Snapshot and diff system

Every imported or generated atlas bundle should be saved as a snapshot.

Snapshot fields:

```ts
type AtlasSnapshot = {
  id: string;
  projectId: string;
  createdAt: string;
  source: 'imported' | 'agent_generated' | 'scanner_generated' | 'manual';
  bundleHash: string;
  summary: string;
  bundle: AtlasBundle;
}
```

Diff algorithm:

- Compare nodes by ID.
- Compare edges by ID or source/type/target signature.
- Compare flows by ID.
- Compare flow steps by ID.
- Compare artifacts by ID/path.
- Compare docs by ID/path.

Show:

- Added.
- Removed.
- Changed.
- Confidence changed.
- Evidence changed.
- Relationship changed.
- Flow order changed.

## 16. Impact analysis algorithm

When the user selects an item:

1. Find direct outgoing edges where selected node affects another node.
2. Find direct incoming edges where another node affects selected node.
3. Traverse downstream edges up to configurable depth.
4. Collect flows containing selected node or downstream nodes.
5. Collect artifacts linked to selected node and affected flows.
6. Collect docs mentioning selected node.
7. Collect tests/scripts/workflows connected to affected area.
8. Calculate risk score.

Risk score inputs:

```text
number_of_downstream_dependents
number_of_critical_flows
number_of_data_writes
runtime_observed_frequency
confidence_level
presence_of_tests
presence_of_docs
external_service_involvement
```

Output:

```ts
type ImpactReport = {
  selectedNodeId: string;
  riskLevel: 'low' | 'medium' | 'high' | 'critical' | 'unknown';
  directDependents: AtlasNode[];
  indirectDependents: AtlasNode[];
  affectedFlows: AtlasFlow[];
  affectedArtifacts: AtlasArtifact[];
  affectedDocuments: AtlasDocument[];
  affectedRuntimeTraces: AtlasTrace[];
  verificationChecklist: string[];
  explanation: string;
}
```

## 17. Runtime trace support

Runtime traces are optional but powerful.

Allow imported trace files in a simplified format:

```json
{
  "traceId": "trace_create_project_001",
  "name": "Create Project Flow - observed run",
  "startedAt": "2026-07-04T10:00:00Z",
  "durationMs": 842,
  "spans": [
    {
      "spanId": "span_001",
      "parentSpanId": null,
      "name": "User submitted form",
      "nodeRefs": ["node_dashboard_create_project_form"],
      "startedAt": "2026-07-04T10:00:00Z",
      "durationMs": 12,
      "status": "ok"
    }
  ]
}
```

Trace view should render:

- Tree of spans.
- Timeline bars.
- Error markers.
- Linked project nodes.
- Comparison to expected flow.

## 18. Agent integration strategy

Project Atlas should not require a special AI API in the app for MVP.

Instead:

- Agents working in the target repo create/update `.project-atlas/*` files.
- Project Atlas validates and renders those files.
- Later, the app can provide copyable prompts and checklists for agents.
- Later still, the app can integrate directly with an agent runner.

This keeps the app reusable and avoids coupling it to one agent platform.

## 19. Static scanner strategy

Scanners should be helpers, not the truth.

The scanner can confidently detect:

- File/folder structure.
- Package scripts.
- Package dependencies.
- GitHub Actions workflows.
- README/AGENTS docs.
- Obvious config files.
- Obvious route files.
- Obvious MCP config files.
- Obvious hooks.

The scanner usually cannot fully know business meaning. Agents add that.

Scanner output should mark many items as:

```text
confirmed_by_config
confirmed_by_docs
agent_inferred
needs_review
```

## 20. Scanner detection targets

### 20.1 Generic repo scanner

Detect:

```text
README.md
AGENTS.md
package.json
pnpm-lock.yaml
yarn.lock
package-lock.json
bun.lockb
pyproject.toml
requirements.txt
Cargo.toml
go.mod
Dockerfile
docker-compose.yml
.env.example
.github/workflows/*.yml
.gitignore
```

### 20.2 Scripts scanner

From package manifests:

- package.json scripts.
- pyproject scripts.
- Makefile targets.
- shell scripts.
- CI workflow jobs.

### 20.3 Docs scanner

Detect:

- README files.
- AGENTS.md files.
- docs folder.
- architecture docs.
- setup docs.
- API docs.
- changelogs.
- decision records.

### 20.4 MCP scanner

Detect config files that define MCP servers/tools. Do not hardcode only one IDE/tool vendor. Use a flexible pattern:

- Known MCP config filenames.
- JSON/TOML/YAML keys containing `mcpServers`, `mcp`, `tools`, `server`, `command`, `args`.
- Agent settings folders.
- Project-specific MCP docs.

The scanner should create nodes for:

- MCP server.
- MCP tool if tool names are listed.
- MCP config file.
- Agent or client that uses it if known.

### 20.5 Hook scanner

Detect categories:

- Git hooks.
- package lifecycle scripts.
- pre-commit configs.
- framework hook files.
- webhook route files.
- CI/CD triggers.
- agent automation hooks.

### 20.6 Route/API scanner

Detect common patterns but allow override:

- `app/api/**/route.ts`
- `pages/api/**`
- `src/routes/**`
- `server/routes/**`
- `api/**`
- `controllers/**`
- framework-specific route config

### 20.7 Data scanner

Detect:

- Prisma schema.
- SQL migration files.
- Drizzle schema files.
- Supabase migrations.
- schema.graphql.
- OpenAPI files.
- JSON schema files.
- database docs.

## 21. Security and privacy

Default posture: local-first and no source-code upload.

Rules:

- Never store raw secret values.
- Environment variables may be represented by name only.
- `.env` values must be ignored/redacted.
- Support `.atlasignore` to exclude paths.
- Mark sensitive files as sensitive.
- Allow target project to choose whether file snippets are included.
- Prefer storing file paths and summaries over full code.

Suggested `.atlasignore` support:

```text
.env
.env.*
node_modules
.git
.next
dist
build
coverage
*.pem
*.key
secrets/**
```

## 22. Performance requirements

The app must handle medium-large projects without becoming unusable.

Targets:

- 5,000 nodes should be importable.
- 20,000 edges should be importable.
- Overview lens should default to a much smaller collapsed graph.
- Search should remain responsive.
- Details drawer should load quickly for selected nodes.

Performance strategies:

- Normalize graph once.
- Build indexes once.
- Use memoized lens graph generation.
- Virtualize long lists.
- Collapse groups by default.
- Avoid rendering thousands of canvas nodes at once.
- Use progressive reveal and filters.

## 23. Testing strategy

### 23.1 Unit tests

Test:

- Schema validation.
- Graph normalization.
- Edge integrity.
- Lens filtering.
- Impact analysis.
- Path finding.
- Snapshot diff.

### 23.2 Fixture tests

Create fixture bundles:

```text
small-saas-app.bundle.json
mcp-heavy-project.bundle.json
docs-heavy-project.bundle.json
runtime-trace-project.bundle.json
broken-links-project.bundle.json
large-synthetic-project.bundle.json
```

### 23.3 UI tests

Test:

- Import flow.
- Lens switching.
- Node selection.
- Drawer tabs.
- Flow timeline.
- Search.
- Impact view.
- Snapshot diff.

## 24. MVP build plan

### Phase 0 — Project setup

Deliverables:

- App scaffold.
- TypeScript configured.
- Dark theme tokens.
- Routing.
- Basic layout shell.
- Example atlas bundle fixture.

Acceptance:

- App opens.
- Dark UI shell visible.
- Demo project selectable.

### Phase 1 — Schema and validation

Deliverables:

- `AtlasBundle` types.
- Runtime validator.
- Validation report UI.
- Example valid and invalid bundles.

Acceptance:

- App can load a JSON bundle.
- App shows validation warnings/errors.
- Invalid IDs are caught.

### Phase 2 — Graph engine

Deliverables:

- Normalize nodes/edges.
- Build indexes.
- Query node details.
- Upstream/downstream traversal.
- Lens graph builder.

Acceptance:

- Test fixture graph can be queried.
- Selecting a node returns dependencies and dependents.

### Phase 3 — Overview canvas

Deliverables:

- Graph canvas.
- Node cards.
- Edge rendering.
- Minimap.
- Zoom/pan.
- Select node.
- Detail drawer summary.

Acceptance:

- User can understand demo project at system level.
- Node click opens useful details.

### Phase 4 — Flow lens

Deliverables:

- Flow list.
- Flow detail page.
- Timeline mode.
- Canvas mode.
- Step detail drawer.

Acceptance:

- User can open a flow and see step-by-step behavior.
- Each step links to supporting nodes/artifacts.

### Phase 5 — Artifact/doc/folder lenses

Deliverables:

- Docs viewer.
- Folder tree.
- Artifact list.
- Scripts list.
- AGENTS.md mapping.

Acceptance:

- User can see README/AGENTS/docs and which systems they apply to.
- User can see package scripts and their purpose.

### Phase 6 — Agent/MCP/hooks lens

Deliverables:

- Agent nodes.
- MCP server/tool nodes.
- Hook nodes.
- Config links.
- Tool input/output display.

Acceptance:

- User can drill from project to MCP server to tools to flows using those tools.

### Phase 7 — Search and command palette

Deliverables:

- Global search index.
- Command palette.
- Jump/focus/open actions.

Acceptance:

- User can find any node, doc, script, flow, MCP tool, or file quickly.

### Phase 8 — Impact analysis

Deliverables:

- Impact traversal.
- Impact page.
- Risk score.
- Verification checklist.

Acceptance:

- User can select a node and see what may break if it changes.

### Phase 9 — Snapshots and diff

Deliverables:

- Save imported snapshots.
- Compare snapshots.
- Diff view.

Acceptance:

- User can compare before/after an agent update.

### Phase 10 — Agent kit and CLI

Deliverables:

- `.project-atlas` templates.
- Agent setup prompts.
- Bundle builder.
- Validator CLI.
- Optional basic scanner.

Acceptance:

- An agent can set up a target repo by following the guide.
- The generated bundle imports into the app.

## 25. Suggested actual build order

For fastest useful result:

1. Create demo `atlas.bundle.json` manually.
2. Build dark UI shell.
3. Build bundle import and validation.
4. Build graph engine.
5. Build overview graph.
6. Build node drawer.
7. Build flow timeline.
8. Build docs/folder/scripts lists.
9. Build agent/MCP/hooks lens.
10. Build search.
11. Build impact view.
12. Build agent setup templates.
13. Build scanner/CLI.

## 26. Example demo project to include

Create a fake but realistic demo bundle:

```text
Demo SaaS Project
  Frontend
    Dashboard Page
    Create Project Form
  Backend
    Create Project API
    Project Service
  Data
    users table
    projects table
    activity_logs table
  External
    Email service
    AI API
  Agent/MCP
    Planning Agent
    GitHub MCP Server
    File System MCP Server
  Automation
    Nightly Cleanup Job
    GitHub Actions Build Workflow
```

Flows:

- Create Project.
- Invite User.
- Process Payment Webhook.
- Nightly Cleanup.
- Agent Creates Pull Request.

This demo should exercise every major feature.

## 27. Quality bar

This app is successful when a user can say:

```text
I can open any project, see the project as a system, understand flows step by step, drill into supporting files/tools/docs, see MCPs/hooks/scripts, and ask an agent to update the atlas after changes.
```

The app should prioritize clarity over perfect automatic extraction.

The most important engineering decision is the reusable data contract. If the contract is good, agents and scanners can keep improving the maps without rewriting the UI.
