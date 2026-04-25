---
name: place-search
description: Search for nearby places (restaurants, cafes, hotels, gas stations, anything with a location) via a verified x402 pay-per-request endpoint. Use when the user asks "find X near Y" — without a Google Places or Foursquare API key.
---

# Searching places via EntRoute

When the user asks for nearby places, use this skill to query a ranked, fulfillment-verified x402 endpoint.

## Steps

1. **Parse the query and location.**
   - **Query:** extract the category or search term ("coffee shops", "sushi", "gas station", "hotels").
   - **Location:** accept a string ("Union Square, SF"), lat/lng, or a current-location reference. If only a string is given, pick an endpoint that accepts a string; otherwise geocode first via `geo.geocode`.
   - **Radius:** default 1000m if the user doesn't specify. Raise for rural areas, lower for dense cities.

2. **Discover the best endpoint.** Call EntRoute's `/discover`:

   ```bash
   curl -s -X POST https://api.entroute.com/discover \
     -H "Content-Type: application/json" \
     -d '{
       "capability_id": "geo.places_nearby",
       "constraints": {"network": "base", "verified_only": true},
       "preferences": {"ranking_preset": "balanced"}
     }'
   ```

   Take `ranked_endpoints[0]`. Check `sample_request` for exact param names — shapes vary significantly across providers. Common: `{ query, lat, lng, radius_m }`, `{ q, location, radius }`, or `{ category, near }`.

3. **Build the request matching `sample_request`.** Don't guess parameter names — providers disagree on almost every field.

4. **Pay with x402.** With `@entroute/mcp-server` or `@entroute/sdk-agent-ts` installed, `call_paid_api` / `discoverAndCall` handles payment.

5. **Parse and return.** Typical response: `results: [{ name, address, lat, lng, rating?, distance_m? }]`. Use `sample_response` to confirm the shape.

## Preferred approach

With Claude Code + `@entroute/mcp-server`, use the MCP tools `discover_paid_api` and `call_paid_api` directly.

## Notes

- Default network is Base; pricing is USDC. Places endpoints typically cost $0.002–$0.02 per call.
- If the user is asking for details on a single place (opening hours, phone, website), that may be a different capability — check `/capabilities` for something like `geo.place_details` before falling back here.
- Full docs: https://entroute.com/docs/quickstart
