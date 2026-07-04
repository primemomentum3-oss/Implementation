# 12 — Visual Technology Stack for a World-Class App

This document defines the recommended technology stack beyond Rust and Tauri for building Project Atlas as a visually powerful, premium, canvas-first application.

It is written for the implementation agent. It should guide technical choices, visual architecture, and quality expectations without forcing exact low-level code.

## 1. Core principle

Rust and Tauri provide the native desktop shell.

They are excellent for:

- Local app packaging.
- Native windowing.
- Secure local file access.
- Rust-side commands.
- Local filesystem operations.
- Native dialogs.
- Local storage integrations.
- Performance-sensitive backend tasks.

But the world-class visual experience will mostly come from the frontend running inside the Tauri WebView.

For Project Atlas, the visual power comes from this combination:

```text
Tauri + Rust native shell
  + TypeScript frontend
  + React-based interactive UI
  + React Flow canvas
  + automatic graph layout
  + dark design system
  + smooth motion
  + excellent detail drawers
  + fast search
  + polished Markdown/docs viewer
  + local-first storage
  + full testing
```

The builder-agent should not assume that Tauri alone creates a premium product. Tauri creates the container. The frontend stack creates the experience.

## 2. Recommended full stack

Use this as the default stack unless the implementation repo strongly requires a different choice.

```text
Desktop shell:
- Tauri
- Rust

Frontend app:
- TypeScript
- React
- Vite
- pnpm

Canvas and graph visuals:
- @xyflow/react / React Flow
- ELK.js
- D3 utilities
- optional PixiJS later for heavier visual effects

Styling and UI system:
- Tailwind CSS
- CSS variables / design tokens
- Radix UI primitives
- shadcn-style components, heavily customized
- Motion

State and data:
- Zustand
- Zod or Valibot for runtime schema validation
- SQLite through Tauri where practical
- IndexedDB fallback where useful

Docs and search:
- react-markdown
- remark-gfm
- rehype-sanitize
- cmdk or equivalent command palette
- Fuse.js or MiniSearch

Icons and visual language:
- lucide-react
- custom SVG icons for Project Atlas-specific concepts

Testing and quality:
- Vitest
- React Testing Library
- Playwright
- axe-core accessibility checks
- Storybook or Ladle for component review
```

## 3. Recommended first-version stack

For the first complete build, prioritize this subset:

```text
Tauri + Rust
TypeScript
React
Vite
@xyflow/react
ELK.js
Tailwind CSS
Radix UI primitives
Motion
Zustand
Zod or Valibot
react-markdown
cmdk
Fuse.js or MiniSearch
lucide-react
Vitest
Playwright
```

Do not add PixiJS, Monaco, complex 3D rendering, or advanced GPU effects until the core product works.

## 4. Frontend foundation

## 4.1 TypeScript

Use TypeScript as the primary frontend language.

Reason:

- The AtlasBundle contract is complex.
- Nodes, edges, flows, artifacts, documents, traces, warnings, and views need strong types.
- Future agents must be able to refactor safely.
- Graph queries and view models need predictable inputs/outputs.

The builder-agent should create types for:

- AtlasBundle.
- AtlasNode.
- AtlasEdge.
- AtlasFlow.
- AtlasFlowStep.
- AtlasArtifact.
- AtlasDocument.
- AtlasTrace.
- AtlasView.
- ValidationReport.
- RenderGraph.
- NodeDetailViewModel.
- FlowDetailViewModel.
- ImpactReport.
- SearchResult.

## 4.2 React

Use React for the frontend UI.

Reason:

Project Atlas needs many interactive UI surfaces:

- Canvas.
- Custom graph nodes.
- Custom graph edges.
- Drawers.
- Panels.
- Tabs.
- Filters.
- Timelines.
- Markdown viewer.
- Command palette.
- Lens switcher.
- Runtime trace replay.
- Snapshot diff.

React is a strong fit for this kind of stateful, component-based interface.

## 4.3 Vite

Use Vite for frontend build/dev tooling.

Reason:

- Fast local development.
- Strong TypeScript support.
- Works well with React.
- Commonly used with Tauri frontends.

## 5. Canvas and graph visualization stack

## 5.1 @xyflow/react / React Flow

Use React Flow as the main world-map canvas renderer.

Project Atlas needs:

- Pan.
- Zoom.
- Node selection.
- Edge selection.
- Custom node cards.
- Custom edge rendering.
- Minimap.
- Controls.
- Grouping.
- Subflows.
- Focus mode.
- Canvas overlays.
- Interactive graph navigation.

React Flow should be the default canvas engine because it already matches the mental model of interactive node-and-edge systems.

Use React Flow for:

```text
Overview world map
Flow canvas
Dependency lens
Data lens
Agent/MCP/Hooks lens
Deployment lens
Impact map foundation
```

## 5.2 ELK.js

Use ELK.js for automatic graph layout.

React Flow renders nodes and edges, but the app still needs readable positioning.

Use ELK.js for:

- Layered overview layout.
- Flow step layout.
- Dependency maps.
- Agent/MCP/tool hierarchy.
- Data read/write maps.
- Compound/nested graph layout.
- Stable auto-layout for imported bundles.

Important rule:

```text
The user should not need to manually arrange a project map before it becomes readable.
```

ELK.js should calculate layout positions. React Flow should render them.

## 5.3 D3 utilities

Use D3 selectively as a utility layer, not as the full UI framework.

Use D3 for:

- Custom path calculations.
- Curved flow routes.
- Timeline scales.
- Impact blast-radius rings.
- Treemap-style folder views.
- Mini visual summaries.
- Experimental force-layout previews if useful.

Do not use D3 to build the entire application UI. Use it only where it improves custom visualization logic.

## 5.4 Optional PixiJS

PixiJS is optional and should be added only if the app needs heavier 2D GPU rendering.

Do not start with PixiJS unless React Flow becomes a real limitation.

Possible later use cases:

- Thousands of background visual elements.
- Animated runtime trace particles.
- Large heatmaps.
- Glow fields.
- Very large graph previews.
- High-performance visual effects behind React Flow.

Recommended first-version decision:

```text
React Flow first.
PixiJS later only if needed.
```

## 6. Styling and visual design stack

## 6.1 Tailwind CSS

Use Tailwind CSS for styling the app.

Use it for:

- Dark theme.
- Layout.
- Spacing.
- Typography.
- Borders.
- Cards.
- Panels.
- Drawers.
- Node cards.
- Badges.
- Hover states.
- Focus states.
- Responsive behavior.

Tailwind should be backed by CSS variables/design tokens so the app has one coherent visual identity.

## 6.2 Design tokens

Create design tokens early.

Recommended tokens:

```text
background.canvas
background.app
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
accent.docs
accent.warning
accent.danger
accent.success
shadow.soft
shadow.glow
```

Project Atlas should not look like an unmodified component library. It should have its own design identity.

## 6.3 Radix UI primitives

Use Radix UI primitives for accessible interaction patterns.

Use Radix-style primitives for:

- Dialogs.
- Popovers.
- Tooltips.
- Tabs.
- Dropdowns.
- Context menus.
- Scroll areas.
- Accordions.
- Sliders.
- Switches.

Radix gives strong behavior. Project Atlas should provide the custom visual styling.

## 6.4 shadcn-style components

A shadcn-style component structure is useful, but the final app should not look generic.

Use the pattern:

```text
Accessible primitive
  + local component wrapper
  + Project Atlas design tokens
  + custom dark visual styling
```

Do not simply install default components and leave them visually unchanged.

## 6.5 Motion

Use Motion for premium transitions and interactions.

Use it for:

- Drawer open/close.
- Command palette open/close.
- Lens switching.
- Node selection glow.
- Focus mode transitions.
- Cluster expand/collapse.
- Runtime trace replay.
- Snapshot diff reveal.
- Flow step transitions.

Motion should be subtle and functional.

Bad animation:

```text
Distracting, decorative movement everywhere.
```

Good animation:

```text
Motion helps the user understand where they are, what changed, and what became selected.
```

## 7. State management

## 7.1 Zustand

Use Zustand for app/session UI state.

Store:

- Active project ID.
- Active lens.
- Selected node ID.
- Selected edge ID.
- Selected flow ID.
- Selected document ID.
- Canvas viewport state.
- Current filters.
- Focus mode state.
- Expanded/collapsed groups.
- Drawer open/closed state.
- Command palette state.
- Search query.
- Snapshot comparison state.

Important rule:

```text
Graph data should be normalized and mostly immutable after import.
UI state should reference graph objects by ID.
```

Do not duplicate large graph objects throughout component state.

## 7.2 Server/cache state

If the app needs async storage loading, use a query/cache layer where useful.

For a local-first app, this can remain simple at first.

## 8. Schema validation

## 8.1 Zod or Valibot

Use a runtime schema validation library.

The app must validate imported atlas bundles before rendering them.

Validate:

- Top-level bundle structure.
- Schema version.
- Project metadata.
- Node IDs.
- Edge references.
- Flow step references.
- Artifact/document references.
- Confidence labels.
- Secret-value violations.

The validation layer is one of the most important trust features in the app.

## 9. Local storage

## 9.1 SQLite through Tauri

For the desktop app, prefer SQLite for durable local storage.

Store:

- Imported project records.
- Atlas bundles.
- Snapshot history.
- Validation reports.
- Saved views.
- User notes.
- Canvas layout overrides.
- Recent searches.

## 9.2 IndexedDB fallback

IndexedDB can be used for browser-like fallback storage or early development.

## 9.3 Storage rules

Never store secret values.

Environment variables should be stored by name only.

Good:

```text
DATABASE_URL exists and is required.
```

Bad:

```text
DATABASE_URL=postgres://actual-secret-value
```

## 10. Markdown and document viewing

## 10.1 Markdown stack

Use:

```text
react-markdown
remark-gfm
rehype-sanitize
```

This is needed for:

- README.md.
- AGENTS.md.
- Agent instructions.
- Setup docs.
- Architecture docs.
- Notes.
- Decision records.

The Markdown viewer should be visually polished and integrated into the app.

## 10.2 AGENTS.md behavior

AGENTS.md must be first-class.

The app should show:

- Title.
- Path.
- Scope.
- Applies-to area.
- Summary.
- Important notes.
- Related agents.
- Related MCP tools.
- Related flows.
- Markdown preview or content if available.

## 10.3 Optional Monaco Editor

Monaco can be considered later for read-only source/code previews.

Do not make Monaco a core requirement for the first version.

Project Atlas should explain code projects visually first, not become a code editor.

## 11. Search and command palette

## 11.1 Command palette

Use `cmdk` or an equivalent command palette pattern.

The command palette should support:

- Search.
- Jump to item.
- Focus item on canvas.
- Open detail drawer.
- Open flow.
- Open document.
- Show impact.
- Switch lens.
- Show low-confidence items.
- Show warnings.

## 11.2 Search engine

Use Fuse.js or MiniSearch.

Search across:

- Nodes.
- Edges.
- Flows.
- Flow steps.
- Artifacts.
- Documents.
- Notes.
- Runtime traces.
- Warnings.
- File paths.
- Script commands.
- MCP tools.
- Hook triggers.
- Evidence sources.

Search is critical because large project maps need direct navigation.

## 12. Icons and visual language

Use `lucide-react` as the base icon library.

Add custom icons where needed for Project Atlas-specific items.

Required icon categories:

```text
Project
System
Frontend
Backend
API endpoint
Page/screen
Component
Service
Database
Table
File
Folder
Document
AGENTS.md
Script
Workflow
Hook
Webhook
MCP server
MCP tool
Agent
Runtime trace
Deployment
Environment variable
Warning
Evidence
Confidence
Search
Impact
```

Do not rely on color alone. Icons, labels, and badges should work together.

## 13. Testing stack

## 13.1 Vitest

Use Vitest for unit tests.

Test:

- Schema validation.
- Graph normalization.
- Edge integrity.
- Lens filtering.
- Impact analysis.
- Path finding.
- Search indexing.
- Snapshot diff.

## 13.2 React Testing Library

Use for component behavior tests.

Test:

- Import validation screen.
- Detail drawer.
- Flow timeline.
- Docs viewer.
- Search results.
- Lens switcher.
- Warning cards.

## 13.3 Playwright

Use Playwright for end-to-end tests.

Test flows:

- Open app.
- Load demo project.
- Open Overview Lens.
- Click Project Service.
- Open Create Project flow.
- Search AGENTS.md.
- Open MCP server/tool.
- Open Scripts lens and Build App.
- Open Impact Lens.
- Open Runtime Lens.

## 13.4 axe-core

Use accessibility checks for key screens.

The app is visual, but it still needs:

- Readable contrast.
- Keyboard-accessible command palette.
- Accessible drawers/dialogs.
- Non-color-only status indicators.

## 13.5 Storybook or Ladle

Use Storybook or Ladle to develop/review components in isolation.

Useful component stories:

- Node cards by type.
- Confidence badges.
- Evidence badges.
- Warning cards.
- Detail drawer tabs.
- Flow step cards.
- Document viewer.
- Search result cards.
- Empty states.

## 14. Tauri-side supporting needs

Beyond base Tauri/Rust, the desktop app may need Tauri capabilities/plugins for:

- File open dialog.
- Folder selection.
- Reading imported local files.
- Writing local workspace data.
- SQLite storage.
- App updater later.
- Secure command invocation if a scanner/CLI is added.

Tauri-side responsibilities should be narrow:

```text
Native access, local storage, filesystem boundaries, import/export, and safe backend commands.
```

Frontend responsibilities should remain:

```text
Visualization, graph exploration, lenses, detail views, search, and user experience.
```

## 15. Performance guidance

Project Atlas may need to handle large project maps.

Use these strategies:

- Normalize graph once after import.
- Build indexes once.
- Memoize lens view models.
- Render collapsed groups by default.
- Avoid rendering thousands of nodes at once.
- Use zoom-level detail rules.
- Virtualize long lists.
- Use Web Workers later if graph analysis becomes heavy.
- Store canvas positions/layout hints.
- Use stable IDs for React rendering.

Performance goal:

```text
The default overview should remain smooth and readable even when the full atlas contains thousands of nodes.
```

## 16. Visual polish requirements

A world-class looking app needs visual consistency.

The builder-agent should implement:

- Dark canvas background.
- Subtle grid or atmospheric map background.
- Smooth panel surfaces.
- High-quality node cards.
- Clear icons.
- Consistent badges.
- Soft shadows and glows.
- Strong hover/selection states.
- Calm transitions.
- Readable typography.
- Good empty states.
- Helpful skeleton/loading states.
- Canvas minimap.
- Command palette.
- Polished Markdown viewer.

Avoid:

- Default white UI.
- Generic admin dashboard look.
- Rainbow color overload.
- Overly neon cyberpunk visuals.
- Tiny unreadable labels.
- Hairball graphs.
- Raw JSON interfaces.

## 17. Design quality checklist

Before calling the UI polished, verify:

```text
Does the app feel dark and premium?
Is the canvas clearly the center of the experience?
Can the user understand the main project map without opening code?
Are nodes visually distinct by type?
Are confidence and warning states obvious?
Are docs/AGENTS.md easy to open and read?
Can MCP servers/tools be explored visually?
Do flow routes feel like routes through the project world?
Does focus mode reduce complexity?
Does search make large maps navigable?
Do animations help orientation instead of distracting?
Does the app avoid looking like a default component template?
```

## 18. Build priority for visual stack

Implement in this order:

1. TypeScript types and validation.
2. React app shell inside Tauri.
3. Tailwind design tokens.
4. Base dark layout.
5. React Flow canvas.
6. Custom node cards and edges.
7. ELK auto-layout.
8. Detail drawer.
9. Flow timeline.
10. Lens switcher.
11. Markdown/AGENTS.md viewer.
12. Search and command palette.
13. Motion transitions.
14. Snapshot/diff visuals.
15. Runtime trace visuals.
16. Optional advanced visual effects.

## 19. Decision rules for the builder-agent

When choosing tools or implementation patterns, follow these rules:

```text
If it affects the graph meaning, put it in graph-engine/view-model logic.
If it affects presentation only, put it in UI components.
If it affects trust, put it in validation/evidence/confidence logic.
If it affects visual navigation, design it around the canvas/world-map metaphor.
If it affects user understanding, use plain-language summaries and progressive disclosure.
If it affects future extensibility, prefer typed contracts and modular boundaries.
```

## 20. Final recommendation

The strongest stack for Project Atlas is:

```text
Tauri + Rust
TypeScript + React + Vite
React Flow + ELK.js + selective D3
Tailwind + Radix + Motion
Zustand + Zod/Valibot
SQLite/local storage
react-markdown + command palette + fuzzy search
Vitest + Playwright + component review
```

This combination gives the app:

- Native desktop capabilities.
- A powerful interactive canvas.
- Automatic layout.
- Beautiful dark UI.
- Smooth transitions.
- Strong local-first behavior.
- Trustworthy validation.
- Reusable architecture.
- A strong foundation for future agents to extend.

The builder-agent should start with this stack unless there is a clear repository-specific reason to choose otherwise.
