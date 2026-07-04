# Project Atlas App — Implementation Planning Folder

This folder contains the implementation plan for a reusable visual project-understanding application.

The app is designed as a separate system that can inspect, import, or receive structured project information from another app/repo and turn it into a dark, visually pleasing, zoomable atlas of systems, flows, files, scripts, hooks, MCPs, agents, data models, runtime traces, documents, and dependencies.

## Intended product name

**Project Atlas**

Alternative names that still fit the plan:

- System Atlas
- Flow Atlas
- Repo Atlas
- Architecture Lens
- Project X-Ray

## Main goal

Create a reusable app that lets a non-coder understand complex projects through visual layers instead of source code.

The app must support:

- High-level system overview.
- Drill-down from project to system to feature to pipeline step to supporting code/files/tools.
- Multiple viewing lenses: overview, flows, data, dependencies, runtime, deployment, docs, agents, MCPs, hooks, scripts, and folder structure.
- A canvas-first 2D world-map experience where zooming reveals more detail.
- Agent-maintained project maps, so the target project can stay updated without the user manually drawing diagrams.
- Evidence and confidence markers, so the app shows what was scanned, agent-authored, runtime-observed, or inferred.
- A reusable data contract that works across many kinds of projects, not only one framework.

## Files in this planning folder

Read these in order:

1. [`01_PRODUCT_AND_UX_BLUEPRINT.md`](./01_PRODUCT_AND_UX_BLUEPRINT.md)  
   Defines the product behavior, visual experience, app screens, navigation model, dark UI direction, and user workflows.

2. [`02_TECHNICAL_IMPLEMENTATION_PLAN.md`](./02_TECHNICAL_IMPLEMENTATION_PLAN.md)  
   Defines the architecture, modules, packages, graph model, import pipeline, rendering approach, state management, storage, APIs, and implementation phases.

3. [`03_AGENT_PROJECT_SETUP_GUIDE.md`](./03_AGENT_PROJECT_SETUP_GUIDE.md)  
   Tells coding agents exactly how to prepare a target project for Project Atlas, what files to create, what to scan, what to explain, and how to keep it updated.

4. [`04_ATLAS_DATA_CONTRACT.md`](./04_ATLAS_DATA_CONTRACT.md)  
   Defines the first version of the reusable `AtlasBundle` schema, node types, edge types, flow schema, artifact schema, evidence model, confidence model, and examples.

5. [`05_IMPLEMENTATION_TASK_BACKLOG.md`](./05_IMPLEMENTATION_TASK_BACKLOG.md)  
   Gives the builder-agent a phase-by-phase backlog with goals, deliverables, acceptance criteria, and self-verification questions.

6. [`06_COMPONENT_SPECIFICATION.md`](./06_COMPONENT_SPECIFICATION.md)  
   Defines the major UI/product components: canvas, nodes, edges, drawers, lenses, docs viewer, flow viewer, search, import, runtime, diff, and agent helper panels.

7. [`07_EXAMPLE_ATLAS_BUNDLE.json`](./07_EXAMPLE_ATLAS_BUNDLE.json)  
   Provides a realistic demo atlas bundle that the builder-agent can import, render, validate, and use as the primary development fixture.

8. [`08_VALIDATION_TEST_CASES.md`](./08_VALIDATION_TEST_CASES.md)  
   Defines schema, graph, canvas, flow, search, runtime, diff, and usability validation cases for the implementation agent.

9. [`09_UI_SCREEN_BY_SCREEN_SPEC.md`](./09_UI_SCREEN_BY_SCREEN_SPEC.md)  
   Specifies every major screen and how it should behave visually and functionally.

10. [`10_AGENT_BUILD_PROMPT.md`](./10_AGENT_BUILD_PROMPT.md)  
    Provides the master prompt that can be given to the implementation agent building the app.

11. [`11_CANVAS_WORLD_MAP_DESIGN_FRAMEWORK.md`](./11_CANVAS_WORLD_MAP_DESIGN_FRAMEWORK.md)  
    Captures the core canvas/world-map metaphor, zoom behavior, spatial layout, detail layers, flow routes, document landmarks, MCP/tool islands, runtime routes, and impact blast-radius model.

12. [`12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md`](./12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md)  
    Defines the recommended Rust/Tauri desktop stack plus the frontend visual stack needed for a world-class experience: TypeScript, React, Vite, React Flow, ELK.js, D3, Tailwind, Radix/shadcn-style primitives, Motion, Zustand, SQLite, Markdown rendering, search, icons, and testing.

13. [`13_ONE_RUN_IMPLEMENTATION_AGENT_CONTEXT_AND_GOAL.md`](./13_ONE_RUN_IMPLEMENTATION_AGENT_CONTEXT_AND_GOAL.md)  
    Provides the updated one-run implementation-agent context prompt and short goal, including Rust/Tauri, the visual stack, Code Philosophy, full sequential implementation, and full-scale testing expectations.

## Core design decision

Project Atlas should not depend on perfect automatic code parsing.

Instead, it should use a hybrid model:

1. **Agent-authored project atlas files** in the target repo.
2. **Optional scanner/CLI output** for file trees, package scripts, dependencies, routes, docs, workflows, MCP configs, hooks, and schemas.
3. **Optional runtime traces** for what actually happened during a real flow.
4. **Manual notes and overrides** for business meaning, plain-language explanations, and project-specific terminology.

This makes the app flexible enough to work across almost any project type.

## Recommended implementation stack

The preferred app direction is:

```text
Desktop shell: Tauri + Rust
Frontend: TypeScript + React + Vite
Canvas: @xyflow/react / React Flow + ELK.js + D3 utilities
Design system: Tailwind CSS + Radix UI/shadcn-style primitives + Motion
State/storage: Zustand + SQLite/IndexedDB
Docs/search: react-markdown + command palette + fuzzy search
Testing: Vitest + React Testing Library + Playwright
```

The full stack guidance is defined in [`12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md`](./12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md).

## Recommended build handoff prompt

For the implementation agent that should build everything in one continuous run, use:

[`13_ONE_RUN_IMPLEMENTATION_AGENT_CONTEXT_AND_GOAL.md`](./13_ONE_RUN_IMPLEMENTATION_AGENT_CONTEXT_AND_GOAL.md)

That file includes the updated context prompt, goal, required stack, Code Philosophy requirement, phase sequence, and full-scale verification standard.

## Recommended first MVP

Build the app so it can import a single `atlas.bundle.json` file and render:

- Project overview.
- Zoomable system graph.
- Feature flow timeline/canvas.
- Node detail drawer.
- Docs/readme/AGENTS.md viewer.
- Folder tree lens.
- Scripts/hooks/MCP lens.
- Search and command palette.
- Confidence/evidence badges.

After that, add the agent/CLI workflow that generates and updates the bundle inside target projects.

## Non-negotiables

- The UI must be dark, calm, readable, and visually pleasant.
- The app must explain systems without requiring the user to read code first.
- The main experience must be canvas-first, like a 2D world map of the project.
- Zooming should reveal more detail instead of only scaling the same graph.
- The implementation should use the visual stack guidance unless there is a strong technical reason to deviate.
- Every important visible item should have a purpose, owner/source, evidence, confidence, and links to related items.
- The graph must support progressive disclosure. Never force the user to look at the whole codebase as one giant graph.
- Agents must be able to create and update the project atlas through a clear file contract.
