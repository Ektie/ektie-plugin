# Ads: Google, Meta and Reddit

Ad platforms only accept their own ids and enum values. A draft with made-up targeting would not show in the Ads builder and would not launch, so Ektie rejects it. Look values up first, every time.

## The flow

1. **Options:** `ad_options` for the platform: objectives, optimization goals, bid strategies, CTAs, placements, Google campaign types / match types / language ids, RSA limits, and for Reddit the account's real pixels and funding instruments.
2. **Targeting lookups:**
   - Meta: `search_meta_targeting` (`interests, job_titles, industries, behaviors, employers, demographics, geo`). Job titles go in `work_positions`, employers in `work_employers`, each item `{id, name}` exactly as returned.
   - Google: `search_google_targeting` (`geo` for location ids, `user_interests`, `audience_segments`), `keyword_ideas` for keywords with volume and bids.
   - Reddit: `search_reddit_targeting` (`communities, interests, geo, keywords`). Put each result's `value` in the matching list. Keywords may be your own words.
   - Audiences: `list_ad_audiences` for existing custom, saved and lookalike audiences.
3. **Size check (Meta):** `estimate_ad_reach` before building the ad set.
4. **Draft:** `create_ad_campaign` (nothing is spent). The result lists what is still missing before launch. Fix with `update_ad_campaign` and `attach_ad_creative`.
5. **Launch:** show the human platform, budget, audience and ads; on a yes, `launch_ad_campaign` with `confirm: true`. Campaigns are created **paused**.
6. **Go live:** only when the human says so, `set_ad_status` with `status: enabled` (this starts spend).

If a value is rejected, the result includes `unresolved` with real `candidates`. Use one of them.

## Building good drafts

- Tie the campaign to an ICP (`icp_profile_id`) and write copy for that buyer's problem.
- **Google Search:** tight ad groups by theme; each ad group owns its keywords and responsive search ads (3 to 15 headlines of max 30 characters, 2 to 4 descriptions of max 90). Start keywords on PHRASE or EXACT; add negatives for job seekers, free, DIY. Over-limit copy is rejected: rewrite shorter, do not truncate mid-word.
- **Meta:** one campaign, ad sets that own their targeting and their ads (`ad_sets[].ads[]`); do not clone one creative across every ad set. Leads objectives need a Facebook `page_id`. `advantage_audience: false` keeps your targeting as hard filters.
- **Reddit:** pick communities where the buyer actually talks; text ads need headline and body, image / video / carousel ads also need media and a `final_url`. The pixel and funding instrument default to the integration settings; any you pass must come from `ad_options`.
- **Creative:** use `upload_media` / `complete_media_upload` ids or Creative Studio outputs (`attach_ad_creative` with `studio_stills` / `studio_exports`). For a local image, `prepare_media_upload`, PUT the raw file, then `complete_media_upload`. Do not paste the image into the tool call as base64. For new ad images use Creative Studio `format_key: ad_image`, one project per visual theme.
- Budgets are daily, in the ad account currency. Start small and say so to the human.

## Live campaigns

| Change | Tool |
|---|---|
| Pause / enable / remove a campaign, ad group, ad set, keyword or ad | `set_ad_status` (`level` + the platform id field) |
| Daily budget, bidding strategy, target CPA | `update_ad_budget` (Reddit: per ad group) |
| Google keywords, bids, negatives | `update_keywords` |
| Locations, languages, audiences, demographics, schedule, devices, placements | `update_live_targeting` (lists replace the current ones) |
| New or edited ads | `update_ad_copy` |
| New ad group / ad set | `add_ad_group` |
| Copy a Meta campaign or ad set (paused) | `duplicate_ad_campaign` |

- Budget, enable and remove changes affect real spend: confirm every one with the human.
- Reddit ads cannot be edited after creation except name and click URL; create a new ad instead. Reddit campaigns cannot be removed here; pause them.

## Reading performance

- `list_ad_campaigns` for ids, status and lifetime totals.
- `ad_performance` for spend, clicks, CTR, conversions and cost per result by campaign or ad group / ad set over a date range.
- `ads_gaql_query` (Google, read-only) for search terms, assets and auction insights that `ad_performance` does not cover.
- Judge on enough data: a few days and a few hundred clicks before calling a winner. Recommend one change at a time.
