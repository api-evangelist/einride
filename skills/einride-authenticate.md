---
name: Authenticate with Einride Extend
description: Exchange an issued client ID and client secret for a short-lived bearer token to call the Saga APIs.
api: openapi/einride-auth-openapi-original.yml
operations: [AuthenticationService_ExchangeSecret]
---

# Authenticate with Einride Extend (Saga)

Use this before any Shipment or Booking call. Credentials (client ID + client
secret) are issued by an Einride representative during the early-access phase.

## Steps
1. `POST /v1beta1/auth:exchangeSecret` (`AuthenticationService_ExchangeSecret`)
   with body `{ "clientId": "<id>", "clientSecret": "<secret>" }`.
2. Read `accessToken` and `expireTime` from the response.
3. Send `Authorization: Bearer <accessToken>` on every subsequent request.
4. Refresh by calling `ExchangeSecret` again before `expireTime`.

## Rules
- Root host: `https://api.saga.einride.tech` (regional: `eu.` / `us.` prefixes).
- Never log or embed the client secret; treat the access token as short-lived.
- On `UNAUTHENTICATED` (code 16 / HTTP 401), re-run this flow, then retry the call.
