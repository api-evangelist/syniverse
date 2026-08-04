# Syniverse (syniverse)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Syniverse is a United States headquartered wholesale telecommunications company that sits between mobile network operators rather than in front of consumers. Its mobility business runs inter-carrier roaming, IPX network and signaling, 5G SEPP, clearing and settlement, global SIM and satellite roaming for carriers, and none of that carrier-side estate is exposed as a public API. Its messaging business is the opposite: an A2P/CPaaS aggregator selling SMS, MMS, RCS, WhatsApp, push, voice, 10DLC campaign registration, messaging trust/anti-spam and mobile identity services to enterprises, and it is published through a genuine self-serve developer portal, the Syniverse Developer Community (SDC), where anyone can sign up for a free API key, read the docs, and download eleven Redoc-rendered OpenAPI/Swagger definitions. On network APIs Syniverse is a channel, not a source — it joined GSMA Open Gateway in April 2025 as a channel partner and announced an integration with Aduna in June 2025, but no CAMARA-defined API is documented or callable on its developer portal today.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/syniverse/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/syniverse/refs/heads/main/apis.yml)

## Tags

- Telecommunications
- United States
- CPaaS
- Messaging
- SMS
- Roaming
- IPX
- Wholesale
- Identity Verification
- SIM Swap
- 10DLC
- Open Gateway
- Network APIs
- Aggregator

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## Developer Surface

- **Developer portal:** [https://developer.syniverse.com/](https://developer.syniverse.com/) — HTTP 200, a real self-serve portal ("GET YOUR FREE API KEY", self signup, public US test channel in the quickstart), not a login wall.
- **Documentation:** [https://sdcdocumentation.syniverse.com/](https://sdcdocumentation.syniverse.com/) — HTTP 200, product guides plus eleven downloadable OpenAPI/Swagger definitions rendered through Redoc.
- **Gateway:** `https://api.syniverse.com` — live; an anonymous call to `/scg-external-api/api/v1/messaging/message_requests` returns HTTP 401.
- **Auth:** Bearer access token issued per registered SDC application. OAuth2 client-credentials only on the Messaging Trust family. **No CIBA anywhere.**
- **SDKs:** [github.com/Syniverse](https://github.com/Syniverse) — source-only SDKs for Node.js, PHP, Python, Java, C++, C#, Ruby and Objective-C push; mostly last touched 2017–2021.

## CAMARA / GSMA Open Gateway posture

Syniverse is not a mobile network operator, so it is not a CAMARA API source. It joined **GSMA Open Gateway as a channel partner** on 29 April 2025 and announced an integration with **Aduna** on 18 June 2025, with SIM Swap, Number Verification, Device Location and Quality on Demand named in the announcements.

Nothing CAMARA-shaped is callable. There is no CAMARA product page, no CAMARA spec in the documentation, `opengateway.syniverse.com` does not resolve, and `/opengateway` on the corporate site is a 404. Grepping all eleven harvested specs for `camara`, `CIBA`, `backchannel`, `open gateway`, `sim swap`, `number verification`, `device location` and `quality on demand` returns zero matches. A press release is not an implementation.

What Syniverse does ship is its own, older, functionally adjacent identity family — Account Takeover Detection (`POST /v1/simCheck`), Right Party Verification (`POST /v2/match`) and Phone Number Verification (`POST /lookup`) — which are Syniverse-proprietary contracts, not CAMARA specifications.

## APIs

### Syniverse Omni-Channel Messaging API (SCG)

The Syniverse Communication Gateway (SCG) API — 72 paths and 148 operations covering message requests, messages, scheduled messages, templates, sender IDs, channels, contacts, attachments, keywords, calls, bridges and conferences for SMS, MMS, RCS, Facebook Messenger, WeChat, WhatsApp, push and voice.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/omni-channel/user-guides/sms-mms-user-guide](https://sdcdocumentation.syniverse.com/index.php/omni-channel/user-guides/sms-mms-user-guide)
- **Base URL:** `https://api.syniverse.com/scg-external-api/api/v1`
- [OpenAPI](openapi/syniverse-omni-channel-messaging-openapi.yml) (Swagger 2.0)

### Syniverse Multi-Factor Authentication API

Generates and validates one-time passwords and delivers tokens over SMS, voice or push.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/multi-factor-authentication/getting-started/multi-factor-authentication-overview](https://sdcdocumentation.syniverse.com/index.php/multi-factor-authentication/getting-started/multi-factor-authentication-overview)
- **Base URL:** `https://api.syniverse.com/scg-external-api/api/v1`
- [OpenAPI](openapi/syniverse-multi-factor-authentication-openapi.yml) (Swagger 2.0)

### Syniverse Phone Number Verification API

Single-number lookup returning current carrier, line type and status so an enterprise can detect ported, deactivated, disconnected or reassigned numbers.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/phone-number-verification/getting-started/phone-number-verification-overview](https://sdcdocumentation.syniverse.com/index.php/phone-number-verification/getting-started/phone-number-verification-overview)
- **Base URL:** `https://api.syniverse.com/numberidentity/v3`
- [OpenAPI](openapi/syniverse-phone-number-verification-openapi.yml) (OpenAPI 3.0.0)

### Syniverse Right Party Verification API

`POST /v2/match` returns identity match scores against Syniverse-sourced mobile data. Published definition points at a sandbox host.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/right-party-verification/getting-started/right-party-verification-overview](https://sdcdocumentation.syniverse.com/index.php/right-party-verification/getting-started/right-party-verification-overview)
- **Base URL:** `https://mobileidsandbox.syniverse.com/cigateway/id`
- [OpenAPI](openapi/syniverse-right-party-verification-openapi.yml) (OpenAPI 3.0.0)

### Syniverse Account Takeover Detection API

`POST /v1/simCheck` reports recent SIM changes or call forwarding on an end user's mobile channel as an account-takeover risk signal.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/account-takeover-detection/getting-started/account-takeover-detection-overview](https://sdcdocumentation.syniverse.com/index.php/account-takeover-detection/getting-started/account-takeover-detection-overview)
- **Base URL:** `https://mobileidsandbox.syniverse.com/cigateway/id`
- [OpenAPI](openapi/syniverse-account-takeover-detection-openapi.yml) (OpenAPI 3.0.0)

### Syniverse Messaging Trust Resolve API

Resolves inbound text and MMS traffic to a spam/abuse disposition. OAuth2 client credentials.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/messaging-trust/getting-started/messaging-trust-overview](https://sdcdocumentation.syniverse.com/index.php/messaging-trust/getting-started/messaging-trust-overview)
- **Base URL:** `https://api.mt1.messaging-trust.syniverse.com/api`
- [OpenAPI](openapi/syniverse-messaging-trust-resolve-openapi.yml) (OpenAPI 3.0.1)

### Syniverse Messaging Trust Spam Datafeed API

`POST /spam/report` submits spam reports into the Messaging Trust datafeed.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/messaging-trust/getting-started/messaging-trust-overview](https://sdcdocumentation.syniverse.com/index.php/messaging-trust/getting-started/messaging-trust-overview)
- **Base URL:** `https://api.mt1.messaging-trust.syniverse.com/api/v1`
- [OpenAPI](openapi/syniverse-messaging-trust-datafeed-openapi.yml) (OpenAPI 3.0.1)

### Syniverse 10DLC API

US 10-digit long code campaign provisioning — campaigns, long codes, number pools and AT&T DCA campaigns. Version 2.3.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/10-dlc/getting-started/10-dlc-overview](https://sdcdocumentation.syniverse.com/index.php/10-dlc/getting-started/10-dlc-overview)
- **Base URL:** `https://api.syniverse.com/engage/tendlc-services/v2`
- [OpenAPI](openapi/syniverse-10dlc-openapi.yml) (OpenAPI 3.0.1)

### Syniverse 10DLC Number Pool API (v1, deprecated)

Still-published v1 of 10DLC provisioning, under an explicitly deprecated documentation section.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/10-dlc/api-version-1-deprecated/explore-api-reference](https://sdcdocumentation.syniverse.com/index.php/10-dlc/api-version-1-deprecated/explore-api-reference)
- **Base URL:** `https://api.syniverse.com/engage/tendlc-services/v1`
- [OpenAPI](openapi/syniverse-10dlc-number-pool-openapi.yml) (Swagger 2.0)

### Syniverse Whitelisting Service API

Developer Community gateway service for whitelisting entities and transactions, used for trial-account destination numbers and similar allow lists.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/developer-community-gateway-services/api-reference/explore-whitelist-api-reference](https://sdcdocumentation.syniverse.com/index.php/developer-community-gateway-services/api-reference/explore-whitelist-api-reference)
- [OpenAPI](openapi/syniverse-whitelisting-service-openapi.json) (Swagger 2.0)

### Syniverse SDC Application Access Token Management API

`GET /saop-rest-data/v1/apptoken-refresh` regenerates the access token bound to an SDC application.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/developer-community-gateway-services/api-reference/explore-token-management-api-reference](https://sdcdocumentation.syniverse.com/index.php/developer-community-gateway-services/api-reference/explore-token-management-api-reference)
- [OpenAPI](openapi/syniverse-token-management-openapi.yml) (OpenAPI 3.0.0)

### Syniverse Event Subscription Service API (Event Manager)

The webhook and event layer — topics, topic-subscriptions, delivery-configurations, event-types, event-deliveries and event-buffer-files, driving scheduled or near-real-time REST delivery of delivery receipts and phone-number status changes. Documented as a rendered RAML console; no downloadable machine-readable definition was found.

- **Human URL:** [https://sdcdocumentation.syniverse.com/index.php/reporting/event-manager/overview](https://sdcdocumentation.syniverse.com/index.php/reporting/event-manager/overview)
- **Base URL:** `https://api.syniverse.com/ess/v1`

## Links

- [Website](https://www.syniverse.com/)
- [Developer Portal](https://developer.syniverse.com/)
- [Documentation](https://sdcdocumentation.syniverse.com/)
- [GitHub](https://github.com/Syniverse)
- [Support](https://sdcsupport.syniverse.com/hc/en-us)
- [Blog](https://www.syniverse.com/blog)
- [News](https://www.syniverse.com/news-and-events)
- [LinkedIn](https://www.linkedin.com/company/syniverse)
