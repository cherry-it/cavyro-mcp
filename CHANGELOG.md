# Changelog

Versions match the `com.cavyro/cavyro` entry in the [MCP Registry](https://registry.modelcontextprotocol.io). The hosted server at `https://ai.cavyro.com/mcp` is updated continuously; a new version here means the registry listing changed.

## 1.1.0 (2026-10-31)

- Project management: projects, folders, task lists, tasks and subtasks, with bulk moves and the project tree in one call.
- Project reports (workload, tasks at risk, throughput, project health, cycle time).
- My Work: what's due, open mentions, Telegram follow-up reminders and assigned deals.
- Coverage grew from about 140 to over 200 operations across 24 resources. Still nine tools.
- Listing description now names projects and tasks.

## 1.0.1 (2026-09-11)

- Listing links this repository (`repository` in `server.json`), as required by the GitHub MCP Registry.

## 1.0.0 (2026-08-30)

- First listing in the MCP Registry as `com.cavyro/cavyro`, verified by DNS on `cavyro.com`.
- Remote: Streamable HTTP at `https://ai.cavyro.com/mcp`, bearer token plus `X-Workspace-Id` header.
- Nine generic tools over a resource and action registry, covering around 90 CRM operations.
