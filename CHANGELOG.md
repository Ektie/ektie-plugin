# Changelog

## 2.0.0

- MCP server now exposes direct tools (CRM, research and fit, email, sequences, forms, social posts, Google / Meta / Reddit ads, Creative Studio) instead of worker tools; results come back in full, with no job polling except Creative Studio.
- New `ektie-operator` skill: tool routing, call order, checks and approvals (`denied`, `needs_approval`, `human_override`), and best practices per area. Replaces `ektie-orchestrator`.
- Tool visibility follows existing Ektie role permissions; the separate MCP access groups are gone.

## Unreleased (pre-2.0)

- Marketplace copy leads with the customer problem (distribution) and hired GTM team, not the operator/orchestrator hook.

## 1.0.0

- Initial multi-host Ektie MCP + skills plugin (Claude Code, Cursor, Codex, portable skills).
- Remote HTTP MCP at `https://app.ektie.io/mcp` (hub OAuth).
- Orchestrator skill for named GTM worker tools + `agent_tool_job_status` polling.
