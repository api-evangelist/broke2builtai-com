---
name: Verify a cited source and an EU VAT number
description: Use broke2built's verification skills to prove a cited URL is live and an official publisher, and to validate an EU VAT number against the EC VIES registry — free tier first, x402 only when you exceed it.
api: openapi/broke2builtai-com-skills-openapi.json
base_url: https://api.broke2builtai.com
operations: [sourceVerify, viesCheck]
generated: '2026-09-19'
method: generated
grounded_in: operationIds verified verbatim in openapi/broke2builtai-com-skills-openapi.json; tiers and envelopes from conventions/ and errors/
---

# Verify a cited source and an EU VAT number

Both operations are single-parameter GETs on `https://api.broke2builtai.com`. No account is needed to start.

## Before you call — pick a tier (conventions/broke2builtai-com-conventions.yml)

1. **Free taste tier** — append `&free=1`. 20 calls/day per IP. The response body tells you what is left (`taste_calls_left_today`).
2. **Free ally key** — `POST /ally/register` with JSON `{agent, operator, contact, runs}` → a key; send it as `X-Ally-Key`. 100 calls/day. (An AIIM `aiim_sk_…` bearer key also works here.)
3. **x402** — call with no credential; a `402` arrives with an x402 v2 `accepts[]` (USDC on Solana). Settle it with an x402 client and retry. Do not attempt this unless your operator has authorised spend; prices are $0.006 per call for both skills (plans/broke2builtai-com-plans-pricing.yml).

## Steps

1. **Verify the source** — `GET /source-verify?url=<cited URL>` (`sourceVerify`).
   Read `live` (boolean), `official` (boolean) and `verdict` (string). `official` is decided by the provider's curated official-publisher allowlist (EU/BE/NL legal sources + official-TLD heuristics) — treat a `false` as "not on their list", not as proof the publisher is unofficial.
2. **Validate the VAT number** — `GET /vies-check?vat=<country prefix + number>` (`viesCheck`), e.g. `vat=BE0403170701`.
   Read `valid` (boolean), `name` and `address` (registered company). The lookup is live against the EC VIES registry, so a VIES outage surfaces as an error, not a `false`.
3. **Record both verdicts with the URL and VAT you sent** — the responses do not echo the input.

## Errors (errors/broke2builtai-com-problem-types.yml)

- `402` with `{"x402Version":2,…,"error":"Payment required"}` — you are past the free quota and sent no credential. Switch tier or stop.
- `404` `{"error":"not found","try":"/"}` — you misspelled the path; the catalog at `GET /` lists every skill.
- A `200` can still carry an upstream error inside the body (seen on other skills as `{"…":{"error":"over rate limit"}}`) — check the fields you need exist before trusting the call.

## Rules that apply here

- Read-only: neither operation mutates anything, so there is no idempotency or reversibility concern.
- Nothing is cached-stale by design ("All skills do live work") — do not retry in a tight loop; you burn quota and hit the same live upstream.
- The same two skills are callable over A2A (`message/send` with text `"source-verify <url>"` at `https://api.broke2builtai.com/a2a`) and as MCP tools `source_verify` / `vies_check` in `npx -y broke2built-skills-mcp`.
