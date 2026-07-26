---
name: Screen inbound messaging traffic for spam and abuse
description: >-
  Classify inbound text and MMS traffic with the Syniverse Messaging Trust Resolve API
  and feed confirmed abuse back through the Spam Datafeed.
api: openapi/syniverse-messaging-trust-resolve-openapi.yml
base_url: https://api.mt1.messaging-trust.syniverse.com/api
operations:
  - POST /v1/txt/resolve
  - POST /v1/mms/resolve
  - POST /spam/report
generated: '2026-07-25'
method: generated
---

# Screen inbound messaging traffic for spam and abuse

Two definitions back this flow:
`openapi/syniverse-messaging-trust-resolve-openapi.yml` (Messaging Trust Resolve 1.0.0)
and `openapi/syniverse-messaging-trust-datafeed-openapi.yml` (MT Spam Datafeed 1.0.0).
Neither declares `operationId`s — address by method + path.

## Authentication is different here

Messaging Trust is the **only** Syniverse family that uses true OAuth 2.0
client-credentials rather than the SDC application bearer token:

```
POST https://api.mt1.messaging-trust.syniverse.com/oauth2/token
grant_type=client_credentials
```

No scopes are declared in the definition. Do not reuse an SDC application token here,
and do not reuse a Messaging Trust token on the SCG API.

**Host caveat:** the host published in `servers[]`
(`api.mt1.messaging-trust.syniverse.com`) does not resolve in public DNS as of
2026-07-25. This is a customer-provisioned or private-network surface — get the
reachable host from your Syniverse account team rather than assuming the published one.

## 1. Resolve a text message

```
POST /api/v1/txt/resolve
```

Request schema `endpoint.TextResolveRequest`, response `endpoint.TextResolveResponse`.
Declared responses: `200`, `400`, `404`, `415`, `500`; errors use
`endpoint.FilterResponseError`.

Responses carry an **`x-request-id`** header (example value
`94bf014e-580b-49ba-b86e-c2fc0920204d`). This is the only correlation header declared
anywhere in the Syniverse estate — log it on every call, it is what support will ask
for.

## 2. Resolve an MMS

```
POST /api/v1/mms/resolve
```

This operation accepts **two** request media types:

- `application/json` → `endpoint.MMSResolveRequestJson`
- `message/rfc822` → `endpoint.MMSResolveRequestMm4`, i.e. a raw 3GPP MM4 message with
  `X-Mms-Message-ID`, `X-Mms-Transaction-ID`, `X-Mms-Message-Type`,
  `X-Mms-MMS-Version`, `X-Mms-3GPP-MMS-Version`, `X-Mms-Delivery-Report`,
  `X-Mms-Ack-Request` and `X-Mms-Originator-System` headers.

Set `Content-Type` deliberately. A `415 Unsupported Media Type` here means you sent the
wrong one — it is not a transient failure and retrying will not help.

## 3. Report confirmed spam

```
POST /api/v1/spam/report
```

Request `endpoint.SpamReportRequest`, response `endpoint.SpamReportResponse`, errors
`endpoint.SpamReportError`. Declared responses: `202`, `400`, `404`, `415`, `500`.

The success is **`202 Accepted`**, not `200`. The report is queued for the datafeed, not
processed inline — do not treat the response as an adjudication.

## Failure handling

Messaging Trust errors do not use the `SCG_ERROR_*` registry; they use their own
schemas (`endpoint.FilterResponseError`, `endpoint.SpamReportError`). Retry `500` with
backoff; do not retry `400`, `404` or `415`. There is no idempotency key — a retried
spam report is a duplicate report.
