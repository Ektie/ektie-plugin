# Gates and errors

## The result shape

Checks that stop an action return:

```json
{
  "status": "denied",
  "reason": "sequence_motion_lock",
  "message": "This contact left a sequence 3 days ago and can join another after 2026-10-09.",
  "details": {"available_after": "2026-10-09"},
  "fix": ["Wait until 2026-10-09"],
  "overridable": false
}
```

- `status: denied`, `overridable: false`: final. Tell the human `message`, offer `fix`. Do not retry, and do not reach the same result through a different tool.
- `status: needs_approval`, `overridable: true`: the human decides. Explain the reason and the numbers (score vs threshold, how the email was found). Only if they agree, call the same tool again with `human_override: {approved: true, note: "<their words>"}`. The approval is logged on the record.

Batch tools (`create_records`, `enroll_in_sequence`) return successes and per-record skips together; report both.

## Reasons you will see

### Required steps (you can fix them)

| Reason | Fix |
|---|---|
| `contact_never_researched`, `contact_research_pending`, `contact_research_incomplete`, `contact_research_failed` | Research the contact, then `complete_research` |
| `contact_never_enriched`, `contact_enrichment_pending`, `contact_enrichment_incomplete`, `contact_enrichment_failed` | Find the details, then `complete_enrichment` |
| `fit_required` | Add a `fit` block (`list_icps`) |

### Human approval (`needs_approval`)

| Reason | Meaning |
|---|---|
| `below_fit_threshold` | New record scores under the fit threshold; not created |
| `contact_low_fit`, `low_fit_score` | Contact fit is low for this outreach |
| `contact_not_verified` | Email is unverified |

### Final (`denied`)

| Reason | Meaning / what to say |
|---|---|
| `contact_opted_out` | Unsubscribed or suppressed. Never contact them. |
| bounced / customer / deal closed | Not an outreach target. |
| `already_in_sequence` | In an active sequence; it handles their emails. Exit first only if the human wants to. |
| `sequence_motion_lock` | Recently left a sequence; eligible after `available_after`. |
| `fatigue_cap` | Contacted too often recently. |
| pending scheduled send / `duplicate_content` | An email is already queued or the same content was just sent. |
| `timezone_mismatch`, `icp_mismatch`, `icp_unclassified` | Contact's region or ICP does not match the sequence. Pick another sequence or fix the fit. |
| `merge_tag_leak`, `placeholder_leak`, `subject_empty`, `body_too_short`, `tracking_reference` | Email content problem. Rewrite and resend. |
| `invalid_timezone`, `duplicate_blocked`, `archived_not_gap`, `learning_patience`, `underperforming_fix_first`, `inventory_cap`, `create_cooldown`, `portfolio_unhealthy`, `icp_inactive` | Sequence creation rule. Usually: use the existing sequence, or wait. |
| `schedule_overlap` | Another post goes out on that platform within 30 minutes. Pick another time. |
| `invalid_targeting` | Ad targeting value is not real. Use one of `details.unresolved[].candidates`. |
| `already_launched`, `not_ready` | Ad draft state. `not_ready` lists what is missing. |

## Error codes

| Code | What to do |
|---|---|
| `unknown_option`, `unknown_object`, `unknown_field` | Use the real values in `details` (from `describe_workspace` / `list_*`). |
| `invalid_input` | Read `message`, fix the arguments. |
| `not_found` | Re-look the id up (`search_records`, `list_*`). |
| `forbidden` | The human's role cannot do this. Say so. |
| `confirmation_required` | Ask the human, then call again with `confirm: true`. |
| `integration_disconnected`, `integration_incomplete` | The mailbox, ad account or channel is not connected. The human connects it in Settings → Integrations. |
| `plan_required` | Their Ektie plan does not include this. |
| `not_live`, `not_a_draft` | Use the draft tools for drafts and the live tools for launched campaigns. |
| `launch_failed` | The ad platform rejected the campaign; `details.errors` says why. Fix the draft and try again. |
