# 04 — Atlas Data Contract

This file defines the first reusable Project Atlas data contract.

The UI, graph engine, scanners, and coding agents should all work against this contract.

The contract should be implemented as TypeScript types plus runtime validation in the actual app.

## 1. Contract goals

The data contract must support:

- Any project type.
- Any programming language.
- Any framework.
- Agent-authored data.
- Scanner-generated data.
- Runtime-observed data.
- Multiple lenses over the same graph.
- Drill-down from project overview to files/tools/docs.
- Evidence and confidence for every important claim.
- Snapshot comparison over time.

The contract should not assume a specific stack like Next.js, Python, Supabase, or MCP-only projects.

## 2. Top-level bundle

```ts
export type AtlasBundle = {
  schemaVersion: '1.0.0';
  bundleId: string;
  generatedAt: string;
  generatedBy: AtlasGenerator;
  project: AtlasProject;
  nodes: AtlasNode[];
  edges: AtlasEdge[];
  flows: AtlasFlow[];
  artifacts: AtlasArtifact[];
  documents: AtlasDocument[];
  traces?: AtlasTrace[];
  views?: AtlasView[];
  notes?: AtlasNote[];
  warnings?: AtlasWarning[];
  snapshots?: AtlasSnapshotReference[];
  metadata?: Record<string, unknown>;
};
```

## 3. Generator metadata

```ts
export type AtlasGenerator = {
  type: 'agent' | 'scanner' | 'manual' | 'hybrid';
  name: string;
  version?: string;
  agentModel?: string;
  scannerVersion?: string;
  sourceRunId?: string;
};
```

Example:

```json
{
  "type": "hybrid",
  "name": "Project Atlas Setup Agent + Static Scanner",
  "version": "0.1.0"
}
```

## 4. Project metadata

```ts
export type AtlasProject = {
  id: string;
  name: string;
  summary: string;
  rootPath?: string;
  repository?: AtlasRepository;
  primaryLanguages?: string[];
  frameworks?: string[];
  packageManagers?: string[];
  deploymentTargets?: string[];
  tags?: string[];
  createdAt?: string;
  updatedAt?: string;
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
};

export type AtlasRepository = {
  provider?: 'github' | 'gitlab' | 'bitbucket' | 'local' | 'other';
  url?: string;
  defaultBranch?: string;
  currentBranch?: string;
  commitSha?: string;
};
```

## 5. Node types

```ts
export type AtlasNodeType =
  | 'project'
  | 'workspace'
  | 'system'
  | 'domain'
  | 'feature'
  | 'flow'
  | 'flow_step'
  | 'frontend'
  | 'backend'
  | 'api_endpoint'
  | 'page'
  | 'screen'
  | 'component'
  | 'service'
  | 'module'
  | 'function'
  | 'middleware'
  | 'job'
  | 'worker'
  | 'cron_job'
  | 'script'
  | 'workflow'
  | 'hook'
  | 'webhook'
  | 'mcp_server'
  | 'mcp_tool'
  | 'agent'
  | 'prompt'
  | 'rule_file'
  | 'database'
  | 'table'
  | 'collection'
  | 'model'
  | 'schema'
  | 'storage_bucket'
  | 'queue'
  | 'cache'
  | 'external_service'
  | 'integration'
  | 'auth_provider'
  | 'payment_provider'
  | 'email_provider'
  | 'ai_provider'
  | 'deployment_target'
  | 'environment'
  | 'env_var'
  | 'config'
  | 'folder'
  | 'file'
  | 'document'
  | 'runtime_trace'
  | 'runtime_span'
  | 'unknown';
```

## 6. Node shape

```ts
export type AtlasNode = {
  id: string;
  type: AtlasNodeType;
  label: string;
  summary?: string;
  description?: string;
  level?: AtlasLevel;
  area?: string;
  tags?: string[];
  status?: AtlasNodeStatus;
  importance?: AtlasImportance;
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  artifactRefs?: string[];
  documentRefs?: string[];
  noteRefs?: string[];
  data?: AtlasNodeData;
  layout?: AtlasLayoutHint;
  metadata?: Record<string, unknown>;
};

export type AtlasLevel = 'L0' | 'L1' | 'L2' | 'L3' | 'L4' | 'L5';

export type AtlasNodeStatus =
  | 'active'
  | 'planned'
  | 'deprecated'
  | 'unknown'
  | 'broken'
  | 'needs_review';

export type AtlasImportance = 'low' | 'normal' | 'high' | 'critical';
```

## 7. Node data extensions

Use `data` for type-specific structured data.

```ts
export type AtlasNodeData = {
  path?: string;
  command?: string;
  url?: string;
  method?: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE' | 'OPTIONS' | 'HEAD' | string;
  inputs?: AtlasIOItem[];
  outputs?: AtlasIOItem[];
  reads?: string[];
  writes?: string[];
  errorModes?: AtlasErrorMode[];
  owner?: string;
  runtime?: string;
  packageName?: string;
  version?: string;
  sensitive?: boolean;
  mcp?: AtlasMcpData;
  hook?: AtlasHookData;
  script?: AtlasScriptData;
  envVar?: AtlasEnvVarData;
  deployment?: AtlasDeploymentData;
  [key: string]: unknown;
};

export type AtlasIOItem = {
  name: string;
  type?: string;
  description?: string;
  required?: boolean;
  sensitive?: boolean;
};

export type AtlasErrorMode = {
  condition: string;
  result: string;
  severity?: 'low' | 'medium' | 'high' | 'critical';
};
```

## 8. MCP data

```ts
export type AtlasMcpData = {
  serverName?: string;
  toolName?: string;
  command?: string;
  args?: string[];
  transport?: 'stdio' | 'http' | 'sse' | 'other' | 'unknown';
  clientRefs?: string[];
  permissions?: string[];
  inputSchemaSummary?: string;
  outputSchemaSummary?: string;
};
```

Example MCP server node:

```json
{
  "id": "node_mcp_github_server",
  "type": "mcp_server",
  "label": "GitHub MCP Server",
  "summary": "Provides repository and pull request tools to project agents.",
  "level": "L3",
  "area": "agent_tooling",
  "confidence": "confirmed_by_config",
  "data": {
    "mcp": {
      "serverName": "github",
      "transport": "stdio",
      "permissions": ["read_repository", "write_pull_request"]
    }
  },
  "artifactRefs": ["artifact_config_mcp_json"]
}
```

Example MCP tool node:

```json
{
  "id": "node_mcp_github_create_pull_request_tool",
  "type": "mcp_tool",
  "label": "Create Pull Request Tool",
  "summary": "Lets an agent create a pull request in GitHub.",
  "level": "L4",
  "area": "agent_tooling",
  "confidence": "agent_inferred",
  "data": {
    "mcp": {
      "serverName": "github",
      "toolName": "create_pull_request",
      "inputSchemaSummary": "Repository, branch, title, body, and changed files.",
      "outputSchemaSummary": "Pull request URL and metadata."
    }
  }
}
```

## 9. Hook data

```ts
export type AtlasHookCategory =
  | 'framework_hook'
  | 'automation_hook'
  | 'integration_webhook'
  | 'git_hook'
  | 'agent_hook'
  | 'package_lifecycle_hook'
  | 'unknown';

export type AtlasHookData = {
  category: AtlasHookCategory;
  trigger?: string;
  handler?: string;
  eventSource?: string;
  eventTypes?: string[];
};
```

Example:

```json
{
  "id": "node_hook_stripe_webhook",
  "type": "webhook",
  "label": "Stripe Payment Webhook",
  "summary": "Receives payment and subscription events from Stripe.",
  "confidence": "confirmed_by_code",
  "data": {
    "hook": {
      "category": "integration_webhook",
      "trigger": "Stripe sends a payment/subscription event",
      "handler": "app/api/stripe/webhook/route.ts",
      "eventSource": "Stripe"
    }
  }
}
```

## 10. Script data

```ts
export type AtlasScriptData = {
  command: string;
  packageManager?: string;
  scriptName?: string;
  workingDirectory?: string;
  whenToUse?: string;
  riskLevel?: 'low' | 'medium' | 'high' | 'critical' | 'unknown';
  expectedOutputs?: string[];
};
```

Example:

```json
{
  "id": "node_script_build",
  "type": "script",
  "label": "Build App",
  "summary": "Creates the production build for the app.",
  "confidence": "confirmed_by_config",
  "data": {
    "script": {
      "scriptName": "build",
      "command": "pnpm build",
      "packageManager": "pnpm",
      "whenToUse": "Before deployment or production verification.",
      "riskLevel": "medium"
    }
  }
}
```

## 11. Environment variable data

```ts
export type AtlasEnvVarData = {
  name: string;
  required?: boolean;
  usedByNodeRefs?: string[];
  description?: string;
  valueIncluded: false;
};
```

Rule: `valueIncluded` must always be false. Never store secret values.

## 12. Deployment data

```ts
export type AtlasDeploymentData = {
  provider?: string;
  environment?: 'local' | 'development' | 'preview' | 'staging' | 'production' | 'unknown';
  region?: string;
  url?: string;
  buildCommand?: string;
  startCommand?: string;
  runtime?: string;
};
```

## 13. Edge types

```ts
export type AtlasEdgeType =
  | 'contains'
  | 'calls'
  | 'reads'
  | 'writes'
  | 'creates'
  | 'updates'
  | 'deletes'
  | 'triggers'
  | 'triggered_by'
  | 'emits'
  | 'listens_to'
  | 'depends_on'
  | 'uses'
  | 'imports'
  | 'renders'
  | 'configured_by'
  | 'documents'
  | 'runs'
  | 'deploys_to'
  | 'handles_webhook'
  | 'exposes_tool'
  | 'uses_mcp_tool'
  | 'authenticates_with'
  | 'stores_in'
  | 'observed_as'
  | 'related_to';
```

## 14. Edge shape

```ts
export type AtlasEdge = {
  id: string;
  type: AtlasEdgeType;
  sourceId: string;
  targetId: string;
  label?: string;
  summary?: string;
  direction?: 'source_to_target' | 'target_to_source' | 'bidirectional';
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};
```

Example:

```json
{
  "id": "edge_dashboard_create_form__calls__api_create_project",
  "type": "calls",
  "sourceId": "node_dashboard_create_project_form",
  "targetId": "node_api_create_project",
  "label": "submits request to",
  "summary": "The form sends the create-project request to the backend API.",
  "confidence": "confirmed_by_code"
}
```

## 15. Flow shape

```ts
export type AtlasFlowType =
  | 'user_flow'
  | 'api_flow'
  | 'data_flow'
  | 'background_job'
  | 'webhook_flow'
  | 'agent_flow'
  | 'mcp_flow'
  | 'build_flow'
  | 'deployment_flow'
  | 'test_flow'
  | 'other';

export type AtlasFlow = {
  id: string;
  type: AtlasFlowType;
  label: string;
  summary: string;
  area?: string;
  trigger?: AtlasFlowTrigger;
  steps: AtlasFlowStep[];
  nodeRefs?: string[];
  artifactRefs?: string[];
  documentRefs?: string[];
  traceRefs?: string[];
  tags?: string[];
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};

export type AtlasFlowTrigger = {
  type: 'user_action' | 'api_request' | 'webhook' | 'cron' | 'agent_action' | 'script' | 'deployment' | 'manual' | 'unknown';
  description: string;
  sourceNodeRef?: string;
};
```

## 16. Flow step shape

```ts
export type AtlasFlowStep = {
  id: string;
  label: string;
  order: number;
  summary: string;
  nodeRefs?: string[];
  artifactRefs?: string[];
  documentRefs?: string[];
  inputs?: AtlasIOItem[];
  outputs?: AtlasIOItem[];
  reads?: string[];
  writes?: string[];
  calls?: string[];
  usesTools?: string[];
  errorPaths?: AtlasErrorMode[];
  expectedDurationMs?: number;
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};
```

Example flow:

```json
{
  "id": "flow_projects_create_project",
  "type": "user_flow",
  "label": "Create Project",
  "summary": "A user creates a new project from the dashboard.",
  "area": "projects",
  "trigger": {
    "type": "user_action",
    "description": "User submits the Create Project form.",
    "sourceNodeRef": "node_dashboard_create_project_form"
  },
  "steps": [
    {
      "id": "step_create_project_01_submit_form",
      "label": "Submit project form",
      "order": 1,
      "summary": "The user enters project details and submits the form.",
      "nodeRefs": ["node_dashboard_create_project_form"],
      "outputs": [
        {
          "name": "Create project request",
          "type": "object",
          "description": "Project name and selected settings."
        }
      ],
      "confidence": "confirmed_by_code"
    },
    {
      "id": "step_create_project_02_call_api",
      "label": "Call Create Project API",
      "order": 2,
      "summary": "The frontend sends the request to the backend API.",
      "nodeRefs": ["node_api_create_project"],
      "calls": ["node_api_create_project"],
      "confidence": "confirmed_by_code"
    }
  ],
  "confidence": "confirmed_by_code"
}
```

## 17. Artifact shape

Artifacts are concrete files, folders, scripts, configs, commands, routes, docs, or generated assets.

```ts
export type AtlasArtifactType =
  | 'file'
  | 'folder'
  | 'script'
  | 'config'
  | 'workflow'
  | 'schema'
  | 'migration'
  | 'route'
  | 'document'
  | 'prompt'
  | 'rule_file'
  | 'trace_file'
  | 'generated_asset'
  | 'other';

export type AtlasArtifact = {
  id: string;
  type: AtlasArtifactType;
  label: string;
  path?: string;
  command?: string;
  summary?: string;
  appliesToNodeRefs?: string[];
  supportsFlowRefs?: string[];
  tags?: string[];
  sensitive?: boolean;
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};
```

Example:

```json
{
  "id": "artifact_file_app_api_projects_route_ts",
  "type": "file",
  "label": "Create Project API Route File",
  "path": "app/api/projects/route.ts",
  "summary": "Contains the backend route for creating projects.",
  "appliesToNodeRefs": ["node_api_create_project"],
  "supportsFlowRefs": ["flow_projects_create_project"],
  "confidence": "confirmed_by_code"
}
```

## 18. Document shape

```ts
export type AtlasDocumentType =
  | 'readme'
  | 'agents_md'
  | 'architecture_doc'
  | 'setup_doc'
  | 'api_doc'
  | 'decision_record'
  | 'note'
  | 'changelog'
  | 'other';

export type AtlasDocument = {
  id: string;
  type: AtlasDocumentType;
  title: string;
  path?: string;
  summary: string;
  appliesToNodeRefs?: string[];
  appliesToPath?: string;
  importantNotes?: string[];
  conflictsWithDocRefs?: string[];
  confidence: AtlasConfidence;
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};
```

Example AGENTS.md document:

```json
{
  "id": "doc_agents_root",
  "type": "agents_md",
  "title": "Root AGENTS.md",
  "path": "AGENTS.md",
  "summary": "Repo-wide instructions for agents working in this project.",
  "appliesToNodeRefs": ["node_project_root"],
  "appliesToPath": ".",
  "importantNotes": [
    "Read before modifying files in this repo.",
    "Follow project validation rules before finishing."
  ],
  "confidence": "confirmed_by_docs"
}
```

## 19. Note shape

```ts
export type AtlasNote = {
  id: string;
  title: string;
  path?: string;
  explanationLevel: 'simple' | 'normal' | 'detailed' | 'technical';
  bodyMarkdown: string;
  appliesToNodeRefs?: string[];
  appliesToFlowRefs?: string[];
  appliesToArtifactRefs?: string[];
  tags?: string[];
  confidence: AtlasConfidence;
};
```

## 20. Runtime trace shape

```ts
export type AtlasTrace = {
  id: string;
  name: string;
  flowRef?: string;
  traceId?: string;
  startedAt?: string;
  durationMs?: number;
  status: 'ok' | 'error' | 'partial' | 'unknown';
  spans: AtlasTraceSpan[];
  evidence?: AtlasEvidence[];
  metadata?: Record<string, unknown>;
};

export type AtlasTraceSpan = {
  id: string;
  spanId?: string;
  parentSpanId?: string | null;
  label: string;
  summary?: string;
  nodeRefs?: string[];
  artifactRefs?: string[];
  startedAt?: string;
  durationMs?: number;
  status: 'ok' | 'error' | 'skipped' | 'unknown';
  attributes?: Record<string, string | number | boolean>;
  error?: {
    message?: string;
    type?: string;
  };
};
```

## 21. View shape

Views store lens-specific filters and layout hints.

```ts
export type AtlasLensId =
  | 'overview'
  | 'flows'
  | 'data'
  | 'dependencies'
  | 'agent_mcp_hooks'
  | 'docs'
  | 'folders'
  | 'scripts'
  | 'runtime'
  | 'deployment'
  | 'impact'
  | 'custom';

export type AtlasView = {
  id: string;
  lens: AtlasLensId;
  label: string;
  summary?: string;
  filters?: AtlasViewFilters;
  layoutHints?: AtlasLayoutHint[];
  pinnedNodeRefs?: string[];
  hiddenNodeRefs?: string[];
  collapsedNodeRefs?: string[];
};

export type AtlasViewFilters = {
  levels?: AtlasLevel[];
  nodeTypes?: AtlasNodeType[];
  edgeTypes?: AtlasEdgeType[];
  areas?: string[];
  tags?: string[];
  confidence?: AtlasConfidence[];
  includeInferred?: boolean;
  includeNeedsReview?: boolean;
};
```

## 22. Layout hints

```ts
export type AtlasLayoutHint = {
  nodeId?: string;
  lens?: AtlasLensId;
  group?: string;
  rank?: number;
  preferredColumn?:
    | 'user'
    | 'frontend'
    | 'backend'
    | 'business_logic'
    | 'data'
    | 'external'
    | 'automation'
    | 'agent_tools'
    | 'deployment'
    | 'docs'
    | 'unknown';
  x?: number;
  y?: number;
  pinned?: boolean;
};
```

## 23. Confidence

```ts
export type AtlasConfidence =
  | 'confirmed_by_code'
  | 'confirmed_by_config'
  | 'confirmed_by_schema'
  | 'confirmed_by_docs'
  | 'observed_runtime'
  | 'agent_inferred'
  | 'user_confirmed'
  | 'needs_review';
```

## 24. Evidence

```ts
export type AtlasEvidenceType =
  | 'file'
  | 'directory'
  | 'config'
  | 'document'
  | 'schema'
  | 'script'
  | 'workflow'
  | 'runtime_trace'
  | 'manual_note'
  | 'agent_inference'
  | 'test_result'
  | 'other';

export type AtlasEvidence = {
  type: AtlasEvidenceType;
  source: string;
  locator?: string;
  lineStart?: number;
  lineEnd?: number;
  reason: string;
  confidence?: AtlasConfidence;
};
```

Example:

```json
{
  "type": "config",
  "source": "package.json",
  "locator": "scripts.build",
  "reason": "Defines the build script for the project.",
  "confidence": "confirmed_by_config"
}
```

## 25. Warning shape

```ts
export type AtlasWarningSeverity = 'info' | 'low' | 'medium' | 'high' | 'critical';

export type AtlasWarning = {
  id: string;
  severity: AtlasWarningSeverity;
  label: string;
  summary: string;
  nodeRefs?: string[];
  flowRefs?: string[];
  artifactRefs?: string[];
  documentRefs?: string[];
  suggestedFix?: string;
};
```

Examples:

```json
{
  "id": "warning_missing_runtime_traces",
  "severity": "low",
  "label": "No runtime traces imported",
  "summary": "Expected flows exist, but no observed runtime traces are available yet.",
  "suggestedFix": "Capture a real run or ask an agent to convert logs into atlas trace format."
}
```

## 26. Snapshot reference

```ts
export type AtlasSnapshotReference = {
  id: string;
  createdAt: string;
  path?: string;
  label?: string;
  summary?: string;
  bundleHash?: string;
};
```

## 27. Minimal valid bundle example

```json
{
  "schemaVersion": "1.0.0",
  "bundleId": "bundle_demo_001",
  "generatedAt": "2026-07-04T10:00:00Z",
  "generatedBy": {
    "type": "agent",
    "name": "Project Atlas Setup Agent"
  },
  "project": {
    "id": "project_demo_saas",
    "name": "Demo SaaS App",
    "summary": "A demo project used to test Project Atlas.",
    "primaryLanguages": ["TypeScript"],
    "frameworks": ["React"],
    "confidence": "user_confirmed"
  },
  "nodes": [
    {
      "id": "node_project_root",
      "type": "project",
      "label": "Demo SaaS App",
      "summary": "Root project node.",
      "level": "L0",
      "confidence": "user_confirmed"
    },
    {
      "id": "node_frontend_app",
      "type": "frontend",
      "label": "Frontend App",
      "summary": "User-facing web application.",
      "level": "L1",
      "confidence": "agent_inferred"
    },
    {
      "id": "node_backend_api",
      "type": "backend",
      "label": "Backend API",
      "summary": "Server-side API layer.",
      "level": "L1",
      "confidence": "agent_inferred"
    }
  ],
  "edges": [
    {
      "id": "edge_project_contains_frontend",
      "type": "contains",
      "sourceId": "node_project_root",
      "targetId": "node_frontend_app",
      "confidence": "agent_inferred"
    },
    {
      "id": "edge_project_contains_backend",
      "type": "contains",
      "sourceId": "node_project_root",
      "targetId": "node_backend_api",
      "confidence": "agent_inferred"
    }
  ],
  "flows": [],
  "artifacts": [],
  "documents": []
}
```

## 28. Validation rules

The app/CLI must validate:

- `schemaVersion` is supported.
- `bundleId` exists.
- Project has `id`, `name`, `summary`, `confidence`.
- Node IDs are unique.
- Edge IDs are unique.
- Edge `sourceId` and `targetId` exist.
- Flow IDs are unique.
- Flow step IDs are unique inside each flow.
- Flow step `nodeRefs` exist.
- Artifact IDs are unique.
- Document IDs are unique.
- Referenced artifact/document/note/trace IDs exist where possible.
- Confidence labels are valid.
- Required fields are present.
- Secret values are not included in env var nodes.

Warnings should be produced for:

- Nodes without summary.
- Important nodes without evidence.
- Edges without label/summary.
- Flows without steps.
- Steps without node references.
- Documents not mapped to anything.
- Files/artifacts not mapped to anything.
- Too many `needs_review` items.
- Too many `agent_inferred` items in critical paths.

## 29. Rendering rules from contract

The UI should derive behavior from the contract:

- `level` controls default zoom visibility.
- `type` controls icon and node card style.
- `area` controls grouping.
- `tags` control filters.
- `confidence` controls badges.
- `evidence` controls trust drawer.
- `artifactRefs` and `documentRefs` control detail tabs.
- `layout` controls optional placement.
- `views` control saved lenses.

## 30. Backwards compatibility

Future schema versions must be migrated.

Add migrations like:

```ts
migrate_1_0_0_to_1_1_0(bundle: AtlasBundle_1_0_0): AtlasBundle_1_1_0
```

Do not break old target projects just because the UI gains new features.

## 31. Contract philosophy

The atlas is not only a code map.

It is a trustable visual knowledge model of a project.

That means the contract must represent:

- Code.
- Files.
- Docs.
- Scripts.
- Data.
- Runtime behavior.
- Tools.
- Agents.
- MCPs.
- Hooks.
- Explanations.
- Confidence.
- Evidence.

Agents can improve the atlas over time. The UI should become more useful as the atlas becomes richer.
