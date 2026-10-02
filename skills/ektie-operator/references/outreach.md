# Outreach: email and sequences

Outreach follows the same rules the hired AI team works under. Expect checks, and relay them honestly.

## Before you write a single email

1. `get_record` with `include: ["readiness", "emails", "sequences", "intelligence"]`.
2. If research or enrichment is missing, do it (`complete_research`, `complete_enrichment`). Ektie will not do it for you on this path.
3. Check the contact is not already in an active sequence (`sequences` in the record). A contact in a sequence gets that sequence's emails, not ad hoc ones.
4. `list_email_accounts` to pick the sending inbox (default: the human's own).

## send_email

- Write the final subject and body with real values. The sender's signature is added automatically, so do not sign off with a name block.
- Show the human the exact email and wait for a yes. Then send. Use `send_at` (ISO datetime) to schedule.
- Blocking content issues come back as errors: empty subject, body too short, leftover merge tags or placeholders, tracking references. Rewrite and resend.
- Warnings (for example another teammate owns the contact) do not block; tell the human.
- `reply_email` answers an inbound email (`email_id` from `get_record include=emails`) from the inbox that received it.

### Writing emails that get replies

- One person, one reason, one ask. Lead with something specific to them (a signal, their role's problem), not with the sender's company.
- Short: 50 to 120 words for a first touch. Plain text reads as personal.
- One clear, low-effort call to action ("Worth a 15 minute call next week?").
- Subject: 2 to 6 words, specific, lower friction (no hype, no ALL CAPS, no fake "Re:").
- Never invent facts, numbers, customers or quotes. Use what research found.

## Sequences

### Create

- `list_sequences` first. One sequence per ICP + type + region is allowed. If one exists, use it.
- `create_sequence` needs `icp_profile_id` and a regional IANA `timezone` (for example `America/New_York`, `Europe/London`; not UTC). Contacts enroll only if their country is in that region.
- Name it after persona and purpose ("SaaS RevOps leaders, US outbound").
- Steps: `email`, `manual_email`, `phone_call`, `action_item`, `mark_goal`, LinkedIn steps. `delay_days` is days after the previous step (0 for the first). Email steps may use `{{first_name}}`-style tags from the contact; A/B with `variants`.
- A good outbound shape: 4 to 6 steps over about 3 weeks; each step adds a new angle or proof point, not "just bumping this".
- Creation can be denied by portfolio rules (duplicate for this ICP and region, a sibling still learning or underperforming, too many sequences, a short cooldown between creations). The denial says what to do instead, usually "use the existing sequence".
- Blocking step issues come back for you to fix; nothing is saved until they pass.

### Edit and activate

- `update_sequence_steps` replaces the full ordered list (include `id` to keep a step). Active sequences are locked: `set_sequence_status` to `draft`, edit, then activate again.
- `set_sequence_status: active` starts sending to enrolled contacts. Confirm with the human first. `archived` exits everyone.

### Enroll

- `enroll_in_sequence` with up to 100 `record_ids`. Each contact is checked: researched and enriched, verified email, matching ICP and region, not a customer, not unsubscribed or bounced, not already in a sequence, not inside the cooldown after leaving one, not over the contact-frequency cap.
- The result lists `enrolled` and `skipped`, each skip with `reason`, `message`, `fix` and details such as `available_after`.
- Skips marked `needs_approval` (unverified email, low fit) can be enrolled only after the human agrees: resend those ids with `human_override: {approved: true, record_ids: [...], note}`.
- Hard skips (already in a sequence, cooldown, opted out, customer) cannot be overridden. Report them with the date they become eligible when given.

### Exit and measure

- `exit_sequence` stops steps for some contacts (give a `reason`). Leaving starts a cooldown before they can join another sequence.
- `sequence_stats` for enrollment states, per-step send / open / click rates and outcomes. Use it before changing copy.
