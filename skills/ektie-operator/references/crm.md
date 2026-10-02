# CRM: records, research and fit

## Schema first

- `describe_workspace` with no arguments lists objects. With `object` (for example `contact`, `company`, `opportunity`, `note`, `task`) it returns every field: slug, type, required, select options (the only valid values) and relation targets.
- Plural or label spellings are accepted for objects ("contacts", "Companies"), but field slugs and option values must match exactly.
- Fields are keyed by **slug** everywhere: `{"email": "ana@acme.io", "status": "lead"}`.

## Reading

- `search_records`: one object, optional `query` (name / title match), `filters` `[{field, op, value}]` combined with `match: all|any`.
  - Ops: `=, !=, >, <, >=, <=, in, not_in, contains, not_contains, between, not_between, is_null, is_not_null`. Use `values` for `in` / `between`.
  - `related_to_record_id` lists notes, tasks or deals attached to a record.
  - Ask only for the `fields` you need; page with `page` / `limit` (max 50).
- `get_record` with `include` (`notes, tasks, activity, emails, signals, intelligence, sequences, readiness`) for full context. Use `readiness` before any outreach to see what is missing.
- `run_report` for counts, sums and averages grouped by fields (deals by stage, contacts created per month). Prefer it over SQL.
- `query_database` for questions a report cannot answer. Call `describe_tables` first. Records are stored as rows in `records` with values in `field_values` (get field ids from `describe_workspace`). Write one `SELECT`, select explicit columns, aggregate, and stay under 100 rows. Every table is automatically limited to this workspace.
- `list_team` for user ids (assignees, user fields) and hired AI agent ids.

## Creating and updating

- `create_records`: one object, 1 to 50 records, each `{fields, fit?, parent_record_id?, human_override?}`.
  - Required fields must be present; select fields take listed option values; emails must have a real mail domain.
  - Possible duplicates (same email, website, LinkedIn or name) come back under `existing` with their ids. Do not create them again.
- `update_records`: send only the fields that change; `null` clears a field.
- `delete_records`: permanent. Show the human which records and get a yes, then call with `confirm: true`.

## ICP fit (you decide it)

Contacts and companies need a fit assessment, which you make from your own research.

1. `list_icps` returns every ICP with its criteria and the workspace **fit threshold**.
2. For each record, score every ICP you assessed from 0 to 1 with a reason that cites evidence, and list `criteria_matched` (for example `"industry: SaaS"`, `"title: VP Revenue Operations"`). Score 0 for ICPs that clearly do not match. Set `confidence` to how complete your evidence is.
3. Send it as the `fit` block in `create_records`, or with `set_fit_score` for an existing record.

```json
{
  "matches": [
    {"icp_profile_id": 12, "score": 0.82, "reason": "VP RevOps at a 220-person SaaS company in Austin", "criteria_matched": ["industry: SaaS", "title: VP Revenue Operations", "size: 200-500"]},
    {"icp_profile_id": 14, "score": 0.0, "reason": "Not an agency"}
  ],
  "confidence": 0.75,
  "reason": "Strong match for the SaaS RevOps ICP"
}
```

- Below the threshold, the record is **not created**; it comes back under `needs_approval` with the score and threshold. Tell the human; only if they still want it, resend that record with `human_override`.
- Your assessment drives which sequences the contact can join (ICP checks) and outreach readiness, so be honest. A generous score to get past a gate is a mistake.

## Research and enrichment (required before outreach)

The same readiness rules as the hired AI team apply: a contact must be researched and enriched before any email or sequence.

- `complete_research`: `summary` (who they are, why now), 1 to 12 `intelligence` entries (category, title, content, importance 1 to 10), and the `sources` URLs you used. At least one insight and one source.
- `complete_enrichment`: details you found (`fields` by slug) and the `email` with how you know it:
  - `status: valid` needs a `source` plus an `evidence_url` where the address appears, or `source: user_provided` when the human gave it to you.
  - Otherwise use `status: unverified`. Sending to an unverified email needs the human's approval later.
  - Never mark a guessed pattern (first.last@) as valid.
- `get_record` with `include: ["readiness"]` shows what is still missing.

## Notes, tasks, lists

- `add_note` / `update_note`: write a clear title and a body a teammate can act on. Delete a note with `delete_records`.
- `create_task`: a CRM task for a person (`assignee_user_id`) or work for an AI agent (`hired_agent_id`, it lands on that agent's board). `status` / `priority` come from `describe_workspace object=task`. `update_task` to complete or reschedule.
- `list_lists`, `create_list` (object, name, column field slugs), `add_to_list` / `remove_from_list` (up to 200 ids, records must be of the list's object).

## Signals and intelligence

- `add_signal`: a dated event with a source URL (funding, hiring, leadership change, launch, review, post). Pick the `platform` and, if you can judge intent, `classification` hot / warm / cold. The same URL is stored once (`duplicate` result).
- `add_intelligence`: one insight per entry, grounded in facts. Categories: `pain_points, qualification, relationship, context, risk, objections, preferences` for facts; `timing_hypothesis, structural_hypothesis, competitive_hypothesis, role_fit_hypothesis` for reasoned guesses.
- `update_intelligence` replaces an entry and keeps the old one as history. Use it instead of adding a contradicting duplicate.

## Hygiene

1. `crm_hygiene_report` (`check: duplicates | missing | orphans | all`).
2. Show the human what you would merge, fill or delete.
3. Fix with `update_records` / `delete_records`.
4. `cleanup_low_fit_contacts` lists contacts below the fit threshold (dry run by default; never customers or contacts with open deals). Delete only with `dry_run: false` and `confirm: true` after approval.
