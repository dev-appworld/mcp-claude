# Cloudgate App Templates — Claude Plugin Marketplace

A Claude plugin marketplace for building and managing **Cloudgate** workflow-APIs
directly from Claude (Cowork, Desktop chat, and web).

## Plugins

| Plugin | Description |
| --- | --- |
| **cloudgate-builder** | Connects Claude to the Cloudgate MCP (`api.cloudgate.dev`) over OAuth and adds the `cloudgate-build` skill for creating projects, endpoints, workflow graphs, databases, and more. |

## Install

### Claude (Cowork / Desktop / Web)

1. Open **Customize → Plugins**.
2. In **Personal plugins**, click **+ → Add marketplace**.
3. Choose **Add from a repository** and enter this repo:
   `https://github.com/<your-org>/cloudgate-app-templates`
4. Install **cloudgate-builder** from the marketplace.
5. Start a chat and ask to list your Cloudgate projects — you'll be sent to
   `hub.cloudgate.dev` to sign in, then the tools become available.

### Claude Code

```bash
claude plugin marketplace add <your-org>/cloudgate-app-templates
claude plugin install cloudgate-builder@cloudgate-app-templates
```

## Authentication

`cloudgate-builder` uses OAuth 2.1 (authorization code + PKCE) against the Cloudgate
authorization server. Each user signs in as themselves; every MCP call is scoped to
that user's tenant and account. No tokens or secrets are stored in this repository.

## Repository layout

```
.claude-plugin/marketplace.json     # marketplace manifest (lists the plugins)
plugins/cloudgate-builder/          # the plugin
  .claude-plugin/plugin.json        # plugin manifest
  .mcp.json                         # MCP server connection (native remote HTTP)
  README.md                         # plugin readme
  skills/cloudgate-build/SKILL.md   # the cloudgate-build skill
```

## License

© Cloudgate Dev LLC.
