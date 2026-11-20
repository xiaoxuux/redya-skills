---
name: redya-xiaohongshu
description: Create Xiaohongshu illustrated posts, product posts, reference adaptations and Live Photos with RedYA. Review sensitive wording or hand a finished draft to the publishing editor.
metadata:
  displayName: "小红书图文与实况图·红鸦 AI"
---

# 小红书图文与实况图·红鸦 AI

Use this skill in an agent that can make HTTPS requests. It creates a title, caption and illustrated pages from a topic, saved product or reference note. It can also process supplied photos, review text on its own, or open a completed post in the platform editor. Use the language requested for the post. Market names and brand names do not establish a partnership.

## Help before configuration

When the user asks how to use this skill (including “怎么用”), show these seven examples directly, in the user's language. Viewing them does not require a key, spend credits, or call the service.

1. **Illustrated post:** “Create four pages about organizing a weekend desk. Show me the title, caption and outline first.”
2. **Product post:** “Use my saved dried-fig product for an office snack post. Use only the facts in its profile, show the packaging only on the last page, and generate the images now.”
3. **Adapt a reference:** “Adapt this post: [paste the entire Xiaohongshu share message]. Keep its page layout, change the subject to skincare after 25, and generate the images in one step.”
4. **Live Photos, originals:** “Use the photos I attached for a weekend café post. Keep the original pictures, write the text, and turn them into Live Photos.”
5. **Live Photos, generated pictures:** “Make a sneaker unboxing post using my product photo as a reference for realistic pictures. Show me the outline before making Live Photos.”
6. **Check copy:** “Check this caption for sensitive wording: [your caption]. Show the terms, reasons and suggestions before changing anything.”
7. **Publish a finished draft:** “Bring the post we just made into the Xiaohongshu publishing editor with the title, caption and images. I will confirm publication.”

For an outline-first post, the user can request edits to a particular page and then ask to generate images. Mode and timing are independent. Never silently substitute a general post for a requested reference adaptation, product post or Live Photo.

## Connect to RedYA

The user generates a key in RedYA Skills at https://hy.ithinkai.cn/developer and configures `REDYA_API_KEY` in their agent environment. Only send this key to `https://hy.ithinkai.cn`, via `Authorization: Bearer`. Do not request that the user paste it into chat, include it in a URL, display it, or forward it to a media host or publishing platform. A missing key or HTTP 401 stops authenticated operations. An outline, parsing a share message, generating images, copy checks and AI video effects can consume the user's credits. Check account information at `GET /openapi/v1/me` and relevant quotes; confirm the scope of paid work before submission. The skill has no bundled credits or keys.

The paths below are relative to `https://hy.ithinkai.cn/openapi/v1`. Send JSON in UTF-8. Save each returned source, record or task ID immediately. Poll that ID after a timeout or lost acknowledgement; do not automatically submit a second paid job. For the complete current wire format, see the [production API skill reference](https://hy.ithinkai.cn/openapi/skill.md).

## Select the requested mode

- **Ordinary:** `POST /outlines` with `{"creation_mode":"normal","topic":"...","auto":false}`. Use `auto:true` when the user requests direct image generation.
- **Product:** look up `GET /products` and match the user's existing product. For new user-supplied photos, `POST /products` accepts JSON with `name` and `image_urls`, or multipart `params` JSON and repeated `images`. Then `POST /outlines` with `creation_mode:"product"`, the string `product_id`, `topic` and `auto`. Use the supplied product facts; ask for missing assets instead of inventing claims. Keep `product_id` when generating product images.
- **Reference adaptation:** send the complete share message as `{"url":"..."}` to `POST /sources`. Poll `GET /sources/:source_id` every 5–10 seconds until `ready` or `failed`. Check the title, author, type and number/order of images. For a supported illustrated note use `POST /outlines` with `creation_mode:"viral_clone"`, `source_id`, `topic`, and `clone_mode:"new_topic"` or `"same_topic"`. Map each source page to the corresponding new page. A clear request for direct output sets `auto:true`; do not ask for the same approval twice. Stop if parsing fails, the source has no required images, or the response's mode differs. Never call an unrelated parser or downgrade the request to ordinary mode.
- **Live Photo post:** `POST /outlines` with `creation_mode:"live_photo"`, `topic`, `auto` and `live_settings`. For original pictures use `{"sourceMode":"upload","withCopy":true,"mode":"live"}`; use `sourceMode:"generate"` for new realistic pictures. Supply accessible `image_urls` or multipart `params` containing the complete JSON with repeated `images`. Save `record_id` and `live_url`. For staged original photos wait at `ready_for_live` for approval. For staged AI images approve the outline, request still images via `POST /images` and wait for them. Once approved, `POST` the returned same-origin `live_url` and poll it with `GET`. Direct mode advances automatically. Handle `review_required` before advancing. Download the resulting assets only when the Live task is complete.

## Review, edit and deliver

Check the `creation_mode` returned by the service. For a staged post present the title, caption and complete `pages[].content`, including visual instructions and source order. When the user asks for edits, `PUT /outlines/:record_id` with the full revised page set, then verify the saved text. On confirmation, `POST /images` with the same `record_id` (and `product_id` in product mode). For direct generation, set `auto:true` on the original outline request and follow the returned record rather than creating images twice. Poll `GET /records/:record_id` roughly every 15–30 seconds. Report completed, partial and failed pages separately. Deliver working image links or downloaded assets and the caption; the existence of a record alone does not mean its images are finished.

- **Standalone copy review:** `GET /content-review/status` gives the account configuration and quote; submit `POST /content-review` with `fields`, e.g. `{"fields":[{"id":"body","label":"Caption","text":"Text to check"}]}`. Include title, caption and per-page copy when checking a post. Show the findings and reasons. Only apply user-confirmed changes via `POST /content-review/replace` with the original fields, complete signed `review`, chosen `finding_ids`, and `action` of `replace` or `delete`. A clean report does not guarantee platform approval.
- **Standalone photo conversion:** check `GET /live-photos/capabilities`. `POST /live-photos` as multipart with the selected `image`, saved UUID `request_id` and `mode` (`live` for camera effects; `ai` only with approval for a current model, quality and quote). Poll `GET /live-photos/:request_id`. Reuse the UUID on recovery. Return the downloaded file; distinguish file delivery from successful import into the user's phone.
- **Publishing handoff:** for an existing completed record use `POST /records/:record_id/publish` with the requested platform, for example `{"platform":"xiaohongshu"}`. On `awaiting_browser`, open `editor_path` on RedYA in the user's signed-in browser with the publishing extension; select the returned `menu_label`. Verify the title, caption, and image count in the destination editor. Report a draft until the user requests publication and the destination confirms it. If authentication, the browser or extension is missing, report the exact point reached.
