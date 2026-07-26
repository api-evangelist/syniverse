---
name: Deliver and validate a one-time passcode
description: >-
  Register a Multi-Factor Authentication application, associate an end user, deliver a
  one-time passcode over SMS, voice or push, and validate what the user typed back.
api: openapi/syniverse-multi-factor-authentication-openapi.yml
base_url: https://api.syniverse.com/scg-external-api/api/v1
operations:
  - POST /applications
  - GET /applications
  - GET /applications/{Application_ID}
  - GET /applications/{Application_ID}/users
  - POST /applications/{Application_ID}/users/{User_ID}/associate
  - POST /applications/{Application_ID}/users/{User_ID}/login_start
  - POST /applications/{Application_ID}/users/{User_ID}/validate
  - DELETE /applications/{Application_ID}/users/{User_ID}/associate
generated: '2026-07-25'
method: generated
---

# Deliver and validate a one-time passcode

Grounded in `openapi/syniverse-multi-factor-authentication-openapi.yml` (Swagger 2.0,
host `api.syniverse.com`, basePath `/scg-external-api/api/v1`). The definition declares
**no `operationId`s** — address operations by method + path. Note that several paths
are published with literal quote characters around them in the source definition
(`/'applications/{Application_ID}/users/{User_ID}/validate'`); the real request path is
the unquoted form used above.

Auth: `Authorization: Bearer {SDC application access token}`.

## 1. Define the MFA application

`POST /applications` creates the passcode policy. The declared attributes are the
policy — set them deliberately:

- `auth_code_length` — how many digits/characters the passcode carries.
- `auth_token_type` — the token format.
- `auth_token_validity_duration` — how long a passcode stays valid. Keep this short.
- `auth_token_prefix` — a prefix carried on the token.
- `auth_token_delivery_methods` — SMS, voice or push.
- `message_from` — the sender the passcode is delivered from.
- `status` — application state.

`GET /applications` and `GET /applications/{Application_ID}` read it back.

## 2. Associate the end user

`POST /applications/{Application_ID}/users/{User_ID}/associate` binds an end user to
the application. `GET /applications/{Application_ID}/users` lists them.
`DELETE /applications/{Application_ID}/users/{User_ID}/associate` unbinds a user — do
this on account closure or device change, not as a retry.

## 3. Deliver the passcode

`POST /applications/{Application_ID}/users/{User_ID}/login_start` generates and
delivers the passcode over the configured channel.

**Gate this call.** Before delivering, check the number is live
(`POST_lookup`) and that its SIM has not moved recently (`POST /v1/simCheck`) —
`skills/syniverse-verify-phone-number.md`. Delivering a passcode to a freshly swapped
SIM is exactly the attack these products exist to stop.

## 4. Validate

`POST /applications/{Application_ID}/users/{User_ID}/validate` checks the passcode the
user supplied.

## Error handling

Every operation in this definition declares `400`, `401`, `402`, `403`, `404`, `409`,
`429` and `500`. This is the **only** Syniverse surface that declares `429` on every
operation — treat `429` as the throttle signal here and back off. No `Retry-After`
header is declared, so use exponential backoff with jitter.

`409 Conflict` on `login_start` typically means a passcode is already outstanding for
that user; do not immediately re-issue — that is how you get throttled and how you
generate the "too many messages to the same phone number" asynchronous failure
(code `1067`).

There is **no idempotency key**. A retried `login_start` is a second passcode delivered
and a second message billed.
