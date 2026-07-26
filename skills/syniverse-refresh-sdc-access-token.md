---
name: Keep an SDC application access token valid
description: >-
  Regenerate the Syniverse Developer Community application access token before it
  expires, and recover cleanly from an expired-token failure mid-flow.
api: openapi/syniverse-token-management-openapi.yml
base_url: https://api.syniverse.com
operations:
  - GET /saop-rest-data/v1/apptoken-refresh
generated: '2026-07-25'
method: generated
---

# Keep an SDC application access token valid

Grounded in `openapi/syniverse-token-management-openapi.yml` and the published security
documentation at
<https://sdcdocumentation.syniverse.com/index.php/reporting/security>.

## The token model

A registered SDC application holds three credentials: a **consumer key**, a **consumer
secret** and an **access token**. Requests carry the access token:

```
Authorization: Bearer {access token}
```

The lifetime rule is unusual and worth stating plainly:

- The **initial** access token generated for an application expires after **1 hour**.
- Once you regenerate it, the token **does not expire** until you regenerate it again.
- You may regenerate as often as you like.
- If the access token leaks, regenerate it — you do not need a new application.
- If the **consumer key or secret** leaks, there is no rotation path: you must create an
  entirely new application with fresh credentials.

## The operation

```
GET https://api.syniverse.com/saop-rest-data/v1/apptoken-refresh
  ?consumerkey={consumer key}
  &consumersecret={consumer secret}
  &oldtoken={current access token}
  &validity={validity}
```

All four are declared query parameters in the definition: `consumerkey`,
`consumersecret`, `oldtoken`, `validity`. Declared responses are `200`, `400` and
`500`.

**The secret travels in the query string.** That means it can land in proxy logs,
browser history and access logs. Call this server-side only, never from a browser or
mobile client, and make sure your own logging redacts the query string for this path.

## Recovering mid-flow

Any SCG call can come back with:

```
HTTP 404
{"error_code": "SCG_ERROR_4046", "error_description": "Access token expired"}
```

Handle it as: refresh once → retry the original request once → if it fails again,
surface the error. Do not loop.

Because the SCG API has **no idempotency key**, be careful what you retry. Re-issuing a
`POST /messaging/message_requests` after a token refresh can send the message twice if
the first attempt actually reached the gateway. Carry your own `external_id` and check
`GET /messaging/messages` filtered by it before resending a write.

## Reference implementation

Syniverse publishes working refresh examples in Java and Python 2/3:
<https://github.com/Syniverse/Example-SDC-Token-Refresh>
