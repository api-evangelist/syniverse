---
name: Send an SMS or MMS with the Syniverse Communication Gateway
description: >-
  Authenticate against the Syniverse Developer Community, send a message through the
  SCG omni-channel messaging API, and follow it to a terminal delivery state.
api: openapi/syniverse-omni-channel-messaging-openapi.yml
base_url: https://api.syniverse.com/scg-external-api/api/v1
operations:
  - POST /messaging/message_requests/
  - GET /messaging/message_requests/{MessageRequest_ID}
  - GET /messaging/message_requests/{MessageRequest_ID}/messages
  - GET /messaging/messages/{Message_ID}
  - GET /messaging/channels
  - GET /messaging/sender_ids
generated: '2026-07-25'
method: generated
---

# Send an SMS or MMS with the Syniverse Communication Gateway

The SCG definition is Swagger 2.0 and declares **no `operationId`s**. Address every
operation by HTTP method + path exactly as written above — they are verbatim from
`openapi/syniverse-omni-channel-messaging-openapi.yml`.

## 1. Authenticate

Every call carries an SDC application access token:

```
Authorization: Bearer {access token}
Content-Type: application/json
```

The token comes from a registered application in the Syniverse Developer Community.
The **first** token issued for an application expires after **1 hour**; once you
regenerate it via `GET /saop-rest-data/v1/apptoken-refresh` it does not expire. If a
call returns `SCG_ERROR_4046` (HTTP 404, "Access token expired"), refresh and retry —
see `skills/syniverse-refresh-sdc-access-token.md`.

## 2. Choose a sender

- `GET /messaging/channels` — list channels available to the application.
- `GET /messaging/sender_ids` — list provisioned sender addresses.

If you are on a trial account you cannot purchase a dedicated sender address. Use the
published public US test channel instead:

```
"from": "channel: 1KJPMkuHQkair_o15etpmg"
```

On a trial account, **every destination US mobile number must be whitelisted first**
(`sandbox/syniverse-sandbox.yml`, or the Whitelisting Service API).

## 3. Send

```
POST /scg-external-api/api/v1/messaging/message_requests
{
  "from": "channel: 1KJPMkuHQkair_o15etpmg",
  "to": ["+15551234567"],
  "body": "Hello from SCG message"
}
```

A success returns the message-request id: `{"id": "..."}`.

**There is no idempotency key on this API.** Retrying a failed `POST` may send a
duplicate message. Deduplicate on your side — carry your own correlation value in
`external_id` and check `GET /messaging/messages` filtered by `external_id` before
resending.

## 4. Follow delivery

A message request fans out into one **Message** per recipient.

- `GET /messaging/message_requests/{MessageRequest_ID}` — the request itself.
- `GET /messaging/message_requests/{MessageRequest_ID}/messages` — the per-recipient messages.
- `GET /messaging/messages/{Message_ID}` — one message and its `state`.

MT states: `CREATED`, `SENT`, `DELIVERED`, `READ`, `CONVERTED`, `FAILED`, `EXPIRED`,
`SCHEDULED`, `TEST`, `PAUSED`, `DELETED`. Only `DELIVERED`, `FAILED` and `EXPIRED` are
terminal for delivery purposes.

**Do not poll for delivery at scale.** Subscribe to the `SCG-Message` topic
(`message_state_change`) through Event Manager and take delivery receipts as pushed
events — `asyncapi/syniverse-event-manager-webhooks.yml`.

## 5. Handle failure

Synchronous errors come back as `error_code` / `error_description` — not RFC 9457
problem details. The ones you will actually hit:

| Code | HTTP | Meaning | Do |
|---|---|---|---|
| `SCG_ERROR_4001` | 400 | Required parameter missing | Fix the payload; do not retry as-is |
| `SCG_ERROR_40024` | 400 | Invalid recipient | Validate the number (see the phone-number-verification skill) |
| `SCG_ERROR_40025` | 400 | Invalid payload | Send valid, non-empty JSON |
| `SCG_ERROR_4005` | 400 | Message size exceeds system limit | Shorten the body |
| `SCG_ERROR_4010` | 401 | Token unauthorized for this resource | Wrong application token |
| `SCG_ERROR_4046` | 404 | Access token expired | Refresh the token and retry |
| `SCG_ERROR_4021` | 402 | Quota exceeded | Stop; contact Syniverse |
| `SCG_ERROR_5000` / `5030` | 500 / 503 | Server error / unavailable | Back off and retry |

Asynchronous failures arrive later on the event stream as numeric codes — `1002`
invalid recipient, `1005` no subscriber consent, `1039` throttling, `1067` too many
messages to the same number. A `200` on the send is **not** proof of delivery. Full
registry: `errors/syniverse-error-codes.yml`.

## 6. Respect throughput

MMS delivery to US operators is capped at **20 TPS**. SMS/MMS output is throttled per
sender address, per customer and system wide, and your sender class governs throughput
and sending window. No `RateLimit-*` or `Retry-After` header is returned — back off on
`503` and on asynchronous code `1039`. See `rate-limits/syniverse-rate-limits.yml`.
