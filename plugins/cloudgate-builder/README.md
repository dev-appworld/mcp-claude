# cloudgate-builder (Cowork plugin)

Bundles three things so Cowork can build Cloudgate workflow-APIs:

1. **MCP server `cloudgate`** (`.mcp.json`) — `mcp-remote` bridge to `/mcp/workflow`
   (projects, endpoints, nodes, **SQLite databases**).
2. **MCP server `cloudgate-data`** (`.mcp.json`) — same token, `/mcp/data`
   (Data Tables + SQLite tools).
3. **Skill `cloudgate-build`** (`skills/cloudgate-build/`) — the playbook that makes
   Claude use the tools correctly (dry-run, begin_workflow_edit, validate, publish;
   never invent graph JSON). Mirrors the backend CORE_SYSTEM_PRIMER.

## Before you install — paste your token
`.mcp.json` ships with a placeholder (or your dev token). Replace the Bearer value in
**both** `cloudgate` and `cloudgate-data` with the token from `npm run mint`, then
re-zip / re-package. Treat the packaged plugin as a secret — it contains standing access.

Update the URL too if your `/mcp` isn't at the ngrok address shown.

## Requirements
- Node 20+ on the machine running Cowork (for `npx mcp-remote`).
- Your Cloudgate API + the `/mcp` endpoint reachable at the configured URL.

## Install
Install the `.plugin` from Cowork's **Plugins** settings page. After install, start a
**new** conversation and ask "List my Cloudgate projects" — it should call `list_projects`.

## Alternative: direct remote URL
If your environment provisions remote MCP servers (admin `managedMcpServers` or an org
plugin), you can replace the `.mcp.json` server entry with a direct remote form:
```json
{ "mcpServers": { "cloudgate": {
  "type": "http",
  "url": "https://your-host/mcp/workflow",
  "headers": { "Authorization": "Bearer YOUR_TOKEN" }
} } }
```

## Token rotation
The token expires (per your LIFETIME_DAYS). Re-mint, update `.mcp.json`, re-package, reinstall.
