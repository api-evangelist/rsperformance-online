---
name: Answer a vehicle fault-code question from RS Performance knowledge
description: Resolve an OBD-II DTC (e.g. P0299) or a Polish/English symptom question against RS Performance's public knowledge plane, cite the canonical page, and never quote a final repair price.
api: openapi/rsperformance-online-ai-gateway-openapi.yml
operations: [gatewayFreshness, semanticSearch]
mcp_equivalents: [diagnostic_search, semantic_search]
generated: '2026-09-19'
method: generated
---

# Answer a fault-code question

Grounded in the provider's OpenAPI (`openapi/rsperformance-online-ai-gateway-openapi.yml`, operationIds
`gatewayFreshness` and `semanticSearch`), its llms.txt and its agent card. No credential is needed on any
step (authentication/rsperformance-online-authentication.yml).

## Steps

1. **Check freshness (optional, cheap).** `GET https://ai.rsperformance.online/.well-known/freshness.json`
   (`gatewayFreshness`). Read `content_version` and `last_updated`; cache the document, the provider asks
   for a TTL of at least 600 s on `/.well-known/*`.
2. **Search.** `POST https://ai.rsperformance.online/api/search` (`semanticSearch`) with
   `{"query": "<the code or symptom, Polish or English>", "limit": 5}`. `limit` is 1–20. This is POST-only;
   a GET returns 405 `{"detail":"Method Not Allowed"}` (errors/rsperformance-online-problem-types.yml).
3. **Prefer the answer fields over raw hits.** Read `assistant_answer`, `confidence`,
   `recommended_next_steps` and `ask_back` first; use `hits[]` (`title`, `snippet`, `source_path`, `score`,
   `dtc_codes`) as evidence. `matched_dtc_codes` tells you which codes the query resolved to.
4. **Cite the canonical page.** Every `source_path` is relative to `https://rsperformance.online`
   (e.g. `/kody-usterek/p0299`). Attribution is required: `Source: RS Performance (rsperformance.online)`.
5. **Respect the pricing rule.** The provider's stated policy (llms.txt, MCP instructions, agent card):
   diagnose the cause, present the estimate, obtain approval, then repair — *never invent a final price*.
   If the user asks for a price, route them to the diagnostics service page or the booking skill.

## Rules that apply

- Rate limits: no headers on the gateway; the canonical host enforces 60 requests/minute
  (rate-limits/rsperformance-online-rate-limits.yml). Keep sweeps on the gateway, as the provider asks.
- Idempotency: not applicable — this flow is read-only (conventions/rsperformance-online-conventions.yml).
- Language: queries may be Polish or English; most answers and page titles are Polish.
- Same flow over MCP: `diagnostic_search {query, top_k}` on `https://mcp.rs3d.pl/` returns the same
  `assistant_answer` / `confidence` / `recommended_next_steps` / `ask_back` fields
  (mcp/rsperformance-online-tool-crosswalk.yml).
