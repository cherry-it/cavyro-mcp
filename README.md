# Cavyro MCP server

[![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%2Fcom.cavyro%252Fcavyro%2Fversions%2Flatest&query=%24.server.version&label=MCP%20Registry&prefix=v&color=5e6ad2)](https://registry.modelcontextprotocol.io/v0/servers?search=com.cavyro)
[![Server status](https://img.shields.io/website?url=https%3A%2F%2Fai.cavyro.com%2F.well-known%2Fmcp%2Fserver-card.json&label=server&up_message=online&down_message=offline)](https://ai.cavyro.com/.well-known/mcp/server-card.json)
[![Tools](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fai.cavyro.com%2F.well-known%2Fmcp%2Fserver-card.json&query=%24.tools.length&label=tools&color=5e6ad2)](https://ai.cavyro.com/.well-known/mcp/server-card.json)
[![Transport](https://img.shields.io/badge/transport-streamable%20HTTP-informational)](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports#streamable-http)
[![Smithery](https://img.shields.io/badge/Smithery-cavyro%2Fcavyro-orange)](https://smithery.ai/servers/cavyro/cavyro)
[![Release](https://img.shields.io/github/v/release/cherry-it/cavyro-mcp?color=5e6ad2)](https://github.com/cherry-it/cavyro-mcp/releases)

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=cavyro&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fai.cavyro.com%2Fmcp%22%2C%22headers%22%3A%7B%22Authorization%22%3A%22Bearer%20%24%7Binput%3Acavyro-token%7D%22%2C%22X-Workspace-Id%22%3A%22%24%7Binput%3Acavyro-workspace%7D%22%7D%7D&inputs=%5B%7B%22id%22%3A%22cavyro-token%22%2C%22type%22%3A%22promptString%22%2C%22description%22%3A%22Cavyro%20API%20token%22%2C%22password%22%3Atrue%7D%2C%7B%22id%22%3A%22cavyro-workspace%22%2C%22type%22%3A%22promptString%22%2C%22description%22%3A%22Cavyro%20workspace%20ID%22%7D%5D)
[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=cavyro&config=eyJ1cmwiOiJodHRwczovL2FpLmNhdnlyby5jb20vbWNwIiwiaGVhZGVycyI6eyJBdXRob3JpemF0aW9uIjoiQmVhcmVyIDx5b3VyLWFwaS10b2tlbj4iLCJYLVdvcmtzcGFjZS1JZCI6Ijx5b3VyLXdvcmtzcGFjZS1pZD4ifX0=)

Cavyro is an agentic CRM for teams that sell in chat: one inbox for WhatsApp, Instagram, LinkedIn, Messenger and Telegram, with contacts, companies, deals, pipelines, projects and tasks, campaigns, docs and reports behind it. This repository describes Cavyro's hosted [Model Context Protocol](https://modelcontextprotocol.io) server, listed in the [MCP Registry](https://registry.modelcontextprotocol.io) as `com.cavyro/cavyro`.

The server is hosted, so there is nothing to install or run. Point your MCP client at it and it reads and acts on your Cavyro workspace with your permissions.

- **Endpoint:** `https://ai.cavyro.com/mcp` (Streamable HTTP)
- **Server card:** [`https://ai.cavyro.com/.well-known/mcp/server-card.json`](https://ai.cavyro.com/.well-known/mcp/server-card.json) (full tool surface with input schemas, no credentials needed)
- **Registry manifest:** [`server.json`](server.json) · **Changes:** [`CHANGELOG.md`](CHANGELOG.md)
- **Website:** [cavyro.com](https://cavyro.com) · **Help:** [help.cavyro.com/ai/mcp](https://help.cavyro.com/ai/mcp/)

The server source is not published here. The tool surface is generated from the running server and exposed in the server card above, so the card is always the authoritative description of what the server does.

## What it does

The server exposes a fixed set of nine generic tools over a resource and action registry, instead of one tool per API operation. The client discovers resources and actions on demand, so the tool list stays small while covering over 200 operations across 24 resources.

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

**Covered domains:** deals, contacts, companies, pipelines and stages, projects with folders, task lists, tasks and subtasks, project reports, comments and reactions, activity feed, notifications, a personal My Work queue with mentions, workspaces, members, teams, groups, invitations, custom field definitions, automations, docs with their pages, reports, Google Calendar meetings, inbox chats across WhatsApp, Instagram, LinkedIn, Messenger and Telegram, and Telegram follow-up reminders.

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

The **Add to Cursor** button above installs the server with placeholder headers; replace them in `~/.cursor/mcp.json`.

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
- "What's on my plate today? Then move every overdue task in the Website project to In progress."

The assistant will typically call `describe_resource` first to learn the shape of a resource, then `query_resources` or `perform_action`. If it needs a multi-step recipe, it calls `load_guidance`.

## Support

- Help center: [help.cavyro.com](https://help.cavyro.com)
- Email: [support@cavyro.com](mailto:support@cavyro.com)
