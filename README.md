# Ektie MCP and Skills Plugin

A [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.com), and [Codex](https://developers.openai.com/codex/cli) plugin that connects your AI assistant to an [Ektie](https://ektie.com) workspace — an AI go-to-market team — via the hosted [MCP](https://modelcontextprotocol.io/) server and an orchestrator skill.

After connect, you are the **orchestrator**. Call named worker tools (`research_worker`, `analytics_worker`, …) with plain-English outcome briefs. Domain micro-tools stay inside workers. The hired team keeps running on heartbeats.

## Installation

### Claude Code

Install from the Claude marketplace (once listed), or load this repo as a local plugin:

```text
/plugin install ektie@<marketplace>
```

The Ektie MCP server is configured from `.mcp.json`. Authenticate on **app.ektie.io**, pick a workspace, and Approve on first use.

You can also add the OAuth connector URL directly: `https://app.ektie.io/mcp`.

### Cursor

Once listed on the [Cursor Marketplace](https://cursor.com/marketplace):

```text
/add-plugin ektie
```

This installs the skill and MCP server together. Authenticate on **app.ektie.io** on first use.

### Codex

Add this repository as a Codex marketplace, then install the plugin:

```sh
codex plugin marketplace add YOUR_ORG/ektie-plugin
codex plugin add ektie@ektie
```

Start a new Codex session. Codex support currently ships the **skills**; wire MCP separately via `https://app.ektie.io/mcp` if your Codex build supports remote HTTP MCP.

### ChatGPT and other MCP clients

Point a custom connector / MCP client at:

```text
https://app.ektie.io/mcp
```

Sign in on **app.ektie.io**, pick a workspace, Approve. No plugin package required for the MCP tools themselves.

### Other agents (skills only)

Install the orchestrator skill with [`npx skills`](https://github.com/vercel-labs/skills#supported-agents):

```bash
npx skills add YOUR_ORG/ektie-plugin -y -a <agent>
```

Examples: `gemini-cli`, `opencode`. Add `-g` to install for your user. This path carries the skill only — configure the MCP URL separately.

## MCP

```json
{
  "mcpServers": {
    "ektie": {
      "type": "http",
      "url": "https://app.ektie.io/mcp"
    }
  }
}
```

Worker tools return `job_id` immediately — poll `agent_tool_job_status` until complete. Do not expect a long-running worker to finish inside a single HTTP round-trip.

## Features

### MCP server

Hosted at `https://app.ektie.io/mcp`:

- **Named GTM workers** — research, analytics, ICP, records, tasks, prospecting, sequences, outreach, ads, content, Creative Studio
- **Protocol** — `ask_clarification`, `agent_tool_job_status`
- **Workspace** — `ai_context`, `brand_kit`
- **Creative Studio guided** — `creative_studio_project`, `approve_stage`, `select_variant`, `revise_*`
- **Resources / prompts** — schema, ICP, brand kit, GTM briefing, and related prompts after auth

Tool visibility follows your workspace role (Settings → Roles → MCP access).

### Skills

- [`ektie-orchestrator`](skills/ektie-orchestrator/SKILL.md) — how to dispatch workers with outcome briefs, poll jobs, and respect ACL / destructive actions

## Usage examples

- “How many contacts have been contacted?”
- “Research our top overdue A-tier accounts”
- “Show what my hired GTM team is working on”
- “Draft Creative Studio project status and anything waiting for my review”

## Auth

| Path | URL |
|---|---|
| OAuth (Claude / ChatGPT / Cursor Marketplace) | `https://app.ektie.io/mcp` |
| Fallback PAT | `https://{workspace}.ektie.com/mcp` + bearer token from Settings → MCP access |

You need an Ektie workspace with a product entitlement and at least one MCP worker group on your role.

Do not put tokens in this repository.

## Local development

This package lives in the Ektie monorepo at `ektie-plugin/` and is excluded from the app Docker image.

```bash
# Cursor local load:
ln -sfn /absolute/path/to/mg-ektie-crm/ektie-plugin ~/.cursor/plugins/local/ektie
```

Reload the window, open Customize, Connect **ektie**, complete OAuth.

## Publish

Push **this folder as the root** of a public GitHub repo (not the Laravel app), then submit each host’s marketplace as needed:

- Cursor: [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)
- Claude: Claude plugin marketplace listing
- Codex: `codex plugin marketplace add …`

Set `repository` in the host manifests once the public URL exists.

## Docs

- [Connect your agent](https://ektie.com/docs/guides/connect-your-agent)
- [Ektie](https://ektie.com)

## License

MIT
