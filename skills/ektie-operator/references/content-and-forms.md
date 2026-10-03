# Social posts, media and forms

## Media

- `upload_media` stores a public `https` URL (or an icon under 100 KB as a `data:` URL). It returns `media_id` and a public `url`.
- A local file uses `prepare_media_upload` (`content_type`, exact `size`), then a raw PUT of the bytes to `upload_url` with the returned headers and no Authorization header, then `complete_media_upload`. Do not base64 an ad creative into `upload_media`.
- Internal or private addresses are refused. JPEG, PNG, GIF, WebP, and PDF only.

## Social posts

1. `list_content_channels` for the connected channels and their `channel_id`.
2. Write the finished post yourself and `create_social_post`. It lands in **Review**.
3. Show the human the post. When they approve the copy and the time, `schedule_social_post` with an ISO datetime including the timezone offset.

### Formats

| `post_format` | Needs |
|---|---|
| `text` | `content` |
| `text_image` | `image_url` |
| `text_link` | `link` to the human's own site |
| `text_image_link` | `image_url` and `link` |
| `carousel` | 2 or more `carousel_images` in order; LinkedIn also needs `carousel_document_url` (a PDF of the slides) |

- Reddit channels: `text` or `text_link` only. Instagram: needs an image or a carousel.
- Scheduling runs the same checks as approving in the app (length, hashtags, link, images) and refuses a second post on the same platform within 30 minutes (`schedule_overlap`): pick another time.
- `update_social_post` edits anything not yet published. Editing a scheduled post sends it back to Review; schedule it again.

### Writing posts people stop for

- First line is the hook: a specific claim, number or tension. No throat-clearing.
- One idea per post. Short paragraphs, plain words, a concrete example or proof.
- End with a question or a clear next step, not a hard sell.
- 3 to 5 relevant hashtags at most. No links to pages you have not confirmed exist.
- Match the brand voice: read `brand_kit` and `ai_context` first.

## Forms

- `create_form` with `name`, `object` (for example `contact`) and `questions` in display order. Map each answer to a CRM field with `maps_to_field` (a field slug from `describe_workspace`) so every submission fills a record; at minimum map email and name.
- Question types: `text, email, tel, number, textarea, select, radio, checkbox, date, datetime, time, file, url`, and layout blocks `h2, paragraph, divider`.
- `publish: true` makes it public and returns the link. Confirm with the human before publishing.
- `update_form` renames, publishes / unpublishes, or replaces the full question list (existing submissions are kept).
- `list_forms` for ids and links; `list_form_submissions` for answers and the record each one created or updated.
- Keep forms short: every extra question lowers completion. Ask only what the follow-up needs.
