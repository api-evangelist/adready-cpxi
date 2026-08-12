---
name: Build and submit a media plan
description: >-
  Create a media plan for an advertiser on the Digital Remedy Platform, shape its allocations across
  product types, product categories and display devices, set its targeting level, then export or
  submit it as an order.
api: openapi/adready-cpxi-kickstart-openapi.yml
operations:
  - getByAccount
  - getAdvertiser
  - getAll_1
  - create_3
  - get_3
  - updatePlan
  - changeProductTypeAllocation
  - changeProductCategoryAllocation
  - changeDisplayDeviceAllocation
  - changeTargetingLevel
  - updateFlag
  - exportOrder
  - sendSubmitOrderNotification
  - delete_3
generated: '2026-08-12'
method: generated
source: >-
  Grounded in openapi/adready-cpxi-kickstart-openapi.yml (harvested from
  https://platform.digitalremedy.com/v3/api-docs). Every operationId was verified verbatim in the
  spec. The provider publishes no prose documentation for these operations.
---

# Build and submit a media plan

Run **Establish and maintain a Digital Remedy Platform session** first. Every step here returns `401`
without a session.

## A warning about this operation set

Most operations in the plans controller carry **no summary and no description** in the published
description — their operationIds are springdoc method-name defaults (`create_3`, `get_3`, `getAll_1`,
`delete_3`). The path tells you the intent; the contract does not. Request bodies are typed against
schemas, but the semantics of allocation and targeting changes are undocumented. **Work against a
non-production advertiser first and verify each mutation by re-reading the plan.**

## Steps

1. **Locate the advertiser.** `GET /api/advertisers` (`getByAccount`) lists advertisers for the
   account, then `GET /api/advertisers/{advertiserId}` (`getAdvertiser`) for detail.

2. **List existing plans.** `GET /api/plans` (`getAll_1`). Paginate with `pageNumber`, `perPage`,
   `sortKey` and `sortOrder`. The paged response is `PageResponsePlan`:
   `{content[], first, last, pageNumber, pageSize, totalElements, totalPages}`. No default or maximum
   `perPage` is documented — request a modest page size and read `totalPages`.

3. **Create the plan.** `POST /api/plans` (`create_3`), body typed as `Plan` (51 properties).
   Capture the returned plan id.

4. **Read it back.** `GET /api/plans/{planId}` (`get_3`) → `ApiResponsePlan`. Do this after every
   mutation; there is no idempotency and no way to confirm a write from the response envelope alone.

5. **Shape the plan.** Each of these is a `PUT` that changes one dimension of the allocation:
   - `PUT /api/plans/{planId}/changeProductTypeAllocation` (`changeProductTypeAllocation`)
   - `PUT /api/plans/{planId}/changeProductCategoryAllocation` (`changeProductCategoryAllocation`)
   - `PUT /api/plans/{planId}/changeDisplayDeviceAllocation` (`changeDisplayDeviceAllocation`)
   - `PUT /api/plans/{planId}/changeTargetingLevel` (`changeTargetingLevel`)

   These are RPC-shaped actions, not resource updates. Apply them one at a time and re-read the plan
   between calls — the interaction between allocation dimensions is not described.

6. **Update plan fields.** `PUT /api/plans/{planId}` (`updatePlan`) for whole-plan edits;
   `PUT /api/plans/{planId}/updateFlag` (`updateFlag`) for the flag field.

7. **Export or submit.**
   - `POST /api/plans/{planId}/export` (`exportOrder`) produces the export.
   - `POST /api/plans/{planId}/sendSubmitOrderNotification` (`sendSubmitOrderNotification`) sends the
     submit-order notification.
   - **Do not use `POST /api/plans/{planId}/createOrder` (`createOrder`) — it is flagged
     `deprecated: true` in the description, with no replacement named and no sunset date.** Treat it
     as removable without warning; the provider publishes no deprecation policy.

8. **Notifications are side effects, not confirmations.** `emailNotification`,
   `workflowEmailNotification`, `updateStatusEmailNotification` and `changeAppliedEmailNotification`
   all send email to real people. Never call them while exploring the API.

9. **Delete only deliberately.** `DELETE /api/plans/{planId}` (`delete_3`).

## Related

- Line items and Xandr push: `adready-cpxi-manage-line-items.md`
- Data model: `data-model/adready-cpxi-data-model.yml`
- Conventions and pagination: `conventions/adready-cpxi-conventions.yml`
