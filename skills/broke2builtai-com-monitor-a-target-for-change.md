---
name: Monitor a target for change
description: Stand up a 24/7 change-watch on an endpoint, DNS name, page, token supply or wallet with broke2built, receive an optional webhook on change, and read the change history back — knowing that no stop operation is published.
api: openapi/broke2builtai-com-skills-openapi.json
base_url: https://api.broke2builtai.com
operations: [createWatch, getWatch]
generated: '2026-09-19'
method: generated
grounded_in: operationIds verified verbatim in openapi/broke2builtai-com-skills-openapi.json; reversibility from conventions/; webhook surface from asyncapi/
---

# Monitor a target for change

## Prerequisite — an ally key

`POST /ally/register` with `{agent, operator, contact, runs}` → a key. Send it as `X-Ally-Key` on both operations (the OpenAPI's `AllyKey` scheme). The ally tier is free and capped at 100 calls/day.

## Steps

1. **Decide what to watch.** `kind` must be one of `endpoint` (uptime), `dns` (hijack/migration), `page` (defacement/content), `token-supply` (mint/burn), `wallet` (movement). `target` is the URL, name or address for that kind.
2. **Create the watch** — `POST /watch` (`createWatch`) with JSON `{kind, target, notify?}`.
   `notify` is optional: an HTTPS URL the service will POST to when a change is detected. If you supply one, it must be a receiver you control — the payload schema, signature and retry policy are **not published** (asyncapi/broke2builtai-com-watch-webhooks.yml), so build the receiver to accept any JSON and to verify by re-reading the watch (step 4) rather than trusting the body.
   Keep the returned `watch_id` and `baseline` — the baseline is the state every later change is measured against.
3. **Wait.** The provider checks every 2 hours; nothing arrives sooner.
4. **Read state and history** — `GET /watch/{id}` (`getWatch`) with the `watch_id`. This is the pull path and the source of truth whether or not you set `notify`.

## What you must tell your operator before step 2 (conventions/ → reversibility)

- **There is no documented way to stop a watch.** The OpenAPI has no `DELETE /watch/{id}` and the catalog documents no cancel. A watch "checks every 2h, 24/7" until the provider stops it. Create watches only when that is acceptable, and keep a record of every `watch_id` you create.
- **Not idempotent.** Retrying `POST /watch` after a timeout creates a second watch on the same target. On an ambiguous failure, stop and reconcile with your operator rather than re-POSTing.

## Errors (errors/broke2builtai-com-problem-types.yml)

- `405` `{"error":"POST JSON: {agent, operator, contact, runs, via?}"…}` — you sent GET to `/ally/register`.
- `404` `{"error":"not found","try":"/"}` — wrong path or an unknown `watch_id`'s route.
- `402` (x402 v2 envelope) — you called without a credential and past the free quota.
