# Creative Studio

Creative Studio produces short videos (Reels, UGC, video ads) and paid ad stills. Generation runs in the background and costs credits, so brief well and avoid needless regenerations.

## Start a project

`create_studio_project` with a complete `brief_text`: product, audience, offer, proof, the on-screen headline, the call to action and the destination URL. A campaign seed alone is not a brief.

- `type`: `ad` (paid ads) or `ugc` / `social` (organic video).
- `format_key`: for example `ad_image` (paid stills), `video_ad_campaign`, `ugc_video`. Organic feed images and carousels are not Studio projects: make them with `generate_still` and post them with `create_social_post`.
- `run_mode`: `auto` runs every stage; `guided` stops at each gate for the human.
- One project per distinct visual theme. An `ad_image` project produces several variants of one theme.
- Read `brand_kit` first so colours, fonts and voice match.

## Follow progress

- `get_studio_project` shows the current stage, whether a gate is waiting for review (with its options), stage outputs and finished assets. Poll it with pauses; do not hammer it.
- Async tools that return a `job_id` can be checked with `agent_tool_job_status`.

## Guided review

At a gate:

- `select_variant` picks a concept, hooks or presenter by id.
- `edit_stage_output` writes the script or storyboard frames yourself (when you know exactly what you want).
- `revise_stage` regenerates concept, hooks, script, storyboard, cast or story_vision (sound and music plan) from your `feedback` (say what to change and why).
- `approve_stage` continues the pipeline. In guided mode, confirm with the human first.
- `set_video_format` sets aspect ratio and resolution before production stages (9:16 for Reels and Stories, 1:1 or 4:5 for feed, 16:9 for YouTube).

## Fix individual shots

- `list_project_assets` / `get_project_asset` (with a viewing link) to inspect frames, scenes, voice and exports.
- `regenerate_still` (by `frame_id` or `frame_number`) and `regenerate_scene` (by `scene_id` or `frame_number`) with specific feedback: framing, posture, set, lighting, motion.
- Give one clear change per regeneration; vague feedback wastes credits.

## Use the output

- Ad stills and exports attach to ad drafts with `attach_ad_creative` (`studio_stills`, `studio_exports`).
- `generate_still` makes a standalone library image for posts, carousel slides or ads (`aspect_ratio` required unless the project already has one); fetch its URL with `get_project_asset`.
