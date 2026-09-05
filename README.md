# Ektie MCP and Skills Plugin

Connect [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.com), [Codex](https://developers.openai.com/codex/cli), ChatGPT, and other agents to an [Ektie](https://ektie.com) workspace.

Ektie is an AI go-to-market team. This plugin wires your assistant to the hosted [MCP](https://modelcontextprotocol.io/) server and an orchestrator skill so you can steer that team when you ask — without replacing how it already runs on its own.

**You are the orchestrator.** Call named worker tools with plain-English outcome briefs. Domain micro-tools stay inside workers. The hired team keeps running on heartbeats.

## Installation

### Claude Code

```text
/plugin install ektie@claude-plugins-official
```

Use once the plugin is on the Claude marketplace. The MCP server is configured automatically; authenticate to your Ektie workspace via OAuth on first use.

Until then, add a custom connector pointed at `https://app.ektie.io/mcp`, or load this repository as a local Claude plugin.

### Cursor

```text
/add-plugin ektie
```

Use once the plugin is on the [Cursor Marketplace](https://cursor.com/marketplace). This installs the skill and MCP server together; authenticate via OAuth on first use.

### Codex

```sh
codex plugin marketplace add <org>/ektie-plugin
codex plugin add ektie@ektie
```

Start a new Codex session after install. Codex currently ships the **skills** from this repo; configure MCP separately with `https://app.ektie.io/mcp` if your client supports remote HTTP MCP.

### ChatGPT and other MCP clients

No plugin package required. Add a custom connector / MCP server:

```text
https://app.ektie.io/mcp
```

Sign in on **app.ektie.io**, pick a workspace, Approve.

### Other agents (skills only)

```bash
npx skills add <org>/ektie-plugin -y -a <agent>
```

Examples: `gemini-cli`, `opencode`. Drop `-y` to pick skills interactively; add `-g` for a user-level install. See [supported agents](https://github.com/vercel-labs/skills#supported-agents).

This path installs skills only — point the agent at `https://app.ektie.io/mcp` separately for tools.

## Requirements

- An Ektie workspace with a product entitlement
- Your role has at least one MCP worker group enabled (Settings → Roles → MCP access)
- OAuth on `https://app.ektie.io/mcp` (recommended), or a workspace PAT from Settings → MCP access for clients that only support bearer tokens

## Features

### MCP server

Remote server: `https://app.ektie.io/mcp`

| Surface | Tools |
|---|---|
| GTM workers | `research_worker`, `analytics_worker`, `marketing_analytics_worker`, `icp_worker`, `record_worker`, `task_worker`, `intelligence_worker`, `prospect_discovery_worker`, `prospect_verify_worker`, `sequence_worker`, `outreach_worker`, `linkedin_worker`, `meeting_worker`, `listener_worker`, `content_ops_worker`, `ad_ops_worker`, `creative_worker` |
| Protocol | `ask_clarification`, `agent_tool_job_status` |
| Workspace | `ai_context`, `brand_kit` |
| Creative Studio | `creative_studio_project`, `approve_stage`, `select_variant`, `revise_script`, `revise_storyboard`, `revise_cast`, `revise_concept`, `revise_hooks` |

Workers return `job_id` immediately and run asynchronously. Poll `agent_tool_job_status` until the job finishes. Tool visibility is role-scoped.

The server also exposes MCP resources (schema, ICP, brand kit, …) and prompts (GTM briefing, prospect research, …) after you authenticate.

### Skills

| Skill | Purpose |
|---|---|
| [`ektie-orchestrator`](skills/ektie-orchestrator/SKILL.md) | How to call workers with outcome briefs, poll jobs, clarify ambiguity, and avoid destructive actions without an explicit ask |

## How to work

1. Call the matching **worker tool** with `{ instruction }` — an outcome, not a list of inner tools or steps.
2. For multi-part asks, call multiple workers; do not stop after a partial answer.
3. If the request is ambiguous, call `ask_clarification` first.
4. When a worker returns `job_id`, poll `agent_tool_job_status` until `success` or `failed`, then use the summary.

| Ask | Worker |
|---|---|
| Contact counts / contacted / outbound activity | `analytics_worker` |
| Research a contact or company | `research_worker` |
| Create or update CRM records | `record_worker` |
| ICP / product fit | `icp_worker` |
| Find or verify prospects | `prospect_discovery_worker` / `prospect_verify_worker` |
| Sequences | `sequence_worker` |
| Send outreach (only when clearly asked) | `outreach_worker` |
| Hired-team tasks | `task_worker` |
| Ads | `ad_ops_worker` |
| Creative Studio production | `creative_worker` (+ guided `creative_studio_*` / `revise_*` tools) |

Do **not** invent micro-tools such as `crm_lookup`, `contact_status`, or `sequence_analytics` — those run inside workers.

## Usage examples

- “How many contacts have been contacted?”
- “Research our top overdue A-tier accounts”
- “Show what my hired GTM team is working on”
- “What’s waiting for my review in Creative Studio?”

## MCP config (manual)

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

Bearer-token fallback (Settings → MCP access):

```json
{
  "mcpServers": {
    "ektie": {
      "url": "https://YOUR_WORKSPACE.ektie.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

Never commit tokens to this repository.

## Limitations

- **Role ACL.** Missing workers on `tools/list` means your role cannot use them — ask a workspace admin to enable the MCP group.
- **Async workers.** Connector timeouts are avoided by returning `job_id` immediately; always poll status instead of expecting a full worker run in one HTTP round-trip.
- **Destructive actions.** Outreach, LinkedIn, meetings, and ads workers can take irreversible external actions; only use them when the human explicitly asks.
- **Codex.** Skills install from this repo; MCP must be configured separately until Codex wires remote MCP in the plugin surface.

## Documentation

- [Connect your agent](https://ektie.com/docs/guides/connect-your-agent)
- [Ektie](https://ektie.com)

## License

MIT
