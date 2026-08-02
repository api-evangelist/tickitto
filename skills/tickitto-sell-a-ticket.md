---
name: Sell an event ticket with Tickitto
description: >-
  Search Tickitto's event inventory, fetch availability, add a selection to a basket, and check
  out — the core distributor flow for selling event tickets through the Tickitto API.
api: openapi/tickitto-openapi-original.json
operations:
  - create_basket
  - search_events
  - get_event
  - get_availability
  - add_basket_item
  - view_basket
  - checkout_basket
  - get_status
---

# Sell an event ticket with Tickitto

Tickitto is an availability-aware ticketing marketplace. Base URL is `https://tickitto.tech/api`
(use `https://dev.tickitto.tech/api` for development). All requests are JSON over HTTPS.

## Authentication
Send your API key in the `key` request header on every call (no password). Keys are issued by your
Tickitto account manager and are tied to your booking-fee and search settings. Never expose the key
in client-side code. Pass an `X-Correlation-ID` header to trace a request end to end.

## Steps

1. **Create a basket** — `create_basket` (POST `/api/basket/`). Baskets are anonymous and tied to
   your distributor account; capture the returned basket `_id` as `basket_id`. Optionally set
   `currency`.
2. **Search inventory** — `search_events` (GET `/api/events/`). Filter by `city`, `country`,
   `category`, price, `text`, and date `range`. Only events with availability in the requested
   window are returned. Page with `skip`/`limit` and order with `sort_by`. Use `get_event`
   (GET `/api/events/{event_id}`) for full detail on one event.
3. **Fetch availability** — `get_availability` (GET `/api/availability/`) with `event_id` and the
   `basket_id`. It returns a **Ticket Selection Widget** URL. Embed that URL as an iframe; the
   widget renders the right UI for the event's admission type (`open_entry`, `date_entry`,
   `slot_entry`, `timed_entry`) and is availability-aware in real time.
4. **Add to basket** — when the buyer selects tickets, the widget posts the selection to the
   basket automatically (via postMessage); you can also call `add_basket_item`
   (POST `/api/basket/add`). Reserved items carry a 15-minute `ttl`.
5. **Review** — `view_basket` (GET `/api/basket/`) with `basket_id`. Check each item's `ttl`;
   expired items are not guaranteed available at checkout.
6. **Check out** — `checkout_basket` (POST `/api/basket/checkout`) with `basket_id`. Payment is
   captured through Stripe or Careem Pay objects created via the basket endpoints. User-specific
   data is only required at checkout.

## Conventions & error handling
- Pagination is offset-based (`skip`/`limit`); currency is passed via `currency`.
- Errors return a structured envelope: `{ "request_id": …, "detail": [ { "type", "msg", "loc" } ] }`.
  Handle `401` (bad/missing key), `403` (permission/search restriction), `404` (missing event or
  basket), `409` (conflict), `410` (reservation gone/expired), `422` (validation), `429` (back off).
- Verify connectivity with `get_status` (GET `/api/status`) → `{ "status": "healthy" }`.
