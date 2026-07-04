# Project Atlas App — Implementation Planning Folder

This folder contains the implementation plan for **Project Atlas**, a reusable visual project-understanding application.

Project Atlas is intended to be a separate desktop app that imports structured project information from another repo, usually from `.project-atlas/atlas.bundle.json`, and renders it as a dark, canvas-first, zoomable 2D project world map.

## Main goal

Create a reusable app that helps a non-coder or vibe coder understand complex projects through visual layers instead of reading source code first.

The app should support:

- High-level system overview.
- Drill-down from project to system to feature to flow step to supporting files/tools/docs.
- Multiple lenses: overview, flows, data, dependencies, runtime, deployment, docs, agents, MCPs, hooks, scripts, and folder structure.
- Canvas-first world-map exploration.
- Agent-maintained project atlas files.
- Evidence and confidence markers.
- A reusable data contract that works across many project types.

## Files in this planning folder

Read these in order:

1. [`01_PRODUCT_AND_UX_BLUEPRINT.md`](./01_PRODUCT_AND_UX_BLUEPRINT.md)
2. [`02_TECHNICAL_IMPLEMENTATION_PLAN.md`](./02_TECHNICAL_IMPLEMENTATION_PLAN.md)
3. [`03_AGENT_PROJECT_SETUP_GUIDE.md`](./03_AGENT_PROJECT_SETUP_GUIDE.md)
4. [`04_ATLAS_DATA_CONTRACT.md`](./04_ATLAS_DATA_CONTRACT.md)
5. [`05_IMPLEMENTATION_TASK_BACKLOG.md`](./05_IMPLEMENTATION_TASK_BACKLOG.md)
6. [`06_COMPONENT_SPECIFICATION.md`](./06_COMPONENT_SPECIFICATION.md)
7. [`07_EXAMPLE_ATLAS_BUNDLE.json`](./07_EXAMPLE_ATLAS_BUNDLE.json)
8. [`08_VALIDATION_TEST_CASES.md`](./08_VALIDATION_TEST_CASES.md)
9. [`09_UI_SCREEN_BY_SCREEN_SPEC.md`](./09_UI_SCREEN_BY_SCREEN_SPEC.md)
10. [`10_AGENT_BUILD_PROMPT.md`](./10_AGENT_BUILD_PROMPT.md)
11. [`11_CANVAS_WORLD_MAP_DESIGN_FRAMEWORK.md`](./11_CANVAS_WORLD_MAP_DESIGN_FRAMEWORK.md)
12. [`12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md`](./12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md)

## Core design decision

Project Atlas should not depend on perfect automatic code parsing. It should use a hybrid model:

1. Agent-authored project atlas files in the target repo.
2. Optional scanner/CLI output for file trees, package scripts, routes, docs, workflows, MCP configs, hooks, and schemas.
3. Optional runtime traces for what actually happened during a real flow.
4. Manual notes and overrides for business meaning and project-specific terminology.

## Recommended implementation stack

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

## Non-negotiables

- The UI must be dark, calm, readable, and visually pleasing.
- The main experience must be canvas-first, like a 2D world map of the project.
- Zooming should reveal more detail instead of only scaling the same graph.
- Every important item should have a purpose, evidence, confidence, and related links.
- The graph must support progressive disclosure.
- Agents must be able to create and update the project atlas through a clear file contract.
