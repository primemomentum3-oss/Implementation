# 03 — Agent Project Setup Guide

This guide is for coding agents that set up or update Project Atlas data inside a target project.

The agent's job is not to explain the whole codebase in prose. The job is to create structured, reusable atlas data that the Project Atlas app can render visually.

## 1. Agent mission

When working in a target project, create and maintain a `.project-atlas/` folder that describes the project as systems, flows, nodes, edges, documents, artifacts, scripts, hooks, MCP tools, data models, deployment pieces, and runtime traces.

The output must help a non-coder understand:

- What exists.
- Why it exists.
- How it connects.
- What happens step by step.
- Which files/scripts/docs/tools support each step.
- What is confirmed vs inferred.
- What may break if something changes.

## 2. Key rule

Use plain-language labels for visible names.

Do this:

```json
{
  "id": "node_billing_stripe_webhook_handler",
  "type": "api_endpoint",
  "label": "Stripe Webhook Handler",
  "metadata": {
    "path": "app/api/stripe/webhook/route.ts"
  }
}
```

Do not make the file path the main label unless no better label exists.

## 3. Files the agent should create

Inside the target project:

```text
.project-atlas/
  atlas.project.json
  atlas.nodes.json
  atlas.edges.json
  atlas.flows.json
  atlas.documents.json
  atlas.artifacts.json
  atlas.traces.json
  atlas.views.json
  atlas.bundle.json
  notes/
  snapshots/
  agent-instructions.md
```

For the first version, the agent may write only `atlas.bundle.json` plus `agent-instructions.md`. Split files are better for maintainability.

## 4. First setup process

### Step 1 — Identify the target project

Gather:

- Project name.
- Project purpose.
- Repo root.
- Main app type.
- Languages/frameworks.
- Package manager.
- Main run/build/test commands.
- Deployment target if visible.
- External services if visible.

Write this into `atlas.project.json`.

### Step 2 — Scan project structure

Inspect:

```text
README files
AGENTS.md files
package manifests
src/app folders
api/routes/controllers
components
services/lib folders
database/schema/migrations
scripts
workflows
config files
MCP config files
hooks/webhooks
.env.example files
Docker/deployment files
```

Do not include secret values.

### Step 3 — Create high-level systems

Create L1/L2 nodes for the major systems.

Examples:

```text
Frontend App
Backend API
Authentication
Database
File Storage
Payment System
Email System
AI Features
Admin Area
Agent Tooling
MCP Tooling
Deployment
```

Use only systems that actually exist or are clearly planned in the target project.

### Step 4 — Create artifacts

Artifacts are concrete files/configs/scripts/docs.

Create artifacts for:

- Important folders.
- Important files.
- Scripts.
- Workflows.
- Config files.
- Docs.
- AGENTS.md files.
- Schema files.
- MCP configs.
- Hook definitions.
- Deployment files.

### Step 5 — Create nodes for important technical items

Create nodes for:

- User-facing pages/screens.
- API endpoints.
- Components.
- Services.
- Jobs.
- Scripts.
- Hooks.
- Webhooks.
- MCP servers.
- MCP tools.
- Agents.
- Database tables/entities.
- External services.
- Deployment targets.

Do not create a node for every tiny function unless it is important to understanding a flow.

### Step 6 — Create edges

Create relationships between nodes.

Common edges:

```text
contains
calls
reads
writes
triggers
depends_on
uses
configured_by
documents
runs
exposes_tool
uses_mcp_tool
handles_webhook
deploys_to
```

Every edge should have a short reason and evidence when possible.

### Step 7 — Create flows

Create flows for the important behaviors.

Include at least:

- Main user flows.
- Main API flows.
- Main background jobs.
- Main webhooks.
- Main agent/MCP flows.
- Main deployment/build/test workflows.

Each flow step should include:

- Name.
- Purpose.
- Trigger.
- Inputs.
- Outputs.
- Node refs.
- Artifact refs.
- Data reads/writes.
- Error paths if known.
- Evidence.
- Confidence.

### Step 8 — Create docs mapping

Every README, AGENTS.md, or important doc should become a document entry.

Map each doc to the system/folder/feature it applies to.

Example:

```json
{
  "id": "doc_agents_root",
  "title": "Root AGENTS.md",
  "path": "AGENTS.md",
  "appliesTo": ["node_project_root"],
  "summary": "Instructions for agents working across the whole repo.",
  "importantNotes": [
    "Follow repo-wide coding conventions.",
    "Run validation before completing changes."
  ]
}
```

### Step 9 — Bundle and validate

Create `atlas.bundle.json` that merges all atlas data.

The app must be able to import this file directly.

### Step 10 — Create first snapshot

Save a copy:

```text
.project-atlas/snapshots/YYYY-MM-DDTHH-mm-ss-initial.json
```

## 5. Update process after project changes

Every time an implementation agent changes the target project, it must update `.project-atlas` in the same work session.

Update checklist:

1. Did any new file/folder become important?
2. Did any route/API/page/component/service change?
3. Did any script or workflow change?
4. Did any AGENTS.md/README/docs change?
5. Did any database/schema/model change?
6. Did any MCP server/tool/config change?
7. Did any hook/webhook/background job change?
8. Did any feature flow change?
9. Did any dependency/relationship change?
10. Did confidence/evidence need to be updated?
11. Should a new snapshot be created?

The agent must update:

- Nodes.
- Edges.
- Flows.
- Artifacts.
- Documents.
- Views/layout hints if necessary.
- Bundle file.
- Snapshot.

## 6. Required writing style for atlas content

Use these rules:

- Plain names first, file paths second.
- Explain purpose in one or two sentences.
- Avoid raw code unless necessary.
- Prefer relationships over prose walls.
- Add evidence for claims.
- Mark uncertain claims as `agent_inferred` or `needs_review`.
- Do not invent unknown systems. Use `unknown` or `needs_review`.
- Do not include secrets.
- Do not include large code snippets.
- Use stable IDs that will not change just because a label changes.

## 7. ID naming convention

Use stable, readable IDs.

Pattern:

```text
node_<area>_<name>
edge_<source>__<relation>__<target>
flow_<area>_<name>
step_<flow>_<number>_<name>
artifact_<type>_<path_slug>
doc_<path_slug>
trace_<flow>_<timestamp>
```

Examples:

```text
node_billing_stripe_webhook_handler
node_mcp_github_server
node_mcp_github_create_pr_tool
node_hooks_pre_commit
node_data_users_table
flow_billing_process_stripe_webhook
step_billing_process_stripe_webhook_03_update_subscription
artifact_file_app_api_stripe_webhook_route_ts
doc_agents_root
```

## 8. Confidence rules

Use confidence honestly.

### `confirmed_by_code`

Use when the agent inspected code and the claim follows directly from code structure or references.

### `confirmed_by_config`

Use when the claim comes from config files like package.json, workflow YAML, MCP config, deployment config, or env example.

### `confirmed_by_schema`

Use when the claim comes from database schema, API schema, JSON schema, GraphQL schema, migrations, or typed models.

### `confirmed_by_docs`

Use when the claim comes from README, AGENTS.md, docs, comments, or project notes.

### `observed_runtime`

Use when the claim comes from actual logs/traces/test runs.

### `agent_inferred`

Use when the agent inferred the claim from names, folder structure, or common framework patterns.

### `user_confirmed`

Use when the user explicitly confirmed the meaning.

### `needs_review`

Use when the item is likely important but uncertain.

## 9. Evidence requirements

Every important node, edge, flow, and step should have evidence.

Evidence object should include:

```json
{
  "type": "file",
  "source": "app/api/stripe/webhook/route.ts",
  "locator": "route handler",
  "reason": "Defines the endpoint that receives Stripe webhook events."
}
```

Use source types:

```text
file
directory
config
document
schema
script
workflow
runtime_trace
manual_note
agent_inference
```

## 10. What to map by project area

### 10.1 Frontend

Map:

- Pages/screens/routes.
- Major components.
- Forms.
- Client actions.
- Data fetching boundaries.
- State stores.
- UI flows.
- Links to API endpoints.

Typical nodes:

```text
Dashboard Page
Settings Page
Create Project Form
Navigation Shell
Auth Guard
Client State Store
```

### 10.2 Backend/API

Map:

- API endpoints.
- Controllers/routes.
- Services.
- Middleware.
- Auth checks.
- Validation.
- External calls.
- Database reads/writes.

Typical nodes:

```text
Create Project API
Project Service
Auth Middleware
Rate Limit Middleware
Email Sender
```

### 10.3 Data

Map:

- Tables.
- Collections.
- Schemas.
- Models.
- Storage buckets.
- Queues.
- Caches.
- Migrations.

For each data node:

- What it stores.
- Who reads it.
- Who writes it.
- Sensitive data flag.
- Related flows.

### 10.4 Scripts and workflows

Map:

- Build command.
- Dev command.
- Test command.
- Lint command.
- Database migration command.
- Seed command.
- Deployment workflow.
- CI workflow.
- Cron/cleanup scripts.

For each script:

- Command.
- Purpose.
- When to run.
- Inputs.
- Outputs.
- Risks.

### 10.5 MCP systems

Map:

- MCP client/agent.
- MCP server.
- MCP tool.
- MCP config file.
- Command used to start server.
- Tool purpose.
- Tool inputs/outputs if known.
- Which agent or feature uses the tool.

Example hierarchy:

```text
Agent Tooling
  -> GitHub MCP Server
    -> read_repository tool
    -> create_issue tool
    -> create_pull_request tool
  -> Filesystem MCP Server
    -> read_file tool
    -> write_file tool
```

### 10.6 Hooks

Map hooks by category:

```text
Framework hooks
Automation hooks
Integration webhooks
Git hooks
Agent hooks
```

For each hook:

- Trigger.
- What runs.
- Inputs.
- Outputs.
- Related files.
- Related flows.
- Risk if broken.

### 10.7 Docs and AGENTS.md

Map docs as active system knowledge.

For each doc:

- Summary.
- Applies to.
- Important instructions.
- Outdated areas if known.
- Conflicts if known.

For AGENTS.md specifically:

- Scope.
- Rules.
- Commands.
- Validation requirements.
- Style requirements.
- Safety/restriction notes.

## 11. Flow creation rules

A flow should represent behavior the user cares about.

Good flows:

```text
User signs up
User creates project
Admin updates subscription
Stripe sends webhook
Agent opens pull request
CI runs build and tests
Nightly cleanup job runs
MCP tool reads repository file
```

Bad flows:

```text
Every function call in the codebase
Every import relationship
Every React render
```

The user needs clarity, not noise.

## 12. Flow step template

Every flow step should answer:

```text
What happens?
Who/what does it?
What triggers it?
What data comes in?
What data goes out?
What files/tools/scripts support it?
What can fail here?
Where is the evidence?
How confident are we?
```

JSON shape example:

```json
{
  "id": "step_create_project_03_auth_check",
  "label": "Check user authentication",
  "order": 3,
  "summary": "The API verifies the current user before creating a project.",
  "nodeRefs": ["node_auth_middleware", "node_api_create_project"],
  "artifactRefs": ["artifact_file_app_api_projects_route_ts"],
  "inputs": ["Session token", "Create project request"],
  "outputs": ["Authenticated user ID"],
  "reads": ["node_data_users_table"],
  "writes": [],
  "errorPaths": [
    {
      "condition": "User is not authenticated",
      "result": "Request is rejected"
    }
  ],
  "confidence": "confirmed_by_code",
  "evidence": [
    {
      "type": "file",
      "source": "app/api/projects/route.ts",
      "reason": "Route checks auth before creating project."
    }
  ]
}
```

## 13. Explanation notes format

When a concept needs extra explanation, create a note.

Path:

```text
.project-atlas/notes/<area>-<topic>.md
```

Note template:

```md
# Plain-language title

## Simple explanation

Explain this for a non-coder.

## Detailed explanation

Explain the moving pieces and relationships.

## Technical anchors

- Related nodes:
- Related flows:
- Related files:
- Related scripts:
- Related docs:

## Risks / common mistakes

List what usually breaks here.

## Agent update checklist

What an implementation agent must update if this area changes.
```

## 14. View/layout hints

Agents may add layout hints, but should not over-control the UI.

Good hints:

```json
{
  "nodeId": "node_frontend_dashboard",
  "lens": "overview",
  "group": "frontend",
  "rank": 1,
  "preferredColumn": "frontend"
}
```

Use columns:

```text
user
frontend
backend
business_logic
data
external
automation
agent_tools
deployment
```

## 15. Agent prompt: create initial atlas

Use this prompt when setting up a new project:

```text
You are creating Project Atlas data for this repository.

Create a .project-atlas folder that helps a non-coder understand the project visually.

Do not create one giant prose explanation. Create structured atlas data.

Required output:
1. atlas.project.json
2. atlas.nodes.json
3. atlas.edges.json
4. atlas.flows.json
5. atlas.documents.json
6. atlas.artifacts.json
7. atlas.views.json
8. atlas.bundle.json
9. agent-instructions.md
10. at least one snapshot

Map:
- high-level systems
- important features
- main flows
- data stores
- API routes
- pages/screens
- scripts
- workflows
- hooks/webhooks
- MCP servers/tools/configs
- agents/prompts/rules
- README/AGENTS/docs
- folder structure summaries
- deployment/config files

For every important node/edge/flow, include:
- plain-language label
- purpose
- evidence
- confidence
- related files/docs/scripts/tools

Use stable IDs.
Mark uncertain items as agent_inferred or needs_review.
Never include secret values.
```

## 16. Agent prompt: update atlas after a feature change

```text
You changed this project. Update .project-atlas before finishing.

Review the files changed in this session and update:
- nodes
- edges
- flows
- flow steps
- artifacts
- documents
- MCP/tool/hook/script mappings
- data read/write relationships
- confidence and evidence
- atlas.bundle.json
- snapshot

Also add a short update summary:
- what changed in the app
- what changed in the atlas
- what areas may be affected
- what the user should review in Project Atlas

Do not leave the atlas stale.
```

## 17. Agent prompt: review atlas quality

```text
Review .project-atlas for quality.

Check:
- Is every important system represented?
- Are the main user/API/automation/agent flows represented?
- Are MCP servers/tools mapped if present?
- Are hooks/webhooks mapped if present?
- Are scripts/workflows mapped?
- Are README and AGENTS.md files mapped?
- Are data stores and read/write relationships mapped?
- Are confidence labels honest?
- Does each important item have evidence?
- Are there outdated references?
- Are there broken node/edge/flow references?
- Is the atlas understandable for a non-coder?

Fix issues and regenerate atlas.bundle.json.
```

## 18. Quality checklist before finishing

The agent should answer yes to all:

- Can the Project Atlas app import `atlas.bundle.json`?
- Are all node IDs unique?
- Do all edges point to existing nodes?
- Do flow steps point to existing nodes/artifacts?
- Are important README/AGENTS/docs mapped?
- Are scripts and workflows mapped?
- Are hooks/webhooks mapped?
- Are MCPs/tools mapped if present?
- Are important data stores mapped?
- Are the main flows easy to follow?
- Are confidence labels honest?
- Are secret values excluded?
- Is there a snapshot?

## 19. What not to do

Do not:

- Dump the entire codebase into notes.
- Create a node for every tiny function by default.
- Use file paths as every label.
- Invent systems that do not exist.
- Mark inferred items as confirmed.
- Include `.env` secret values.
- Make one huge unreadable graph.
- Ignore AGENTS.md files.
- Ignore scripts/workflows.
- Ignore MCP/tooling configs.
- Forget to update the atlas after code changes.

## 20. Minimum viable atlas

A target project is minimally mapped when it has:

- Project metadata.
- 5-20 high-level system nodes.
- Main user/API/automation flows.
- Key data nodes.
- Important scripts/workflows.
- Docs/AGENTS mapping.
- Folder summaries.
- Evidence/confidence on important items.
- Bundle file.

## 21. Excellent atlas

A target project has an excellent atlas when:

- User can drill from project overview to any major feature.
- Every major feature has a flow.
- Every flow step links to supporting files/tools/docs/data.
- MCPs/tools/hooks/scripts are visible.
- Data reads/writes are understandable.
- Runtime traces exist for important flows.
- Impact analysis is useful.
- Snapshot diff shows changes after agent work.
- Low-confidence areas are clearly marked.
