---
name: Provision numbers on a US 10DLC campaign
description: >-
  Attach and detach long codes and number pools on a registered US 10DLC campaign, and
  suspend or resume it, using the Syniverse 10DLC v2.3 API.
api: openapi/syniverse-10dlc-openapi.yml
base_url: https://api.syniverse.com/engage/tendlc-services/v2
operations:
  - getCampaigns
  - getCampaign
  - create10DLCNumberV2
  - delete10DLCNumberV2
  - getApplicationAddress
  - campaignWithNumberPoolAssociationV2
  - campaignWithNumberPoolRemovalV2
  - suspendCampaign
  - resumeCampaign
  - getCampaign_1
  - getApplicationAddress_3
generated: '2026-07-25'
method: generated
---

# Provision numbers on a US 10DLC campaign

Every `operationId` above is verbatim from `openapi/syniverse-10dlc-openapi.yml`
(spec version 2.3, published 2025-12-04). Auth is the `SyniverseToken` scheme —
`http`/`bearer` with `bearerFormat: SDC`:

```
Authorization: Bearer {SDC application access token}
```

## 1. Find the campaign

- `getCampaigns` — `GET /engage/tendlc-services/v2/campaigns`. This is the one paged
  surface in the whole Syniverse estate: query params `page` (default `0`) and `size`
  (default `10`), response envelope `PageCampaignResponse`. Nothing else in the
  catalog paginates this way, so do not generalize it.
- `getCampaign` — `GET /engage/tendlc-services/v2/campaigns/{campaignId}`.
- `getCampaign_1` — `GET /engage/tendlc-services/v2/campaigns/att/{campaignId}` for the
  AT&T DCA view of a campaign.

## 2. Attach and detach long codes

- `create10DLCNumberV2` — `POST .../campaigns/{campaignId}/longcodes/{longcode}`
- `delete10DLCNumberV2` — `DELETE .../campaigns/{campaignId}/longcodes/{longcode}`
- `getApplicationAddress` — `GET .../campaigns/{campaignId}/longcodes/{longcode}`
- `getApplicationAddress_3` — `GET .../att/application-address/{applicationAddress}`

The spec publishes the exact 400 conditions. Treat them as validation rules, not as
retryable failures:

- `Invalid Longcode. Longcode must be a number.`
- `Invalid Longcode. Longcode must be 11 digits and must start with country code 1.`
- `Longcode cannot be added/deleted as it is already associated with another campaign.`
- `Longcode cannot be added to campaign. Limit exceeded`
- `Longcode cannot be added/deleted as Campaign is not Deployed.`
- `Request failed. Daily quota exceeded`

`Request failed. Daily quota exceeded` is a **daily** ceiling on provisioning calls
whose value Syniverse does not publish. Batch your provisioning; do not loop.

## 3. Attach and detach number pools

- `campaignWithNumberPoolAssociationV2` — `POST .../campaigns/{campaignId}/pools/{numberPoolId}/types/{numberPoolType}`
- `campaignWithNumberPoolRemovalV2` — `DELETE .../campaigns/{campaignId}/pools/{numberPoolId}`

Published 400 conditions include `Number Pool cannot be added as it is already
associated with another campaign`, `Number Pool cannot be added to campaign. Limit
exceeded`, `NNID invalid`, `Number Pool type is invalid` and `Number pool does not
match with campaign`.

## 4. Suspend and resume

- `suspendCampaign` — `POST .../campaigns/{campaignId}/suspend`
- `resumeCampaign` — `POST .../campaigns/{campaignId}/resume`

Both are destructive to live traffic. Several operations return `202 Accepted`
(`ApiAcceptedResponse`) rather than `200` — the change is queued, not applied. Re-read
with `getCampaign` before you assume the state changed. `409 Conflict` on these
operations means the change is already in flight or already applied
(`ApiProcessedAlreadyResponse`).

## Do not use v1

`openapi/syniverse-10dlc-number-pool-openapi.yml` (`create10DLCNumberUsingPOST`,
`delete10DLCNumberUsingDELETE`, `campaignWithNumberPoolAssociationUsingPOST`,
`campaignWithNumberPoolRemovalUsingDELETE`) is published under an explicitly
**deprecated** documentation section. No sunset date is published, but new
integrations belong on v2. See `lifecycle/syniverse-lifecycle.yml`.
