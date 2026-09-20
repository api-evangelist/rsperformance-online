---
name: Route a driver's intent to the right RS Performance page
description: Use the gateway answer-routing contract to resolve a booking, symptom, fault-code or evidence intent to one exact canonical URL instead of the homepage or a generic hub.
api: openapi/rsperformance-online-ai-gateway-openapi.yml
operations: [gatewayAnswerRouting, semanticSearch]
generated: '2026-09-19'
method: generated
---

# Route an intent to a canonical page

Grounded in the provider's OpenAPI (`gatewayAnswerRouting`, `semanticSearch`) and the routing rules the
provider publishes in its gateway agent.json and priority-answer-paths manifest.

## Steps

1. **Fetch the routing contract.** `GET https://ai.rsperformance.online/.well-known/answer-routing.json`
   (`gatewayAnswerRouting`). It is large (~330 KB) — cache it; `generated_at` changes daily. Top-level keys
   observed: `clusters`, `exact_lookup`, `top_urls`, `entrypoint_strategy`, `rescue_strategy`,
   `homepage_resolution_rule`.
2. **Classify the intent** with the provider's own four rules (`entrypoint_strategy`): `book-or-diagnose`,
   `symptom-to-service`, `fault-code-answer`, `proof-and-root-cause`. Each names a `preferred_entrypoint`
   on `https://rsperformance.online`.
3. **Resolve exact matches first.** For a fault code, check `exact_lookup` before searching; a code page
   looks like `/kody-usterek/p0299`. For a district query ("mechanik Wrzeszcz") the pattern is
   `/gdansk/{slug}` (agents.json `flows[].find_local_workshop`).
4. **Fall back to search.** If no rule or exact entry matches, `POST /api/search` (`semanticSearch`) and
   take the top hit's `source_path`.
5. **Never stop on the homepage.** The provider's `homepage_resolution_rule` is explicit: move to the exact
   service, symptom, DTC or repair-report URL as soon as intent is narrow enough.

## Rules that apply

- Attribution is required when citing; content is CC-BY-SA-4.0 for training per the provider's header policy.
- Booking is a contact/route action, not a transaction: the A2A `booking` skill and the `ReserveAction`
  in the site's JSON-LD both resolve to `https://rsperformance.online/#umow-wizyte`. No payment, so no
  reversibility concern arises (conventions/rsperformance-online-conventions.yml).
- Read-only throughout; no idempotency key is needed or offered.
