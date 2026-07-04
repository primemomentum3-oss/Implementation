# 12 — Visual Tech Stack and World-Class App Guide

This document defines the recommended technology stack for making Project Atlas feel visually powerful, polished, and world-class.

It assumes the app will use **Rust + Tauri** as the desktop shell/foundation, but explains what else is needed to create the actual premium visual experience.

This is not implementation code. It is a stack and quality framework for the builder-agent.

## 1. Core position

Rust + Tauri gives Project Atlas:

- Native desktop shell.
- Local file access.
- Secure app permissions.
- App packaging.
- Rust-side performance for file operations, validation helpers, local storage, and future scanners.
- A lightweight desktop wrapper around a web-based UI.

But the world-class visual experience will mostly come from the frontend running inside the Tauri WebView.

The recommended frontend stack is:

```text
Tauri + Rust
TypeScript
React + Vite
@xyflow/react / React Flow
ELK.js
D3 utilities
Tailwind CSS
Radix UI / shadcn-style primitives
Motion
Zustand
SQLite / IndexedDB
Markdown renderer
Command palette
Full test stack
```

## 2. Recommended full stack

## 2.1 Desktop shell

Use:

```text
Tauri
Rust
```

Purpose:

- Desktop app wrapper.
- Local filesystem access.
- Importing atlas bundles from disk.
- Reading `.project-atlas` folders later.
- SQLite storage access.
- Native menus and app window behavior.
- Secure command bridge between frontend and local system.
- Future local scanners and bundle builders.

Rust should not contain UI logic. Rust should support the app with local capabilities.

## 2.2 Frontend foundation

Use:

```text
TypeScript
React
Vite
pnpm
```

Purpose:

- Main UI application.
- Canvas workspace.
- Drawers.
- Lenses.
- Search.
- Tabs.
- Validation screens.
- Document viewer.
- Visual interaction state.

Why:

- TypeScript keeps the atlas contract safer.
- React is strong for complex interactive UI.
- Vite keeps development/build fast.
- This stack works well inside Tauri.

## 2.3 Canvas and graph rendering

Use:

```text
@xyflow/react / React Flow
```

Purpose:

- Main world-map canvas.
- Node rendering.
- Edge rendering.
- Custom node cards.
- Custom edges.
- Pan and zoom.
- Selection.
- Minimap.
- Canvas controls.
- Grouped/parent nodes.
- Focus mode.
- Lens-specific graph rendering.

Project Atlas should use React Flow as the main interactive graph/canvas layer unless a later performance test proves it cannot handle the needed scale.

## 2.4 Automatic layout

Use:

```text
ELK.js
```

Purpose:

- Position nodes automatically.
- Create readable layered diagrams.
- Support hierarchical/compound graph layout.
- Layout flow maps.
- Layout dependency maps.
- Layout Agent/MCP/tool hierarchy.
- Reduce manual placement work.

React Flow renders nodes and edges. ELK.js calculates positions.

This separation is important.

## 2.5 Custom visualization utilities

Use:

```text
D3
```

Use D3 selectively. Do not use it as the full UI framework.

Good D3 uses:

- Custom route curves.
- Timeline scales.
- Treemap calculations.
- Impact blast-radius rings.
- Data lens overlays.
- Path calculations.
- Small charts and visual summaries.

The main UI should still be React-based.

## 2.6 Optional high-performance rendering layer

Optional later:

```text
PixiJS
```

Only add PixiJS if React Flow is not enough for:

- Very large graph previews.
- Heavy animated background layers.
- Runtime trace animations.
- Heatmaps.
- Particle/glow effects.
- High-performance GPU-rendered visual overlays.

Do not start with PixiJS unless necessary.

Recommended approach:

```text
React Flow = primary interactive graph
PixiJS = optional advanced visual/performance layer later
```

## 3. Design system stack

## 3.1 Styling

Use:

```text
Tailwind CSS
CSS variables
Design tokens
```

Purpose:

- Dark theme.
- Layout.
- Spacing.
- Typography.
- Borders.
- Shadows.
- Glow effects.
- Responsive panels.
- Node card styling.
- Badges.
- Drawers.
- Filters.

Design tokens should be created early.

Suggested token groups:

```text
background.canvas
background.panel
background.panelRaised
background.card
border.subtle
border.active
text.primary
text.secondary
text.muted
accent.frontend
accent.backend
accent.data
accent.agent
accent.runtime
accent.warning
accent.danger
accent.success
```

## 3.2 UI primitives

Use:

```text
Radix UI primitives
shadcn-style component patterns
```

Purpose:

- Dialogs.
- Drawers/sheets.
- Dropdowns.
- Tabs.
- Tooltips.
- Popovers.
- Context menus.
- Scroll areas.
- Accordions.
- Sliders.
- Switches.
- Command palette shell.

Important: do not let the app look like a generic shadcn dashboard.

Use the primitives, but heavily customize the visual style so Project Atlas feels like its own premium product.

## 3.3 Motion and transitions

Use:

```text
Motion
```

Purpose:

- Drawer transitions.
- Lens switching.
- Node selection glow.
- Cluster expand/collapse.
- Focus mode transition.
- Command palette animation.
- Flow step transitions.
- Runtime trace replay.
- Snapshot diff transitions.

Motion should help orientation. It should not feel flashy or distracting.

Rules:

- Subtle.
- Fast.
- Smooth.
- Directional.
- Meaningful.

## 4. State, storage, and data handling

## 4.1 UI state

Use:

```text
Zustand
```

Purpose:

- Active project.
- Active lens.
- Selected node.
- Selected edge.
- Selected flow.
- Selected document.
- Drawer state.
- Canvas viewport state.
- Filters.
- Focus mode.
- Expanded/collapsed groups.
- Command palette state.
- Search query.

Keep graph data immutable after import. UI state should reference IDs.

## 4.2 Local storage

Recommended for Tauri desktop:

```text
SQLite
```

Optional browser-style fallback:

```text
IndexedDB
```

Store:

- Imported project records.
- Atlas snapshots.
- Validation reports.
- User notes.
- Saved views.
- Canvas layout overrides.
- Recent searches.
- App settings.

Do not store secret values.

## 4.3 Schema validation

Use:

```text
Zod or Valibot
JSON Schema export if useful
```

Purpose:

- Validate `AtlasBundle` imports.
- Validate nodes, edges, flows, artifacts, documents, traces, views, warnings.
- Produce user-readable validation reports.
- Prevent broken data from crashing the canvas.

## 5. Documents and Markdown

Use:

```text
react-markdown
remark-gfm
rehype-sanitize
```

Purpose:

- Render README files.
- Render AGENTS.md files.
- Render `.project-atlas/agent-instructions.md`.
- Render architecture docs.
- Render notes.

AGENTS.md is a first-class document type.

The user must be able to:

```text
Find AGENTS.md
Click AGENTS.md
See its scope
Read its important notes
Open/render Markdown content or preview
See related agents, folders, systems, flows, and MCP tools
```

Optional later:

```text
Monaco Editor in read-only mode
```

Use Monaco only for richer file/code previews. Do not make raw code the main experience.

## 6. Search and command palette

Use:

```text
cmdk or custom command palette
Fuse.js or MiniSearch
```

Purpose:

- Fast global navigation.
- Search nodes, edges, flows, steps, docs, files, folders, scripts, MCP tools, hooks, traces, warnings.
- Execute commands like focus, open, show impact, switch lens, open docs, open flow.

Search is critical because large maps need instant navigation.

Required searches must work:

```text
AGENTS
MCP
build
Create Project
Project Service
Stripe
DATABASE_URL
```

## 7. Icons and visual language

Use:

```text
lucide-react
custom SVG icons where needed
```

Required icon categories:

- Project.
- System.
- Frontend.
- Backend.
- API.
- Database.
- Table.
- File.
- Folder.
- Document.
- AGENTS.md.
- Script.
- Workflow.
- Hook.
- Webhook.
- MCP server.
- MCP tool.
- Agent.
- Runtime trace.
- Deployment.
- Warning.
- Evidence.
- Confidence.

Do not rely on color alone. Use icon + shape + label + badge.

## 8. Testing stack

Use:

```text
Vitest
React Testing Library
Playwright
axe-core accessibility checks
Storybook or Ladle
```

Purpose:

- Unit test schema validation.
- Unit test graph engine.
- Unit test lens filtering.
- Unit test impact analysis.
- Component test drawers, badges, docs viewer, flow timeline.
- End-to-end test import, canvas, search, AGENTS.md opening, MCP drilldown, impact view.
- Visual review component states.

Minimum required test coverage:

- Demo atlas imports.
- Broken atlas validation fails clearly.
- Overview canvas renders.
- Node drawer works.
- Edge drawer works.
- Flow timeline works.
- Docs/AGENTS.md viewer works.
- MCP server/tool drill-down works.
- Scripts/workflows lens works.
- Data lens works.
- Impact lens works.
- Search works.
- Snapshot/diff foundation works.

## 9. Recommended package map

Suggested package responsibilities:

```text
apps/desktop
  Tauri app and React frontend

packages/schema
  AtlasBundle types and runtime validation

packages/graph-engine
  Graph normalization, traversal, lens filtering, impact analysis

packages/ui
  Design system, tokens, primitives, badges, panels, drawers

packages/canvas
  React Flow rendering, custom nodes, custom edges, layout adapters

packages/renderers
  Flow timeline, docs viewer, trace viewer, folder tree, data lens

packages/agent-kit
  Agent prompts, setup instructions, validation checklists

packages/test-fixtures
  Demo and invalid atlas bundles
```

The builder-agent can adjust exact folder names, but must preserve the architectural separation.

## 10. World-class visual quality requirements

The app should feel premium because of consistency, not decoration.

Requirements:

- Dark canvas-first layout.
- Strong visual hierarchy.
- Smooth pan/zoom.
- Stable layouts.
- Beautiful custom node cards.
- Clear edge styling.
- Subtle glows and shadows.
- Useful minimap.
- Clean detail drawers.
- Excellent empty states.
- Fast search.
- Markdown docs that are pleasant to read.
- Clear confidence/evidence badges.
- Good keyboard shortcuts.
- Responsive interactions.
- No raw JSON in normal user flows.

## 11. Canvas-specific technology guidance

Use React Flow for the main map.

Expected custom React Flow features:

- Custom node components for each atlas node type.
- Custom edge components for each relationship type.
- Parent/group nodes for regions.
- Minimap styled to match the world-map concept.
- Controls styled as Project Atlas controls.
- Viewport persistence.
- Fit-to-view behavior.
- Focus mode subgraph rendering.
- Zoom-level detail changes.
- Selection and hover highlighting.
- Lens-specific graph filtering.

Use ELK.js before rendering to calculate readable positions.

Use D3 only for special calculations/visuals.

## 12. Performance guidance

Project Atlas must stay smooth for medium-large maps.

Strategies:

- Do not render the full raw graph by default.
- Collapse L4/L5 details until zoomed in or focused.
- Memoize graph queries.
- Precompute indexes.
- Virtualize long lists.
- Avoid unnecessary React re-renders.
- Keep graph data immutable.
- Use ID references in UI state.
- Use lens-specific subgraphs.
- Use worker/background processing later if needed.

Target behavior:

```text
The app should feel fast with the demo bundle.
The app should remain usable with thousands of nodes by collapsing/grouping details.
```

## 13. Visual stack decision hierarchy

When choosing between libraries or approaches, prioritize:

1. Canvas/world-map quality.
2. Maintainable code.
3. Strong TypeScript support.
4. Good performance.
5. Easy testing.
6. Extensibility for future agents.
7. Dark UI customization.
8. Accessibility.

Do not choose a library just because it looks impressive in a demo.

Choose tools that support the Project Atlas model.

## 14. Final recommended stack summary

```text
Desktop:
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
- optional PixiJS later

Design system:
- Tailwind CSS
- CSS variables/design tokens
- Radix UI primitives
- shadcn-style component patterns
- Motion
- lucide-react + custom SVG icons

State/data:
- Zustand
- SQLite through Tauri
- IndexedDB fallback if needed
- Zod or Valibot

Docs/search:
- react-markdown
- remark-gfm
- rehype-sanitize
- cmdk
- Fuse.js or MiniSearch

Testing:
- Vitest
- React Testing Library
- Playwright
- axe-core
- Storybook or Ladle
```

## 15. Stack acceptance criteria

The chosen stack is acceptable when the builder-agent can implement:

- A Tauri desktop app.
- A React/TypeScript visual frontend.
- A dark canvas-first world map.
- Custom atlas node cards.
- Custom relationship edges.
- Auto-layout.
- Smooth pan/zoom/minimap.
- Lens filtering.
- Detail drawers.
- Markdown/AGENTS.md viewer.
- Global search and command palette.
- Local project/snapshot storage.
- Full validation and test coverage.

## 16. Key warning

The stack alone will not make Project Atlas world-class.

The world-class feeling comes from combining:

```text
Strong AtlasBundle data contract
+ Clean graph engine
+ React Flow canvas
+ ELK auto-layout
+ Tailwind/Radix/Motion design system
+ Beautiful custom node cards
+ Excellent detail drawers
+ Fast global search
+ Markdown/AGENTS.md viewer
+ Honest confidence/evidence badges
+ Full testing
```

The builder-agent must implement all of these as one coherent product.
