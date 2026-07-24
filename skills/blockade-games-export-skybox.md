---
name: Export a completed Skybox
description: Request an export of a finished skybox into a downloadable file format and retrieve the export.
api: openapi/blockade-games-skybox-openapi.yml
operations: [getExportTypes, requestExport, getExportRequest]
---

# Export a completed Skybox (Blockade Labs Skybox AI)

Use this skill to turn a completed skybox generation into a downloadable file
(equirectangular JPG/PNG, cube map, depth map, HDRI, video, etc.).

## Auth
Header `x-api-key: {YOUR_API_KEY}`. Base URL `https://backend.blockadelabs.com`.

## Steps

1. **List export types** — `getExportTypes`
   `GET /api/v1/skybox/export`. Choose a format and note its `type_id`.

2. **Request the export** — `requestExport`
   `POST /api/v1/skybox/export` (application/json). Body: `skybox_id` (the `id`
   of a completed skybox from the generate skill), `type_id`, and optional
   `webhook_url`. Returns an export request with an `id` and a `status`.

3. **Wait for completion** — `getExportRequest`
   `GET /api/v1/skybox/export/{id}` until `status: complete`, or use the
   `webhook_url` callback. On completion the export carries the downloadable
   `file_url`. Use `cancelExportRequest` (`DELETE /api/v1/skybox/export/{id}`)
   to abort.

## Notes
- Export only completed skyboxes — a `skybox_id` whose generation is not
  `complete` cannot be exported.
- Same async status model as generation (see
  `conventions/blockade-games-conventions.yml`).
