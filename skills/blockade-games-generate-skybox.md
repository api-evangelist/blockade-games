---
name: Generate and retrieve a Skybox
description: Pick a style, submit a text-prompt skybox generation, and retrieve the finished 360 panorama and depth map.
api: openapi/blockade-games-skybox-openapi.yml
operations: [getSkyboxStyles, generateSkybox, getSkyboxById]
---

# Generate and retrieve a Skybox (Blockade Labs Skybox AI)

Use this skill to create a 360 equirectangular panorama from a text prompt.

## Auth
Send the header `x-api-key: {YOUR_API_KEY}` on every request. Base URL is
`https://backend.blockadelabs.com`. (A query param `api_key` also works but is
less secure — prefer the header.)

## Steps

1. **List available styles** — `getSkyboxStyles`
   `GET /api/v1/skybox/styles`. Choose a style and note its `id`
   (`skybox_style_id`). Optionally group with `getSkyboxStyleFamilies`.

2. **Submit the generation** — `generateSkybox`
   `POST /api/v1/skybox` (multipart/form-data). Required: `prompt`. Common
   fields: `skybox_style_id`, `negative_text`, `seed`, `remix_imagine_id` (to
   remix an existing skybox), and `webhook_url` for async callbacks. The
   response returns an imagine request with an `id`, a `status` of `pending`,
   and a `pusher_channel`.

3. **Wait for completion** — do NOT hammer the poll endpoint.
   - Preferred: pass `webhook_url` and handle the POST callbacks, or subscribe
     to the returned `pusher_channel` (`status_update` event).
   - Fallback: `getSkyboxById` — `GET /api/v1/imagine/requests/{id}` — until
     `status` is `complete`. Polling can trip the rate limiter.

4. **Read the result** — on `status: complete` the request carries `file_url`
   (full 8192x4096 image), `thumb_url`, and `depth_map_url`.

## Status model
`pending -> dispatched -> processing -> complete` (terminal), or `abort` /
`error` (terminal; `error` carries an `error_message`). See
`conventions/blockade-games-conventions.yml` and
`asyncapi/blockade-games-webhooks.yml`.

## Notes
- Generation is asynchronous — treat `id` as a job handle.
- No idempotency key is supported; a resubmit creates a new generation.
