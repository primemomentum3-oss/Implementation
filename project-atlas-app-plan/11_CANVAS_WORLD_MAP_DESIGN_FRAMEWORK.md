# 11 — Canvas World-Map Design Framework

This document captures the core visual essence of Project Atlas.

Project Atlas should not feel like a normal dashboard with diagrams attached. It should feel like a living, zoomable 2D world map of a project.

The builder-agent must preserve this idea throughout implementation.

## 1. Core metaphor

A software project should be represented like a world map.

```text
Project = world
Major systems = continents
Domains/features = countries
Flows = roads, routes, pipelines, or journeys
Services/APIs/data stores = cities
Files/docs/scripts/hooks/MCP tools = buildings and landmarks
Runtime traces = recorded trips through the world
Warnings = hazard markers
Confidence/evidence = map reliability indicators
```

The user should be able to visually travel through the project.

## 2. Why this matters

The target user does not want to read raw code to understand a system.

They want to see:

- What exists.
- Where it lives.
- What connects to what.
- What happens step by step.
- Which files/docs/tools/scripts support each thing.
- What an agent should update or review.

A world-map canvas gives the user spatial memory.

Instead of remembering:

```text
app/api/projects/route.ts calls lib/project-service.ts which writes db/schema.ts tables
```

The user remembers:

```text
The Create Project road goes from the frontend area to the backend city, passes through auth, writes to the data region, and sends an email to an external service.
```

That is the level of clarity Project Atlas must create.

## 3. Map layers

The canvas must support layered detail.

## 3.1 Layer 0 — Project world

Shows:

- Project root.
- Main purpose.
- Main regions.

User question:

```text
What project am I looking at?
```

## 3.2 Layer 1 — Continents / systems

Shows:

- Frontend.
- Backend.
- Data.
- External services.
- Agent tooling.
- Automation.
- Deployment.
- Runtime.

User question:

```text
What are the main parts of this project?
```

## 3.3 Layer 2 — Countries / domains

Shows:

- Billing.
- Project creation.
- User management.
- Agent review.
- Docs/instructions.
- CI/CD.

User question:

```text
What functional areas exist inside each system?
```

## 3.4 Layer 3 — Cities / services and flows

Shows:

- Pages.
- APIs.
- Services.
- Webhooks.
- Jobs.
- Major flows.
- Database tables.
- MCP servers.

User question:

```text
What does this area actually do?
```

## 3.5 Layer 4 — Buildings / concrete supporting items

Shows:

- Files.
- Folders.
- Docs.
- AGENTS.md.
- Scripts.
- Configs.
- Hooks.
- MCP tools.
- Runtime spans.

User question:

```text
What concrete things support this behavior?
```

## 3.6 Layer 5 — Interior details

Shows:

- Inputs.
- Outputs.
- Commands.
- Paths.
- Evidence.
- Confidence.
- Notes.
- Error paths.
- Verification checklists.

User question:

```text
What exactly should I know or ask an agent to change/check?
```

## 4. Zoom behavior

Zoom must change the level of detail.

It should not only scale the same objects.

## 4.1 Far zoom

Visible:

- L0/L1 items.
- Major system names.
- Main relationships.
- Warning counts.
- Health badges.

Hidden:

- File paths.
- Small docs.
- Low-level scripts.
- Detailed edge labels.

Feeling:

```text
I understand the whole project shape.
```

## 4.2 Medium zoom

Visible:

- L2/L3 items.
- Features.
- APIs.
- Services.
- Data stores.
- External services.
- Flows as routes.
- Confidence markers.

Feeling:

```text
I understand how the main areas connect.
```

## 4.3 Close zoom

Visible:

- L4 items.
- Flow steps.
- Supporting files.
- Docs.
- Scripts.
- Hooks.
- MCP servers/tools.
- Reads/writes.

Feeling:

```text
I understand what supports this part.
```

## 4.4 Deep zoom

Visible:

- Metadata.
- Paths.
- Commands.
- Inputs/outputs.
- Evidence details.
- Error paths.
- Runtime span details.

Feeling:

```text
I know what to inspect or ask an agent to change.
```

## 5. Spatial layout rules

The world should have consistent geography so users build memory.

Recommended default geography:

```text
Left side:
  Users and user actions.

Left-center:
  Frontend pages, screens, forms, UI components.

Center:
  Backend APIs, services, business logic.

Right-center:
  Data stores, schemas, tables, queues, storage.

Right side:
  External services, integrations, providers.

Top-right:
  Agents, MCP servers/tools, hooks, automation.

Bottom-right:
  Deployment, environments, CI/CD, runtime traces.

Bottom-left:
  Docs, README, AGENTS.md, notes, instructions.
```

The exact layout can adapt, but it should remain stable across sessions.

## 6. Region design

A region is a visual grouping.

Examples:

- Frontend region.
- Backend region.
- Data region.
- Agent Tooling region.
- Billing region.
- Docs region.

Regions should have:

- Label.
- Subtle boundary.
- Count badges.
- Health/warning badge.
- Expand/collapse control.
- Summary on hover/click.

Region visual treatment:

- Soft dark panel/territory background.
- Slight border glow when active.
- Muted when unrelated.
- Stronger when focused.

## 7. Flow route design

Flows should feel like routes through the map.

A flow route can connect:

```text
User Action -> Frontend -> API -> Auth -> Service -> Database -> External Service -> Response
```

Flow routes should support:

- Direction arrows.
- Step numbers.
- Route highlighting.
- Error branches.
- Runtime-observed overlay.
- Expected vs observed comparison.

When a flow is selected, the rest of the map should dim so the route becomes obvious.

## 8. Node density rules

The canvas must avoid visual overload.

Rules:

- Never show every node at default zoom.
- Collapse low-level file/function/config nodes into their parent region by default.
- Show counts instead of all details when zoomed out.
- Use focus mode for local detail.
- Use lens filters to reduce unrelated areas.
- Show detailed labels only when there is enough space.

Bad default:

```text
300 nodes, all visible, all labels shown, all edges visible.
```

Good default:

```text
12 major regions/systems, important connections, warning counts, drill-down available.
```

## 9. Edge density rules

Edges can destroy readability if overused.

Rules:

- Show only important edges at high level.
- Hide low-level import/dependency edges unless in Dependency Lens.
- Bundle or summarize repeated edges.
- Use edge labels only when helpful.
- On hover/select, reveal more related edges.
- In focus mode, show local edges in detail.

Edge priority from most visible to least visible:

1. Flow route edges.
2. Calls/triggers edges.
3. Reads/writes edges.
4. Uses/configured-by edges.
5. Documents/supports edges.
6. Low-level imports.

## 10. Visual grammar

## 10.1 Node shape by meaning

The exact shapes can vary, but the meaning should be consistent.

Suggested grammar:

- Systems/regions: large rounded territories.
- Features/domains: medium cards inside territories.
- Flows: route cards or path labels.
- Steps: numbered cards.
- Data: cylinder/table-like cards or database icons.
- Docs: document cards.
- AGENTS.md: document card with agent badge.
- Scripts: terminal/command cards.
- MCP servers: server/tooling cards.
- MCP tools: small tool cards attached to MCP server.
- Hooks: trigger/event cards.
- Runtime spans: pulse/timeline cards.
- Warnings: hazard markers.

## 10.2 Badge grammar

Every important card can show badges:

- Type.
- Confidence.
- Warning.
- Runtime observed.
- Agent inferred.
- Needs review.
- Sensitive data.
- Has docs.
- Has flows.

Badges should be short and readable.

## 10.3 Color grammar

Use dark base colors and restrained accents.

Suggested semantic accents:

- Frontend: cool blue accent.
- Backend: violet/indigo accent.
- Data: cyan/teal accent.
- External services: amber accent.
- Agent/MCP tooling: purple accent.
- Docs: soft gray/blue accent.
- Runtime: pink accent.
- Warnings: yellow/orange/red based on severity.

Do not make the whole map rainbow-colored. Use color as guidance, not decoration.

## 11. Interaction model

## 11.1 Click

Click selects an item and opens details.

## 11.2 Double click / drill

Double click or explicit drill action zooms into that region/item.

## 11.3 Hover

Hover shows a lightweight preview and highlights connections.

## 11.4 Focus

Focus mode centers selected item and shows only relevant neighborhood.

## 11.5 Path mode

Path mode shows a route from one item to another.

Example:

```text
Show path from Create Project Form to Projects Table.
```

## 11.6 Lens switch

Lens switch changes what the map emphasizes without discarding context.

Example:

- User is focused on Project Service.
- Switches from Overview to Data Lens.
- The map should keep Project Service context and reveal reads/writes.

## 12. Detail drawer as guidebook

The canvas is the map.

The detail drawer is the guidebook.

Clicking anything opens the guidebook entry.

Each entry should explain:

- What it is.
- Why it exists.
- What region it belongs to.
- What it connects to.
- What flows use it.
- What files/docs/scripts/tools support it.
- What evidence supports it.
- What confidence level it has.
- What may break if it changes.

The drawer should never be a raw dump of JSON.

## 13. Document landmarks

Docs are landmarks on the map.

AGENTS.md is a special landmark because it controls how agents should work.

The user should be able to:

```text
Find AGENTS.md on the map
Click it
See its scope
Read its important notes
Open its Markdown preview/source reference
See which systems/agents it affects
```

Docs should appear in:

- Docs Lens.
- Agent/MCP/Hooks Lens if agent-related.
- Detail drawers for systems they explain.
- Search results.

## 14. Agent/MCP/tool islands

Agent tooling should be visually understandable.

Suggested map pattern:

```text
Agent Tooling Region
  Implementation Agent
    uses -> GitHub MCP Server
      exposes -> Read Repository Tool
      exposes -> Create Pull Request Tool
    follows -> Root AGENTS.md
    updates -> .project-atlas folder
```

The user should be able to see how agents are equipped to act in the project.

## 15. Hooks as trigger points

Hooks should look like trigger/event points.

Examples:

- Stripe Webhook.
- Git pre-commit hook.
- CI workflow trigger.
- Package postinstall script.
- Framework lifecycle hook.

Every hook should explain:

- What triggers it.
- What handles it.
- What it affects.
- What can fail.

## 16. Runtime as animated/observed route

Runtime traces should feel like observed trips through the map.

When a runtime trace is selected:

- Highlight observed nodes in order.
- Show timings.
- Show failed spans.
- Compare expected route to observed route.
- Mark unmapped runtime spans.

This helps the user distinguish:

```text
What the project is expected to do
vs
What actually happened
```

## 17. Change impact as blast radius

Impact view should feel like a blast-radius map.

When an item is selected:

- Selected item is center.
- Direct dependents are first ring.
- Indirect dependents are second/third rings.
- Affected flows are highlighted.
- Affected docs/scripts/tests are listed.
- Risk level is visible.

This helps the user know what to ask an agent to verify.

## 18. Lenses as map modes

Lenses should feel like map modes.

Examples:

```text
Overview Lens = political map
Flow Lens = route map
Data Lens = utility/data pipeline map
Dependency Lens = infrastructure dependency map
Docs Lens = landmarks/library map
Agent/MCP/Hooks Lens = automation/tooling map
Runtime Lens = traffic/replay map
Impact Lens = risk/blast-radius map
Deployment Lens = environment map
```

The same world, different overlays.

## 19. Progressive disclosure checklist

The builder-agent should verify:

- Default view is not overwhelming.
- Major systems are immediately visible.
- User can drill into a system.
- User can drill into a flow.
- User can drill into a step.
- User can see supporting files/docs/scripts/tools.
- User can return back using breadcrumbs.
- Search can jump anywhere.
- Focus mode can isolate context.

## 20. Canvas acceptance tests

The canvas implementation is acceptable when:

1. A user can identify the main project regions at a glance.
2. A user can zoom into the Create Project flow and understand each step.
3. A user can click AGENTS.md and read its mapped instructions.
4. A user can inspect MCP server -> MCP tools.
5. A user can focus on Project Service and see upstream/downstream impact.
6. A user can search for `build`, `MCP`, `AGENTS`, and `create project` and jump to results.
7. The map remains calm and readable.

## 21. Final design standard

Project Atlas should make a project feel less invisible.

The final experience should feel like:

```text
I am not reading code.
I am exploring a map.
I can see the systems.
I can follow the roads.
I can click landmarks.
I can inspect details.
I can ask agents better questions.
I know what is confirmed and what needs review.
```

That is the design goal.
