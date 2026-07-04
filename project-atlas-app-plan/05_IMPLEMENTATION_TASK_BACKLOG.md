# 05 — Implementation Task Backlog

This document gives the builder-agent a structured implementation backlog for Project Atlas.

It is intentionally not application code. It is a build framework: what must be created, why it exists, how the pieces fit together, and how the builder-agent should verify its own work while implementing.

## 1. Builder-agent operating principles

The implementation agent must build the app in a way that serves the primary user: a non-coder or vibe coder who wants to visually understand a complex project.

The implementation agent must optimize for:

1. **Visual clarity over technical completeness.**
2. **Progressive disclosure over huge graphs.**
3. **Plain-language meaning over raw file names.**
4. **Evidence and confidence over fake certainty.**
5. **Reusable data contracts over one-off hardcoded diagrams.**
6. **Canvas-first exploration over static documentation pages.**
7. **Agent-maintainable project maps over manual diagram drawing.**

The builder-agent should continuously ask itself:

```text
Can a non-coder understand what this object does?
Can the user zoom out and still understand the project?
Can the user zoom in and discover supporting files, docs, scripts, hooks, MCP tools, and flows?
Can another agent update this data without changing the app code?
Is this claim backed by evidence or clearly marked as inferred?
```

## 2. Definition of the final product

Project Atlas is a standalone visual application that imports an `atlas.bundle.json` file generated from a target project and turns it into an interactive, dark, canvas-based, zoomable world map of the project.

The app must allow the user to:

- Add/import a project atlas bundle.
- See a beautiful high-level project map.
- Switch viewing lenses.
- Click any system/item and open details.
- See feature flows step by step.
- See which files, docs, scripts, MCP tools, hooks, and configs support each step.
- Open mapped Markdown documents inside the app.
- Search the entire atlas.
- Understand what may break if something changes.
- Compare snapshots over time.
- Identify low-confidence or missing areas.

## 3. Build philosophy

The app should be built from the inside out:

```text
Reusable data contract
  -> Validation
    -> Graph engine
      -> Lens view models
        -> Canvas and panels
          -> Polished UX
```

Do not start by making a pretty static diagram. The beauty of the app must come from a strong graph model and consistent rendering rules.

## 4. Implementation phases overview

| Phase | Name | Goal |
|---|---|---|
| 0 | Repo/app foundation | Create app skeleton and design foundation. |
| 1 | Schema and validation | Make atlas data safe and predictable. |
| 2 | Demo bundle | Provide real fixture data for development. |
| 3 | Graph engine | Normalize, query, traverse, filter, and analyze atlas data. |
| 4 | Canvas-first UI shell | Build the main world-map experience. |
| 5 | Detail system | Make every clicked item explain itself. |
| 6 | Lens system | Add overview, flows, data, dependencies, docs, folders, scripts, agents/MCP/hooks, runtime, deployment, impact. |
| 7 | Import and project library | Let user add/manage multiple project atlases. |
| 8 | Search and command palette | Make everything findable. |
| 9 | Snapshot and diff | Let user review changes over time. |
| 10 | Agent kit integration | Provide templates/prompts/validation for target-project agents. |
| 11 | Polish and usability | Make it feel premium, calm, and easy to use. |

## 5. Phase 0 — Foundation

### 5.1 Goal

Create the basic application foundation without implementing the full product yet.

### 5.2 Required decisions

The builder-agent must choose the actual framework and packages, but should preserve the architecture in `02_TECHNICAL_IMPLEMENTATION_PLAN.md`.

Recommended direction:

- TypeScript.
- React-based app.
- Canvas/graph rendering library.
- Dark design system.
- Schema validation library.
- Local-first storage option.

### 5.3 Deliverables

- App shell.
- Dark theme tokens.
- Route/page structure.
- Layout system with top bar, left sidebar, main canvas area, right drawer.
- Empty project library screen.
- Demo mode placeholder.
- Build/test/lint scripts.

### 5.4 Self-verification

The builder-agent should verify:

- The app opens without requiring a target project.
- The dark theme is applied globally.
- The layout already resembles the final canvas-first product.
- The app has room for project switcher, lens switcher, search, canvas, minimap, and detail drawer.

### 5.5 Acceptance criteria

The user can open the app and see a polished dark workspace, even before real atlas rendering exists.

## 6. Phase 1 — Schema and validation

### 6.1 Goal

Make `AtlasBundle` a first-class contract.

The app should not trust imported JSON blindly.

### 6.2 Deliverables

- Runtime schema validator for `AtlasBundle`.
- Type definitions aligned with `04_ATLAS_DATA_CONTRACT.md`.
- Validation report model.
- Import validation screen.
- Error and warning categories.
- Schema migration entry point.

### 6.3 Validation must detect

- Missing required top-level fields.
- Duplicate node IDs.
- Duplicate edge IDs.
- Edges pointing to missing nodes.
- Flow steps pointing to missing nodes/artifacts/docs.
- Documents not mapped to anything.
- Artifacts not mapped to anything.
- Invalid confidence labels.
- Secret values accidentally included.
- Unsupported schema versions.

### 6.4 Self-verification

The builder-agent should intentionally load both good and broken bundles and confirm that the validation report is understandable.

The validation screen must explain issues in plain language, not only technical terms.

Example:

```text
Broken edge: Create Project Form calls Missing API Node.
Reason: The edge target ID does not exist in this atlas bundle.
Suggested fix: Add the missing API node or remove/update the edge.
```

### 6.5 Acceptance criteria

A builder can import `07_EXAMPLE_ATLAS_BUNDLE.json`, pass validation, and see a clean validation summary.

## 7. Phase 2 — Demo bundle

### 7.1 Goal

Create a realistic demo atlas that exercises the product.

The demo bundle is not optional. It is the anchor for implementation and testing.

### 7.2 Deliverables

- `07_EXAMPLE_ATLAS_BUNDLE.json` used as the initial fixture.
- Demo project loaded by default if no user project exists.
- Demo includes systems, flows, docs, scripts, MCPs, hooks, files, data, deployment, and runtime traces.

### 7.3 Self-verification

The builder-agent should check:

- The demo can show every major lens.
- The demo can support canvas zoom/drill-down.
- The demo has enough data for the drawer tabs to feel real.
- The demo does not require real source code.

### 7.4 Acceptance criteria

The app can be developed and reviewed using only the demo bundle before any real target project is imported.

## 8. Phase 3 — Graph engine

### 8.1 Goal

Convert imported atlas data into a useful graph model.

The graph engine is the brain of the product.

### 8.2 Deliverables

- Normalized graph model.
- Node index.
- Edge index.
- Incoming/outgoing relationship indexes.
- Flow usage index.
- Artifact usage index.
- Document usage index.
- Search index source model.
- Lens filtering engine.
- Impact analysis model.
- Path finding model.
- Graph health summary.

### 8.3 Required graph queries

The UI must be able to ask:

```text
Give me this node.
Give me this node's upstream dependencies.
Give me this node's downstream dependents.
Give me flows using this node.
Give me artifacts supporting this node.
Give me documents explaining this node.
Give me MCP tools used by this node or flow.
Give me hooks connected to this node or flow.
Give me scripts connected to this node or flow.
Give me a renderable graph for this lens.
Give me a focus graph around this node.
Give me impact report for this node.
Give me path from node A to node B.
```

### 8.4 Self-verification

The builder-agent should write tests against the demo bundle for:

- Node lookup.
- Edge traversal.
- Lens filtering.
- Flow step resolution.
- Impact analysis.
- Missing reference handling.

### 8.5 Acceptance criteria

The UI can be built entirely from graph-engine view models, not by manually searching raw JSON inside components.

## 9. Phase 4 — Canvas-first UI shell

### 9.1 Goal

Create the main experience: a 2D world map of the project.

The canvas is the heart of Project Atlas.

### 9.2 Required canvas behavior

The canvas must support:

- Pan.
- Zoom.
- Smooth movement.
- Minimap.
- Click node.
- Click edge.
- Hover preview.
- Group/cluster display.
- Collapse/expand group.
- Focus mode.
- Fit-to-view.
- Reset view.
- Highlight connected nodes.
- Dim unrelated nodes.
- Confidence/evidence badges.
- Visual level-of-detail based on zoom.

### 9.3 World-map metaphor

Think of the project as a world map:

```text
Zoomed out:
  Continents = major systems/domains.

Medium zoom:
  Countries = features, subsystems, integrations.

Close zoom:
  Cities = flows, services, pages, APIs, data stores.

Deep zoom:
  Buildings = files, scripts, hooks, MCP tools, docs, configs.
```

The user should feel like they are exploring a map, not reading a technical report.

### 9.4 Canvas information density rules

- At low zoom, hide small details.
- At medium zoom, show names and important badges.
- At high zoom, show summaries and key relationships.
- Never render thousands of tiny labels at once.
- Prefer grouped islands over hairball graphs.
- Use edge bundling/grouping if needed.
- Use filters to reduce noise.

### 9.5 Acceptance criteria

The user can open Overview Lens and visually understand the demo project without opening any side panel.

## 10. Phase 5 — Detail system

### 10.1 Goal

Every item on the canvas must be explainable.

Clicking an item should answer:

```text
What is this?
Why does it exist?
Where does it fit?
What does it connect to?
What files/docs/scripts/tools support it?
What flows use it?
How confident are we?
What evidence supports it?
```

### 10.2 Deliverables

- Right-side detail drawer.
- Node detail tabs.
- Edge detail drawer.
- Flow step detail drawer.
- Markdown document viewer.
- Evidence panel.
- Confidence panel.
- Related items list.

### 10.3 Drawer tabs

Use these tabs when relevant:

1. Summary.
2. Connections.
3. Flows.
4. Artifacts.
5. Docs.
6. Data.
7. Scripts.
8. MCP/Tools.
9. Hooks.
10. Runtime.
11. Impact.
12. Notes.

### 10.4 Acceptance criteria

The user can click an `AGENTS.md` document node and read its mapped summary and content/reference in the app. The user can click an MCP server and drill to its tools. The user can click a flow step and see supporting files, docs, data reads/writes, and errors.

## 11. Phase 6 — Lens system

### 11.1 Goal

Create multiple ways of viewing the same graph.

A lens is not a separate data model. It is a filtered, styled, purpose-driven view of the same atlas graph.

### 11.2 Required initial lenses

- Overview Lens.
- Flow Lens.
- Data Lens.
- Dependency Lens.
- Agent/MCP/Hooks Lens.
- Docs Lens.
- Folder Structure Lens.
- Scripts and Workflows Lens.
- Runtime Trace Lens.
- Deployment Lens.
- Change Impact Lens.

### 11.3 Lens acceptance rule

Every lens must answer a specific user question.

Examples:

```text
Overview: What exists and how does the project fit together?
Flows: What happens step by step?
Data: Where does data live and move?
Dependencies: What depends on what?
Agent/MCP/Hooks: What automation and tools exist?
Docs: What documents/instructions explain this project?
Folders: Where is everything located?
Scripts: What commands/workflows can run?
Runtime: What actually happened during a real run?
Deployment: Where does everything run?
Impact: What might break if this changes?
```

### 11.4 Acceptance criteria

The user can switch lenses without losing project context. Selected item, breadcrumbs, and relevant filters should carry over where possible.

## 12. Phase 7 — Import and project library

### 12.1 Goal

Let the user use Project Atlas across many projects.

### 12.2 Deliverables

- Project library screen.
- Import bundle screen.
- Import validation screen.
- Project metadata cards.
- Last updated timestamp.
- Snapshot count.
- Warnings count.
- Delete/archive project.
- Re-import updated bundle.

### 12.3 Acceptance criteria

The user can import at least one atlas bundle, open it, close the app, return later, and still see the project.

## 13. Phase 8 — Search and command palette

### 13.1 Goal

Make the atlas navigable even when the graph becomes large.

### 13.2 Required search targets

Search across:

- Node labels.
- Node summaries.
- Flow labels and steps.
- File paths.
- Folder paths.
- Script commands.
- MCP servers/tools.
- Hook names/triggers.
- Document titles/summaries.
- AGENTS.md content summaries.
- Evidence sources.
- Tags and areas.

### 13.3 Required command actions

- Open item.
- Focus on item.
- Show item in current lens.
- Show impact.
- Open related flow.
- Open related docs.
- Switch lens.
- Fit canvas.
- Show low-confidence items.
- Show recently changed items.

### 13.4 Acceptance criteria

The user can press `Cmd/Ctrl + K`, type `stripe`, `AGENTS`, `build`, `create project`, or `mcp`, and quickly jump to the relevant item.

## 14. Phase 9 — Snapshot and diff

### 14.1 Goal

Help the user review how the project map changed after agents modify the target project.

### 14.2 Deliverables

- Snapshot storage.
- Snapshot list.
- Compare two snapshots.
- Added/removed/changed nodes.
- Added/removed/changed edges.
- Flow step changes.
- Confidence changes.
- Evidence changes.
- Visual diff overlay.

### 14.3 Acceptance criteria

After importing a newer bundle, the user can see what changed in the atlas since the previous import.

## 15. Phase 10 — Agent kit integration

### 15.1 Goal

Make it easy for implementation agents to set up and maintain atlas files in target projects.

### 15.2 Deliverables

- Copyable setup prompt.
- Copyable update prompt.
- Copyable quality review prompt.
- `.project-atlas` file templates.
- Validation checklist.
- Bundle generation guidance.
- In-app explanation of how to ask an agent to update a project atlas.

### 15.3 Acceptance criteria

From inside Project Atlas, the user can copy instructions and give them to a coding agent working in a target repo. That agent can produce an atlas bundle that Project Atlas can import.

## 16. Phase 11 — Polish and usability

### 16.1 Goal

Make the app feel like a premium tool that a non-coder wants to use.

### 16.2 Required polish

- Smooth canvas transitions.
- Clear empty states.
- Beautiful node cards.
- Readable badges.
- Fast search.
- Helpful loading states.
- Friendly validation messages.
- Keyboard shortcuts.
- Stable layouts.
- Good contrast.
- No overwhelming default views.

### 16.3 Acceptance criteria

A first-time user can import the demo bundle and understand the project within five minutes.

## 17. Cross-phase design constraints

The builder-agent must preserve these constraints throughout implementation:

- The app is canvas-first.
- The graph is the central model.
- Details come from structured atlas data.
- Markdown/docs are first-class objects.
- AGENTS.md files are visible and scoped.
- MCP servers/tools are visible and drillable.
- Hooks are categorized.
- Scripts/workflows are visible and explainable.
- Confidence/evidence is always shown for generated knowledge.
- Project maps are reusable across target projects.
- No hard dependency on one framework or language.

## 18. Non-goals for the first version

Do not prioritize these before the core product works:

- Fully automatic perfect code understanding.
- Real-time source-code editing.
- Full IDE replacement.
- Automatic execution of arbitrary project scripts.
- Uploading private source code to an external service.
- Building custom scanner support for every framework before the atlas bundle contract works.
- Making a giant raw file dependency graph the main interface.

## 19. Builder-agent reasoning loop

At the end of each implementation phase, the builder-agent should write a short internal project note or commit summary answering:

```text
What user capability did this phase unlock?
What part of the atlas contract does it rely on?
What assumptions did I make?
What is still mocked or incomplete?
How can this be verified with the demo bundle?
What should the next agent continue from?
```

This makes the implementation chain understandable across agent sessions.

## 20. Final build readiness checklist

Before calling the app usable, verify:

- Demo bundle imports.
- Validation report works.
- Overview canvas works.
- Zoom/pan/minimap/focus works.
- Node drawer works.
- Flow lens works.
- Docs viewer works.
- AGENTS.md mapping works.
- Folder lens works.
- Scripts lens works.
- Agent/MCP/hooks lens works.
- Data lens works.
- Dependency lens works.
- Impact lens works.
- Search works.
- Snapshot diff works.
- Empty states work.
- Low-confidence items are visible.
- The UI remains readable and beautiful in dark mode.
