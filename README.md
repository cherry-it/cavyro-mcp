# Cavyro MCP server

Cavyro is a Telegram-native CRM for agents and small sales teams: contacts, companies, deals, pipelines, campaigns, docs and reports. This repository describes Cavyro's hosted [Model Context Protocol](https://modelcontextprotocol.io) server, listed in the [MCP Registry](https://registry.modelcontextprotocol.io) as `com.cavyro/cavyro`.

The server is hosted, so there is nothing to install or run. Point your MCP client at it and it reads and acts on your Cavyro workspace with your permissions.

- **Endpoint:** `https://ai.cavyro.com/mcp` (Streamable HTTP)
- **Server card:** [`https://ai.cavyro.com/.well-known/mcp/server-card.json`](https://ai.cavyro.com/.well-known/mcp/server-card.json) (full tool surface with input schemas, no credentials needed)
- **Website:** [cavyro.com](https://cavyro.com) · **Help:** [help.cavyro.com/ai/mcp](https://help.cavyro.com/ai/mcp/)

The server source is not published here. The tool surface is generated from the running server and exposed in the server card above, so the card is always the authoritative description of what the server does.

## What it does

The server exposes a fixed set of nine generic tools over a resource and action registry, instead of one tool per API operation. The client discovers resources and actions on demand, so the tool list stays small while covering around 90 CRM operations.

| Tool | Purpose |
| --- | --- |
| `describe_resource` | Catalog of all resources and global actions, or one resource's filters, sort fields, enums, operations and actions |
| `query_resources` | List records of any resource with filters, sort, pagination and custom-field filters |
| `get_resource` | Fetch one record by ID |
| `create_resource` | Create a record (custom fields nested under `custom_fields`) |
| `update_resource` | Change fields on a record |
| `delete_resource` | `discard` (archive), `restore`, or `destroy` a record |
| `search` | Omnibox search across entity types |
| `perform_action` | Any specialized non-CRUD action by slug: move a deal between stages, link records, bulk operations, comments, reports |
| `load_guidance` | On-demand workflow guidance: create flows, custom fields, automations |

**Covered domains:** deals, contacts, companies, pipelines and stages, comments and reactions, activity feed, notifications, workspaces, members, teams, groups, invitations, custom field definitions, automations, docs with their pages, reports, and read-only Telegram chats.

All tool responses share one envelope, `{ "data": ..., "pagination": ... }`, with `pagination` present only on paginated list results.

The connection acts **as you**. An external AI can only see and do what your Cavyro role allows in that workspace.

## Authentication

Every request needs two headers:

| Header | Value |
| --- | --- |
| `Authorization` | `Bearer <cavyro_api_token>` |
| `X-Workspace-Id` | Your workspace ID |

Create an API token in Cavyro under **Settings → API tokens**. Tokens can expire after 7, 30 or 90 days, or never for integrations that cannot rotate them. Revoking a token cuts the client off immediately.

There is no OAuth flow. The server answers unauthenticated requests with `401` and a `WWW-Authenticate: Bearer realm="cavyro"` challenge. MCP is a Pro feature; MCP calls do not consume Cavyro AI credits, the model tokens are paid by your own client subscription.

## Configuration

Replace `<your-api-token>` and `<your-workspace-id>` in the snippets below. Cavyro's settings page generates the same snippet with your workspace ID filled in.

### VS Code (GitHub Copilot)

`.vscode/mcp.json`, or **MCP: Add Server** from the command palette:

```json
{
  "servers": {
    "cavyro": {
      "type": "http",
      "url": "https://ai.cavyro.com/mcp",
      "headers": {
        "Authorization": "Bearer ${input:cavyro-token}",
        "X-Workspace-Id": "${input:cavyro-workspace}"
      }
    }
  },
  "inputs": [
    {
      "id": "cavyro-token",
      "type": "promptString",
      "description": "Cavyro API token",
      "password": true
    },
    {
      "id": "cavyro-workspace",
      "type": "promptString",
      "description": "Cavyro workspace ID"
    }
  ]
}
```

### Cursor and Claude Desktop

Cursor: `~/.cursor/mcp.json`. Claude Desktop: `~/Library/Application Support/Claude/claude_desktop_config.json`.

```json
{
  "mcpServers": {
    "cavyro": {
      "url": "https://ai.cavyro.com/mcp",
      "headers": {
        "Authorization": "Bearer <your-api-token>",
        "X-Workspace-Id": "<your-workspace-id>"
      }
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http cavyro https://ai.cavyro.com/mcp \
  --header "Authorization: Bearer <your-api-token>" \
  --header "X-Workspace-Id: <your-workspace-id>"
```

### ChatGPT

Add the server as a connector in Developer Mode (Plus, Pro, Business or Enterprise) with the URL `https://ai.cavyro.com/mcp` and the two headers above.

## Usage

Once connected, ask your assistant in plain language. A few things that work well:

- "Show my open deals sorted by value, then move the top one to Negotiation."
- "Create a contact for Ana Petrović at Acme, phone +381…, and link her to the Acme company."
- "Which deals in the Enterprise pipeline haven't had activity in 30 days?"
- "Summarize the last week of the Telegram chat with Acme and add it as a comment on their deal."
- "Add a custom field 'Lead source' to contacts and set it to 'Referral' on the contacts I just created."

The assistant will typically call `describe_resource` first to learn the shape of a resource, then `query_resources` or `perform_action`. If it needs a multi-step recipe, it calls `load_guidance`.

## Support

- Help center: [help.cavyro.com](https://help.cavyro.com)
- Email: [support@cavyro.com](mailto:support@cavyro.com)
