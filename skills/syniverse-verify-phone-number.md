---
name: Verify a phone number before you message it
description: >-
  Use the Syniverse Phone Number Verification API to check carrier, line type and
  status (ported, deactivated, disconnected, reassigned) before sending, and pair it
  with the SIM-swap and identity-match risk signals.
api: openapi/syniverse-phone-number-verification-openapi.yml
base_url: https://api.syniverse.com/numberidentity/v3
operations:
  - POST_lookup
  - POST /v1/simCheck
  - POST /v2/match
generated: '2026-07-25'
method: generated
---

# Verify a phone number before you message it

Three separate Syniverse services answer three different questions. They are separate
APIs on separate hosts with separate definitions — do not assume one call covers all
three.

## 1. Is this number live, and on which carrier?

`POST_lookup` — `POST https://api.syniverse.com/numberidentity/v3/lookup`
(`openapi/syniverse-phone-number-verification-openapi.yml`; this is the one Syniverse
definition that declares a real `operationId`).

Auth: `Authorization: Bearer {SDC application access token}` (declared as
`http`/`bearer` with `bearerFormat: JWT` in the spec).

Optional correlation parameters declared on the operation: `ext_trx_id` and
`ext_reseller_cust_id`. Set `ext_trx_id` to your own request id — this API declares no
`x-request-id` response header, so your value is the only correlation handle you get.

Declared responses: `200`, `400`, `401`, `403`, `404`, `405`, `500`, `503`. Error code
meanings are published separately in the support knowledge base (see
`errors/syniverse-error-codes.yml`).

Use it to drop disconnected and reassigned numbers **before** they burn message spend
and before they trigger `SCG_ERROR_40024` / asynchronous failure `1002`.

For continuous coverage rather than point lookups, subscribe to the `NIS-Events` topic
in Event Manager and take `porting_event` and `deactivation_event` as pushed events —
`asyncapi/syniverse-event-manager-webhooks.yml`.

## 2. Has this number's SIM changed recently?

`POST /v1/simCheck` on `https://mobileidsandbox.syniverse.com/cigateway/id`
(`openapi/syniverse-account-takeover-detection-openapi.yml`). Returns whether the
mobile channel has had recent SIM changes or call forwarding enabled, as an
account-takeover risk signal.

Two cautions:

- The **only** host published in the definition is a sandbox host. Get the production
  host from your Syniverse account team; do not assume it.
- This is **not** CAMARA SIM Swap. It is a Syniverse-proprietary contract with its own
  schema and naming. Code written against CAMARA will not fit.

Use it as a step-up gate before an OTP: if the SIM changed inside your risk window, do
not deliver a one-time passcode to that number.

## 3. Does the asserted identity match the number?

`POST /v2/match` on `https://mobileidsandbox.syniverse.com/cigateway/id`
(`openapi/syniverse-right-party-verification-openapi.yml`). Returns match scores
comparing an asserted identity against Syniverse-sourced mobile data. Again
sandbox-hosted in the published definition, and again not the CAMARA KYC Match
specification.

## Sequencing

For a "send an OTP to a new user" flow the defensible order is:

1. `POST_lookup` — is the number live and mobile?
2. `POST /v1/simCheck` — has the SIM moved recently?
3. `POST /v2/match` — does the identity line up? (only where you have asserted identity data)
4. Only then `POST .../login_start` to deliver the passcode
   (`skills/syniverse-send-otp-and-validate.md`).

Every one of these is a paid lookup. Cache within a sensible window rather than calling
per message.
