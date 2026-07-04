# 10 — Agent Build Prompt

This is the master prompt to give to the implementation agent that will build Project Atlas.

It tells the agent how to reason, what to build, what to preserve, and how to verify progress.

The prompt should be used together with all other files in this folder.

## 1. Master build prompt

```text
You are the implementation agent responsible for building Project Atlas.

Project Atlas is a standalone, dark, visually pleasing, canvas-first application for helping a non-coder or vibe coder understand complex software projects.

The app imports structured atlas data from another target project, usually from .project-atlas/atlas.bundle.json, and turns that data into an interactive, zoomable 2D world map of the project.

The user must be able to:
- Add/import a project atlas bundle.
- See the whole project as a visual system map.
- Zoom out to see major systems.
- Zoom in to see features, flows, steps, files, docs, scripts, hooks, MCP servers/tools, agents, data, deployment, runtime traces, and dependencies.
- Click any item and understand what it is, why it exists, where it fits, what supports it, what depends on it, and what evidence supports it.
- Open mapped Markdown documents such as README.md and AGENTS.md inside the app.
- Search everything without knowing code structure.
- Review flows step by step.
- See what may break if something changes.
- Compare atlas snapshots over time.
- Give clear instructions to another agent for keeping the atlas updated in target projects.

You are not building a generic diagram toy.
You are building a visual project-understanding system.

Read and follow these documents in order:
1. project-atlas-app-plan/README.md
2. project-atlas-app-plan/01_PRODUCT_AND_UX_BLUEPRINT.md
3. project-atlas-app-plan/02_TECHNICAL_IMPLEMENTATION_PLAN.md
4. project-atlas-app-plan/03_AGENT_PROJECT_SETUP_GUIDE.md
5. project-atlas-app-plan/04_ATLAS_DATA_CONTRACT.md
6. project-atlas-app-plan/05_IMPLEMENTATION_TASK_BACKLOG.md
7. project-atlas-app-plan/06_COMPONENT_SPECIFICATION.md
8. project-atlas-app-plan/07_EXAMPLE_ATLAS_BUNDLE.json
9. project-atlas-app-plan/08_VALIDATION_TEST_CASES.md
10. project-atlas-app-plan/09_UI_SCREEN_BY_SCREEN_SPEC.md
11. project-atlas-app-plan/10_AGENT_BUILD_PROMPT.md

Do not skip the example atlas bundle. It is the concrete fixture you should use to build and verify the UI.

Your job is to implement the app, but preserve the product intent exactly:
- Dark visual workspace.
- Canvas-first world-map experience.
- Progressive drill-down.
- Multiple lenses over the same graph.
- Plain-language labels.
- Evidence and confidence markers.
- Agent-maintainable atlas contract.
- Non-coder usability.

Build in phases. After each phase, verify using the demo bundle and summarize what user capability now works.
```

## 2. Builder-agent mental model

Use this mental model while building:

```text
Project Atlas is Google Maps for a code project.

Zoomed out:
  You see continents: frontend, backend, data, agent tooling, external services, deployment.

Medium zoom:
  You see countries/cities: features, APIs, services, tables, flows, MCP servers, scripts.

Zoomed in:
  You see buildings: files, docs, AGENTS.md, hooks, MCP tools, commands, configs, trace spans.

Clicking anything:
  Opens a guidebook explaining what it is, where it fits, what it connects to, and how trustworthy the information is.
```

Do not reduce this to a normal node graph. It must feel like a navigable project world.

## 3. Product essence to preserve

The user has little or no coding knowledge.

The app must therefore answer questions in the user's language:

```text
What is this?
What does it do?
Why does it matter?
What is connected to it?
What happens before and after it?
What files/docs/scripts/tools support it?
What can break if it changes?
How sure are we?
Where did this information come from?
```

The app should make technical projects feel visible and reviewable.

## 4. Non-negotiable requirements

The implementation must include:

1. Atlas bundle import.
2. Atlas bundle validation.
3. Demo bundle support.
4. Canvas-first project overview.
5. Zoom/pan/minimap/focus behavior.
6. Node and edge detail drawers.
7. Flow list and flow detail timeline.
8. Lens system.
9. Docs/Markdown viewer.
10. AGENTS.md visibility and scope display.
11. Folder/artifact visibility.
12. Scripts/workflows visibility.
13. Agent/MCP/hooks visibility.
14. Data lens.
15. Dependency/impact analysis.
16. Search/command palette.
17. Confidence/evidence badges.
18. Warning/health system.
19. Snapshot/diff foundation.
20. Agent setup/update prompt surfaces.

If time or scope forces tradeoffs, preserve the data contract, graph engine, canvas, detail drawer, flow lens, docs viewer, and search first.

## 5. Things not to do

Do not:

- Build only a static documentation website.
- Build only a raw file explorer.
- Build only a Mermaid renderer.
- Build only a dependency graph.
- Make the user read raw JSON.
- Make file paths the main labels everywhere.
- Render the entire codebase as one huge hairball.
- Hide confidence/evidence.
- Ignore AGENTS.md.
- Ignore MCPs/tools/hooks/scripts.
- Hardcode the demo project as the only possible project.
- Couple the app to one project framework.

## 6. Recommended implementation order

Follow this order unless the existing repo structure strongly requires a minor adjustment:

### Phase A — Understand and plan

1. Read all Project Atlas planning files.
2. Inspect the current implementation repo.
3. Decide where the app should live.
4. Create a short implementation plan in the repo before coding.
5. Identify assumptions and open risks.

### Phase B — Foundation

1. Set up app shell.
2. Add dark theme.
3. Add project workspace layout.
4. Add project library/import placeholder.
5. Add routing/navigation.

### Phase C — Schema and demo data

1. Implement AtlasBundle types.
2. Implement runtime validation.
3. Load `07_EXAMPLE_ATLAS_BUNDLE.json` as fixture.
4. Show validation report.

### Phase D — Graph engine

1. Normalize atlas data.
2. Build node/edge indexes.
3. Resolve flows/artifacts/docs/traces.
4. Implement upstream/downstream traversal.
5. Implement lens graph creation.
6. Implement impact report.

### Phase E — Canvas world map

1. Render overview graph.
2. Add zoom/pan.
3. Add minimap.
4. Add node cards.
5. Add edges.
6. Add selection/highlighting.
7. Add focus mode.
8. Add level-of-detail behavior.

### Phase F — Details and flows

1. Add detail drawer.
2. Add node detail tabs.
3. Add edge detail.
4. Add flow list.
5. Add flow timeline.
6. Add flow canvas/sequence if possible.
7. Link flow steps to supporting artifacts/docs/tools/data.

### Phase G — Lenses

Implement lens views:

1. Overview.
2. Flows.
3. Data.
4. Dependencies.
5. Agent/MCP/Hooks.
6. Docs.
7. Folders.
8. Scripts/Workflows.
9. Runtime.
10. Deployment.
11. Impact.

### Phase H — Search and review tools

1. Global search index.
2. Command palette.
3. Health/warnings view.
4. Snapshot list.
5. Diff foundation.
6. Agent instruction panel.

### Phase I — Polish

1. Improve empty states.
2. Improve visual hierarchy.
3. Improve dark theme contrast.
4. Improve canvas interactions.
5. Add first-run guide.
6. Run validation checklist.

## 7. How to reason while building

At each step, ask:

```text
What user question does this feature answer?
Which atlas data field powers it?
What should happen if the data is missing?
How does this help a non-coder understand the project?
Does it preserve the world-map canvas idea?
Can the demo bundle verify it?
What should a target-project agent update if this item changes?
```

If a technical implementation decision conflicts with user clarity, user clarity wins unless it breaks data correctness.

## 8. Required view models

Do not make UI components manually dig through raw atlas JSON.

Create view models for:

- Project summary.
- Validation report.
- Render graph.
- Node detail.
- Edge detail.
- Flow detail.
- Flow step detail.
- Document detail.
- Artifact detail.
- Lens view.
- Search result.
- Impact report.
- Runtime trace detail.
- Snapshot diff.
- Health report.

This keeps the app maintainable and makes it easier for future agents to extend.

## 9. Required user flows to verify

The implementation is not acceptable until these work with the demo bundle:

### Flow 1 — Open project map

```text
Open app
-> select demo project
-> see overview world map
-> zoom/pan
-> click Frontend App
-> detail drawer opens
```

### Flow 2 — Understand Create Project

```text
Open Flow Lens
-> select Create Project
-> see timeline steps
-> click Create project records step
-> see Project Service, reads/writes, files, evidence
```

### Flow 3 — Open AGENTS.md

```text
Search AGENTS
-> click Root AGENTS.md
-> see document summary, scope, important notes, path, related agent/tooling nodes
-> open document viewer
```

### Flow 4 — Explore MCP tools

```text
Open Agent/MCP/Hooks Lens
-> click GitHub MCP Server
-> see Read Repository Tool and Create Pull Request Tool
-> click Create Pull Request Tool
-> see tool purpose, inputs/outputs, related agent flow
```

### Flow 5 — Understand impact

```text
Click Project Service
-> Show Impact
-> see affected API, tables, email service, Create Project flow, artifacts, verification checklist
```

### Flow 6 — Search

```text
Cmd/Ctrl+K
-> search build
-> open Build App script
-> see command, risk, CI workflow, deployment usage
```

### Flow 7 — Runtime trace

```text
Open Runtime Lens
-> select Create Project observed run
-> see trace timeline/spans
-> see mapping to expected Create Project flow
```

## 10. UX quality bar

The app should be understandable without reading source code.

For every important item, the user should see:

- Plain label.
- Type.
- Purpose.
- Connections.
- Related flows.
- Related files/docs/scripts/tools.
- Evidence.
- Confidence.
- Impact.

The user should not need to know what `route.ts`, `schema.ts`, or `.cursor/mcp.json` means before understanding the system. Those paths should be visible as supporting metadata.

## 11. Canvas-specific requirements

The canvas is not optional.

It must support:

- Spatial layout.
- Pan.
- Zoom.
- Minimap.
- Node cards.
- Edge lines.
- Focus mode.
- Grouping/collapsing.
- Zoom-level detail.
- Connected-item highlighting.
- Lens-specific filtering.

The default overview should feel like this:

```text
User / Frontend area
  -> Backend / business logic area
    -> Data area
      -> External services
        -> Automation / agent tooling / deployment areas
```

The user should be able to move around the project world naturally.

## 12. Handling missing or incomplete data

Atlas data will often be incomplete.

The app must handle this gracefully:

- Missing docs: show empty state and agent fix prompt.
- Missing flows: show empty state and agent fix prompt.
- Missing runtime traces: explain that traces are optional.
- Inferred MCP tools: show low-confidence badge.
- Broken references: show validation warning/error.
- Unmapped files: show as unmapped in Folder Lens.

Do not pretend incomplete data is complete.

## 13. Evidence and confidence behavior

Every generated item should show trust level.

Examples:

```text
Confirmed by code
Confirmed by config
Confirmed by schema
Confirmed by docs
Observed runtime
Agent inferred
User confirmed
Needs review
```

Use badges consistently.

The user should always know whether something is confirmed or inferred.

## 14. AGENTS.md behavior

AGENTS.md files are first-class project objects.

The app must:

- Detect them from atlas documents/artifacts.
- Show them in Docs Lens.
- Show them in Agent/MCP/Hooks Lens if they instruct agents.
- Show scope/path.
- Show important notes.
- Open a document viewer.
- Link them to agents, folders, systems, and flows where applicable.

The user specifically wants to click an AGENTS.md item and read it inside Project Atlas.

## 15. MCP and hooks behavior

The app must make MCPs and hooks visible.

MCP drill-down must support:

```text
Agent Tooling
  -> MCP Server
    -> MCP Tools
      -> Tool inputs/outputs
      -> Used by flows/agents
      -> Config file
```

Hooks must be categorized:

- Framework hook.
- Automation hook.
- Integration webhook.
- Git hook.
- Agent hook.
- Package lifecycle hook.
- Unknown.

## 16. Scripts and workflows behavior

Scripts should not be shown as random commands only.

Each script/workflow should explain:

- What it does.
- When to use it.
- What runs it.
- What it affects.
- Risk level.
- Related systems.

## 17. Impact behavior

Impact view must be useful before the user asks an implementation agent to change something.

For a selected item, show:

- Direct dependents.
- Indirect dependents.
- Affected flows.
- Affected data.
- Affected scripts/workflows.
- Affected docs.
- Affected runtime traces.
- Verification checklist.

This is one of the most valuable non-coder features.

## 18. Agent handoff behavior

Project Atlas should help the user work with other agents.

Add surfaces where the user can copy prompts like:

```text
Update .project-atlas after changing this feature.
Make sure nodes, edges, flows, artifacts, documents, confidence, evidence, and snapshots are updated.
```

Context-specific agent prompts are useful:

- For a selected node.
- For a selected flow.
- For validation errors.
- For low-confidence warnings.
- For missing MCP/docs/runtime data.

## 19. Verification before completing work

Before finishing a build session, run through this checklist:

```text
Can the demo atlas import?
Can validation detect errors?
Can the overview world map render?
Can the user zoom and pan?
Can the user click nodes and edges?
Can detail drawer explain selected items?
Can the Create Project flow be understood?
Can AGENTS.md be opened in-app?
Can MCP server/tools be explored?
Can scripts/workflows be understood?
Can data reads/writes be seen?
Can impact analysis show affected items?
Can search find AGENTS, MCP, build, and create project?
Are confidence/evidence badges visible?
Are missing/inferred items honestly marked?
Is the UI dark, calm, and visually pleasing?
```

If any answer is no, continue implementation or document what remains incomplete.

## 20. Commit/session summary format

After each meaningful implementation session, leave a summary like:

```text
Implemented:
- What was built.

User capability unlocked:
- What the user can now do.

Verified with demo bundle:
- What was tested.

Known gaps:
- What remains incomplete.

Next recommended step:
- What the next agent should do.
```

This keeps future agent sessions coherent.

## 21. Final reminder to builder-agent

Project Atlas exists because the user wants to understand complex projects visually without needing to know code.

Do not build for developers only.

Build for someone who wants to see:

```text
the system,
the flows,
the supporting files,
the docs,
the scripts,
the hooks,
the MCP tools,
the data,
the runtime behavior,
and the impact of changes
```

all inside one beautiful, dark, zoomable project world map.
```
