---
name: cloudgate-build
description: >
  Build or edit a Cloudgate workflow-API ("project") through the Cloudgate MCP
  (/mcp, streamable HTTP, Bearer auth). Use when the user asks to create a project,
  add an endpoint/action, wire workflow nodes, clone a cookbook recipe, or publish
  a Cloudgate API. Mirrors the backend CORE_SYSTEM_PRIMER.
---

# Cloudgate project/workflow builder

Cloudgate APIs are workflow graphs: an endpoint (route + HTTP method) plus nodes,
served at /prod|sbx/{projectPath}/{route}. Terminology: Project = Controller;
Endpoint = Action = Workflow API; Action Logs = request logs (not definitions).

## Connection
- MCP endpoint: the scoped route is best — `…/mcp/workflow` for build/edit work,
  `…/mcp/data` for databases, `…/mcp/codegen` for read-only node scripting.
- Auth header on every call: `Authorization: Bearer {token}` (signed with
  Agent:McpTokenSigningKey; tokens are per-job/4h by default — use a longer-lived
  dev/service token for a standing Claude connection).
- Tools surface as `mcp__cloudgate__*` (workflow) and `mcp__cloudgate-data__*` (data tables)
  once connected. SQLite database tools are on **both** `/mcp/workflow` and `/mcp/data`.

## Mandatory rules (from the primer — do not skip)
- ALWAYS use live MCP tools for workflow work; NEVER guess graph JSON.
- Prefer structured node tools (add_request_workflow_node, add_function_workflow_node,
  add_database_workflow_node, remove_workflow_node) over hand-editing graph JSON.
- NEVER invent nodes[] or MainScript. ALWAYS validate_workflow_json before update_endpoint.
- Draft-only: if editState.isPublishedOnly, call begin_workflow_edit (or ask the user
  to click Edit Workflow) before any mutation.
- Safe apply: dry_run=true → explain the diff → dry_run=false only after user confirms.
- Mutating tools need Agent Edit permission; on agent_edit_required, switch to read-only.

## Build a new project (happy path)
1. list_projects — does the controller exist? If not, create_project.
2. create_endpoint (or create_hello_world_get_endpoint to bootstrap).
3. begin_workflow_edit, then add_*_workflow_node to wire request/function/database steps.
4. validate_workflow_json → update_endpoint.
5. publish_endpoint.
6. ui_navigate(/flows/workflows/{endpointId}) so the user lands on the result.

## Prefer a template when one fits
- list_cookbook_recipes → get_cookbook_recipe → clone_workflow_from_endpoint
  (dry_run first). Skip get_workflow_summary when templateEndpointId is returned.
- Platform templates: list_platform_templates → get_platform_template → import_platform_template.

## SQLite project databases
Cloudgate stores project databases as SQLite files. Workflow **Database nodes** use
`fileId` (not databaseId). Call `get_database_model` before your first DB mutation.

1. `list_databases(project_id)` — see existing databases (id + fileId).
2. `create_sqlite_database(project_id, name, dry_run=true)` → review → `dry_run=false`.
   Returns **fileId** — keep it for all later table/SQL/node steps.
3. Schema (pick one):
   - `create_database_table(file_id, table_name, columns_json, dry_run)` — **Id is auto-added**
     if omitted; column JSON example: `[{"name":"Email","type":"TEXT"}]`
   - `execute_database_sql(file_id, sql, dry_run)` — CREATE TABLE must include
     `Id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL` explicitly.
4. `list_database_tables(file_id)` — verify tables before wiring the workflow.
5. `begin_workflow_edit` → `add_database_workflow_node(file_id=…)` → validate → publish.

**Shortcut:** `import_platform_template(project_id, "database_crud.json", dry_run)` clones a
full CRUD API with database + nodes already wired.

**Data Tables** (tenant UI at /home/data-tables) are a different feature — use the
`cloudgate-data` MCP server (`list_data_tables`, `query_data_table`, etc.).

## Domain references (load lazily, only when needed)
get_database_model, get_selector_model, get_metrics_model, get_documentation_model,
get_under_attack_model, get_test_automation_model, get_import_agent_model,
get_api_key_model, get_origin_validator_model, get_auto_proxy_model,
get_schedule_model, get_websocket_model, get_sample_request_model.

## Verification (always last)
- Re-read the workflow (get_workflow_summary / get_workflow_graph) and confirm the
  nodes and route match the request.
- Confirm publish succeeded; do not claim success on a draft.
- Report endpoint URL + any manual follow-ups (selectors, API keys, CORS).
