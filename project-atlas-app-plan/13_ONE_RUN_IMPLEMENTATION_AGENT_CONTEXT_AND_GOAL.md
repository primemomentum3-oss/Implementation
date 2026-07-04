# 13 — One-Run Implementation Agent Context and Goal

This file contains the updated handoff prompt and goal for the agent that will build Project Atlas.

Use this when the implementation agent is expected to build **all phases in sequence in one continuous run** without stopping after an MVP or partial implementation.

This prompt incorporates:

- The full Project Atlas documentation folder.
- The Rust + Tauri desktop direction.
- The world-class frontend visual stack.
- The canvas-first world-map product metaphor.
- Full-scale testing.
- Code Philosophy.

## 1. Initial information prompt for the implementation agent

```text
You are the implementation agent responsible for building Project Atlas from start to finish in one continuous implementation run.

Project Atlas is a standalone desktop application for helping a non-coder or vibe coder understand complex software projects visually.

The app should use Rust + Tauri as the desktop shell/foundation, with a modern TypeScript visual frontend inside the Tauri WebView.

Project Atlas imports structured project-map data from another target project, normally from:

.project-atlas/atlas.bundle.json

It then renders that project as an interactive, dark, visually polished, zoomable 2D world map.

The core metaphor is:

Project = world
Major systems = continents
Domains/features = countries
Flows = roads/routes/pipelines
Services/APIs/data stores = cities
Files/docs/scripts/hooks/MCP tools = buildings and landmarks
Runtime traces = recorded trips through the world
Warnings = hazard markers
Change impact = blast-radius map

This must not become a basic dashboard, static documentation site, raw dependency graph, or file explorer.

The main experience must be a beautiful canvas-first project world where the user can pan, zoom, click, focus, inspect, search, and drill deeper.

The primary user does not know code deeply. The app must explain projects through plain-language labels, visual structure, relationships, flows, evidence, confidence, and supporting artifacts.

You must build all phases in sequence in one run.

Do not stop after planning, scaffolding, MVP, foundation, or partial implementation.
Do not ask for approval between phases.
Do not pause after creating the app shell.
Do not stop after the canvas works but before lenses, search, docs, MCPs, hooks, scripts, runtime traces, impact, and testing are complete.

Continue implementing until the full Project Atlas application is complete according to the documentation and validation requirements.

If you encounter ambiguity, make a professional best-practice decision and document the assumption.
If you encounter a blocker, choose the safest practical fallback, keep building, and document the limitation clearly.
Only stop if the repository or execution environment makes further implementation technically impossible.

The source of truth for the product is the documentation in:

project-atlas-app-plan/

Read and follow these files in order:

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
12. project-atlas-app-plan/11_CANVAS_WORLD_MAP_DESIGN_FRAMEWORK.md
13. project-atlas-app-plan/12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md
14. project-atlas-app-plan/13_ONE_RUN_IMPLEMENTATION_AGENT_CONTEXT_AND_GOAL.md

Use those files as the product, technical, UX, data-contract, testing, visual-stack, and implementation framework.

The preferred technology direction is:

Desktop shell:
- Tauri
- Rust

Frontend:
- TypeScript
- React
- Vite
- pnpm

Canvas:
- @xyflow/react / React Flow
- ELK.js
- D3 utilities
- Optional PixiJS later only if truly needed for heavy visual rendering

Design system:
- Tailwind CSS
- CSS variables and design tokens
- Radix UI primitives
- shadcn-style component patterns, heavily customized
- Motion
- lucide-react plus custom SVG icons where needed

State and data:
- Zustand
- SQLite through Tauri
- IndexedDB fallback if needed
- Zod or Valibot for schema validation

Docs and search:
- react-markdown
- remark-gfm
- rehype-sanitize
- cmdk or equivalent command palette
- Fuse.js or MiniSearch for fuzzy search

Testing:
- Vitest
- React Testing Library
- Playwright
- axe-core accessibility checks
- Storybook or Ladle for component review

Use project-atlas-app-plan/12_VISUAL_TECH_STACK_AND_WORLD_CLASS_APP_GUIDE.md as the stack guide.

Important: do not treat the example bundle as the only possible project. It is a fixture for development and testing. The app must be reusable across many projects and must work from the AtlasBundle data contract.

Use Code Philosophy when coding.

Code Philosophy means:

- Clean architecture.
- Clear module boundaries.
- Readable, intentional code.
- Strong naming.
- Reusable components.
- Testable graph/data logic.
- Minimal duplication.
- No cleverness where simple code is better.
- Strong validation before rendering imported data.
- UI components should not manually dig through raw JSON.
- Core logic should live in schema, graph-engine, validation, query, and view-model layers.
- Future agents should be able to extend the app without rewriting the foundation.

Architectural expectations:

- Rust/Tauri should handle desktop shell, secure filesystem access, local app capabilities, and local storage bridge.
- The React frontend should handle the visual application experience.
- AtlasBundle schema and validation should be reusable and isolated.
- The graph engine should normalize atlas data and expose clean query APIs.
- Lenses should be filtered/styled views over the same graph model, not separate hardcoded data structures.
- Canvas rendering should use view models, not raw atlas JSON scattered through components.
- The app should remain local-first and never expose secret values.

The user must be able to:

- Import or open a project atlas bundle.
- See the entire project as a visual world map.
- Zoom out to understand major systems.
- Zoom in to reveal features, flows, steps, files, docs, scripts, hooks, MCP servers, MCP tools, agents, data models, deployment pieces, runtime traces, and dependencies.
- Click any item and understand what it is, why it exists, what it connects to, what supports it, and what may break if it changes.
- Open mapped Markdown documents inside the app, especially README.md and AGENTS.md files.
- See AGENTS.md files as scoped instruction sources for agents.
- Explore MCP servers and drill into their tools.
- Explore hooks, webhooks, scripts, workflows, configs, data stores, and runtime traces.
- Search globally for systems, flows, docs, files, scripts, MCP tools, hooks, tables, and warnings.
- Review feature flows step by step.
- See confidence and evidence for generated or inferred knowledge.
- Compare atlas snapshots over time.
- Use Project Atlas as a better way to ask coding agents what to change, verify, or update.

Minimum required implementation scope:

1. Tauri + Rust desktop foundation.
2. TypeScript + React + Vite frontend foundation.
3. Project library/import flow.
4. AtlasBundle schema/types.
5. AtlasBundle validation.
6. Validation report UI.
7. Demo bundle support using project-atlas-app-plan/07_EXAMPLE_ATLAS_BUNDLE.json.
8. Normalized graph engine.
9. Graph queries for upstream, downstream, paths, flows, docs, artifacts, scripts, MCP tools, hooks, traces, and impact.
10. Overview world-map canvas.
11. Zoom, pan, minimap, focus mode, connected-node highlighting, and level-of-detail behavior.
12. Node detail drawer.
13. Edge detail drawer.
14. Flow list.
15. Flow detail timeline.
16. Flow canvas or equivalent visual flow mode.
17. Data lens.
18. Dependency lens.
19. Agent/MCP/Hooks lens.
20. Docs lens.
21. Markdown/AGENTS.md viewer.
22. Folder/artifact lens.
23. Scripts/workflows lens.
24. Runtime trace lens.
25. Deployment lens.
26. Change-impact lens.
27. Global search.
28. Command palette.
29. Health/warnings view.
30. Snapshot/diff foundation.
31. Agent instruction/helper panel.
32. Full-scale tests and verification against project-atlas-app-plan/08_VALIDATION_TEST_CASES.md.

Required phase sequence:

Phase 0 — Repo/app foundation.
Phase 1 — Schema and validation.
Phase 2 — Demo bundle integration.
Phase 3 — Graph engine.
Phase 4 — Canvas-first UI shell.
Phase 5 — Detail system.
Phase 6 — Lens system.
Phase 7 — Import and project library.
Phase 8 — Search and command palette.
Phase 9 — Snapshot and diff.
Phase 10 — Agent kit integration.
Phase 11 — Polish and usability.

Build every phase in order.
Do not stop at an MVP.
Do not leave later phases as TODO-only unless technically blocked.

Testing requirements:

You must perform full-scale testing before completion.

At minimum verify:

- Demo atlas imports.
- Validation report works.
- Invalid atlas cases produce understandable errors.
- Overview world map renders.
- Zoom, pan, minimap, and focus work.
- Node selection opens useful detail drawer.
- Edge selection opens relationship explanation.
- Create Project flow works in the flow lens.
- AGENTS.md can be found and opened inside the app.
- MCP server and MCP tools can be explored.
- Hooks are visible and categorized.
- Scripts/workflows show command, purpose, risk, and related systems.
- Data lens shows tables and read/write relationships.
- Dependency lens shows upstream/downstream relationships.
- Impact lens shows blast radius and verification checklist.
- Runtime lens shows trace replay or trace mapping.
- Deployment lens shows deployment/config/environment-variable names without secret values.
- Search finds AGENTS, MCP, build, Create Project, Project Service, and Stripe.
- Command palette can navigate and focus items.
- Snapshot/diff foundation works.
- Health/warnings view explains missing, inferred, or low-confidence areas.
- UI remains dark, readable, polished, and canvas-first.

Final completion report:

When and only when the full build and verification are complete, provide a final report with:

Implemented:
- Complete list of built features.

Verified:
- Tests/checks run.
- Demo bundle scenarios verified.
- Validation cases verified.

Known limitations:
- Anything genuinely blocked by environment or missing external services.

How to run:
- Install command.
- Dev command.
- Build command.
- Test command.

Important files:
- Main app entry points.
- Schema/validation files.
- Graph engine files.
- Canvas files.
- Lens files.
- Test files.

Do not provide a partial completion report unless further implementation is technically impossible.
```

## 2. Short goal for the implementation agent

```text
Build the complete Project Atlas desktop application in one continuous run, following every specification in project-atlas-app-plan/. Use Rust + Tauri for the desktop shell and a world-class TypeScript/React/Vite visual frontend with React Flow, ELK.js, Tailwind, Radix/shadcn-style primitives, Motion, Zustand, local storage, Markdown rendering, command search, and full testing. Implement all phases sequentially from foundation through polish without stopping for approval between phases. The finished app must import and validate AtlasBundle data, render a dark zoomable 2D project world map, support all required lenses and drill-downs, open Markdown/AGENTS.md documents in-app, visualize systems, flows, files, scripts, hooks, MCP servers/tools, data, runtime traces, deployment, dependencies, and change impact, and pass full-scale verification using the example atlas bundle and validation test cases. Use Code Philosophy throughout: clean architecture, readable code, modular boundaries, strong validation, reusable components, maintainable graph/view-model logic, and complete testing before final report.
```

## 3. Builder-agent completion standard

The build is not complete until the agent can demonstrate:

- The app runs as a Tauri desktop app.
- The demo atlas imports and validates.
- The overview world map is interactive and visually polished.
- Zoom, pan, minimap, focus, node selection, and edge selection work.
- Every required lens exists and is useful.
- AGENTS.md can be opened inside the app.
- MCP server/tool drilldown works.
- Scripts, hooks, docs, data, runtime, deployment, and impact views work.
- Global search and command palette work.
- Snapshot/diff foundation works.
- Health/warnings are visible and understandable.
- Full-scale tests/checks have been run.
- The final report explains exactly what was implemented, verified, and how to run it.
