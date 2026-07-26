---
name: Subscribe to Syniverse events instead of polling
description: >-
  Use Event Manager (the Event Subscription Service) to receive message delivery
  receipts, inbound messages, number-status changes and batch job events as pushed
  REST deliveries rather than polling the messaging API.
api: null
base_url: https://api.syniverse.com/ess/v1
operations:
  - GET /topics
  - POST /topic-subscriptions
  - POST /delivery-configurations
  - GET /event-types
  - GET /event-deliveries
  - GET /event-delivery-attempts
  - GET /dictionaries
generated: '2026-07-25'
method: generated
---

# Subscribe to Syniverse events instead of polling

Event Manager is the one Syniverse Developer Community product with **no downloadable
machine-readable definition** — it is documented as a rendered RAML console at
<https://sdcdocumentation.syniverse.com/index.php/reporting/event-manager/resources>.
The resource paths and topic/event names below are taken verbatim from that
documentation, not from a spec. Confirm request and response shapes against the console
before you code against them. The captured catalog lives in
`asyncapi/syniverse-event-manager-webhooks.yml`.

Base URL: `https://api.syniverse.com/ess/v1`.

## 1. Discover what you can subscribe to

- `GET /topics` — available event topics.
- `GET /event-types` — supported event types and their fields.
- `GET /dictionaries` — supported delivery formats, protocols and authentication types.

The topics that matter:

| Topic | Events |
|---|---|
| `SCG-Message` | `message_state_change`, `mo_message_received` |
| `NIS-Events` | `porting_event`, `deactivation_event` |
| `SCG-Voice-Call` | call state events, DTMF events |
| `MSS-Messages` | `event_file_complete` |
| `ABA-Messages` | batch job lifecycle events |

`message_state_change` is what replaces polling `GET /messaging/messages`.
`porting_event` and `deactivation_event` are what replace repeated Phone Number
Verification lookups on numbers you already know.

## 2. Describe where events should land

`POST /delivery-configurations` defines the endpoint and the delivery contract:

- **Protocol:** REST.
- **Format:** JSON or XML.
- **Mode:** near real time, or aggregated on a simple/cron schedule into JSON files
  (retrievable through `/event-buffer-files`).
- **Throttling:** optional rate limit expressed in events per minute — set this to
  protect your own endpoint.
- **Authentication** Syniverse presents to *your* endpoint: `NONE`, `BASIC`, or
  `OAUTH` (client_credentials, password or refresh_token).

**Never leave this at `NONE` in production.** Syniverse does not sign event payloads —
there is no HMAC signature header to verify. Endpoint authentication is the only thing
standing between your handler and a forged delivery receipt. Use `OAUTH`, or `BASIC`
over TLS at minimum, and treat the event body as untrusted until you re-read the
resource from the SCG API.

## 3. Subscribe

`POST /topic-subscriptions` binds your application to a topic and a delivery
configuration. `GET /topic-subscriptions` reads back every subscription for the
company.

## 4. Verify delivery

- `GET /event-deliveries` — delivery status per event.
- `GET /event-delivery-attempts` — individual attempts.
- `GET /event-deliveries/{EVT_ID}:{DEL_CFG_ID}/tracking` — the attempt trail for one
  event on one delivery configuration.

Delivery states are `SUCCESS`, `PENDING`, `RETRY` and `FAILED`. Syniverse retries, so
your handler **must be idempotent** — the same event can arrive more than once and the
platform gives you no idempotency key. Deduplicate on the event id.

## 5. Read the failure codes

Asynchronous message failures arrive on `message_state_change` as numeric codes:
`1002` invalid recipient, `1005` no subscriber consent, `1007` expired, `1012` message
too large, `1039` throttled, `1067` too many messages to the same number. A `200` on the
original send is not delivery — this stream is where you learn what actually happened.
Full registry: `errors/syniverse-error-codes.yml`.
