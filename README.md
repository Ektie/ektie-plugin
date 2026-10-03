# Ektie MCP and Skills Plugin

Finding customers is the hard part. [Ektie](https://ektie.com) runs your go-to-market work in one place: CRM, email and sequences, forms, social posts, ads and Creative Studio, with an AI team you can hire to keep working after you close the tab.

This plugin lets you work in Ektie directly from [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.com), [Codex](https://developers.openai.com/codex/cli), ChatGPT, and other agents, with the same checks your team works under.

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

Start a new Codex session after install. The plugin installs the `ektie-operator` skill and the Ektie MCP server (`https://app.ektie.io/mcp`); sign in to your workspace via OAuth on first use.

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

This path installs skills only. Point the agent at `https://app.ektie.io/mcp` separately for tools.

## Requirements

- An Ektie workspace on a plan that includes MCP access
- OAuth on `https://app.ektie.io/mcp` (recommended), or a workspace token from Settings → MCP access for clients that only support bearer tokens
- The tools you see follow your Ektie role's permissions, the same ones the app uses

## Features

### MCP server

Remote server: `https://app.ektie.io/mcp`. Your assistant does the thinking and the writing; each tool does one job directly in Ektie and returns the full result.

| Area | Tools |
|---|---|
| Schema and lookup | `describe_workspace`, `search_records`, `get_record`, `run_report`, `describe_tables`, `query_database`, `list_team` |
| Records and hygiene | `create_records`, `update_records`, `delete_records`, `crm_hygiene_report`, `cleanup_low_fit_contacts` |
| Notes, tasks, lists | `add_note`, `update_note`, `create_task`, `update_task`, `list_lists`, `create_list`, `add_to_list`, `remove_from_list` |
| Signals and intelligence | `add_signal`, `add_intelligence`, `update_intelligence` |
| ICP fit, research, enrichment | `list_icps`, `set_fit_score`, `complete_research`, `complete_enrichment` |
| Email | `list_email_accounts`, `send_email`, `reply_email` |
| Sequences | `list_sequences`, `create_sequence`, `update_sequence_steps`, `set_sequence_status`, `enroll_in_sequence`, `exit_sequence`, `sequence_stats` |
| Forms | `list_forms`, `create_form`, `update_form`, `list_form_submissions` |
| Media and social posts | `upload_media`, `prepare_media_upload`, `complete_media_upload`, `list_content_channels`, `create_social_post`, `update_social_post`, `schedule_social_post` |
| Ads (Google, Meta, Reddit) | `list_ad_campaigns`, `ad_performance`, `ads_gaql_query`, `keyword_ideas`, `ad_options`, `search_meta_targeting`, `search_google_targeting`, `search_reddit_targeting`, `list_ad_audiences`, `estimate_ad_reach`, `create_custom_audience`, `upload_customer_match`, `create_lookalike_audience`, `create_ad_campaign`, `update_ad_campaign`, `attach_ad_creative`, `launch_ad_campaign`, `delete_ad_campaign_draft`, `set_ad_status`, `update_ad_budget`, `update_keywords`, `update_live_targeting`, `update_ad_copy`, `add_ad_group`, `duplicate_ad_campaign` |
| Creative Studio | `create_studio_project`, `get_studio_project`, `advance_studio_project`, `set_video_format`, `approve_stage`, `select_variant`, `edit_stage_output`, `revise_stage`, `list_project_assets`, `get_project_asset`, `regenerate_still`, `regenerate_scene`, `generate_still`, `agent_tool_job_status` |
| Workspace | `ai_context`, `brand_kit` |

### Skills

| Skill | Purpose |
|---|---|
| [`ektie-operator`](skills/ektie-operator/SKILL.md) | Which tool to use when, the order to call them in, how to handle checks and approvals, and best practices for records, research, email, sequences, posts, ads and Creative Studio |

## How to work

1. Start with `describe_workspace` and the matching `list_*` tool so you use real field slugs, option values and ids.
2. Write the content yourself: emails, posts, notes, research, sequence steps, ad copy.
3. Before anything that sends, spends, publishes or deletes, show the human and get a yes.
4. A `denied` result is final; report its message. `needs_approval` means ask the human, then retry with `human_override` only if they agree.

## Usage examples

- "How many contacts have been contacted this month?"
- "Research Acme's VP of Sales, score their ICP fit and add them as a contact."
- "Draft a follow-up to everyone who asked for pricing and show me before sending."
- "Build a Meta lead campaign for our SaaS RevOps ICP with a $30 daily budget."
- "What is waiting for my review in Creative Studio?"

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

- **Permissions.** A tool missing from your tool list means your Ektie role or plan does not include it. Ask a workspace admin.
- **Same rules as the team.** Outreach needs research and enrichment first, and respects opt-outs, sequence membership, cooldowns and contact frequency limits.
- **No prospecting.** Finding new leads is not part of this server; bring your own sources and add results with `create_records`.
- **Integrations.** Email, ads and social tools need the matching account connected in Ektie (Settings → Integrations).
- **Background work.** Only Creative Studio generation runs in the background; poll `get_studio_project`.

## Documentation

- [Connect your agent](https://ektie.com/docs/guides/connect-your-agent)
- [Ektie](https://ektie.com)

## License

MIT
