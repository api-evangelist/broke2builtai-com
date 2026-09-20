---
name: Render a narrated video from a topic
description: Turn a topic into a finished narrated vertical video (mp4 plus title/tags kit) with broke2built's async render pipeline, polling the job until it finishes and respecting the one-free-render limit.
api: openapi/broke2builtai-com-skills-openapi.json
base_url: https://api.broke2builtai.com
operations: [renderVideo, getVideo]
generated: '2026-09-19'
method: generated
grounded_in: operationIds, request schema and enum values verified verbatim in openapi/broke2builtai-com-skills-openapi.json; tiers from plans/
---

# Render a narrated video from a topic

## Prerequisite — an ally key

`POST /ally/register` → key → header `X-Ally-Key`. Each ally gets **one free render**; after that the operation is x402-paid or "self-serve with your own free GLM key" (the provider's words). Do not start a second render without operator approval of spend.

## Steps

1. **Start the render** — `POST /video` (`renderVideo`) with JSON `{"topic": "<one sentence>", "voice"?: "af_heart" | "af_bella" | "am_michael" | "bm_george"}`. `topic` is required; `voice` must be one of those four enum values or omitted.
   The response is **not the video**: `{job_id, poll}`. Store `job_id`.
2. **Poll** — `GET /video/{id}` (`getVideo`) with the `job_id`. Read `status`; when finished the same document carries `video` (URL of the h264 mp4), `title` and `tags[]`. Poll at a humane interval — the pipeline (script → Kokoro voice → scenes → captions + music → encode) is minutes, not seconds; a tight loop spends your daily quota (100 calls/day on the ally tier) for nothing.
3. **Fetch the mp4 from `video` and keep the kit** (`title`, `tags`) alongside it — that is the upload-ready metadata the product promises.

## Rules that apply here

- **Not idempotent.** A retried `POST /video` after a network failure is a second render and may consume the single free render. On ambiguity, poll the `job_id` you already have; do not re-POST.
- **No cancel.** Nothing in the contract stops a render once started (conventions/broke2builtai-com-conventions.yml → reversibility.not_documented).
- The spec declares only a `200` for both operations; expect the host's generic shapes on failure (errors/broke2builtai-com-problem-types.yml): `404` `{"error":"not found","try":"/"}` and the x402 v2 `402`.
