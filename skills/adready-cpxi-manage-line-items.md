---
name: Manage insertion-order line items and push to Xandr
description: >-
  Create and maintain line items under an insertion order on the Digital Remedy Platform, calculate
  CPM and price spend, track workflow status, and push a line item to Xandr for activation.
api: openapi/adready-cpxi-kickstart-openapi.yml
operations:
  - index
  - create_8
  - getLineItem
  - update_6
  - calculateCpm
  - calculatePriceSpend
  - updateFlag_1
  - pushToXandr
  - lineItemStatusCount
  - workflowStatusCount
  - getInsertionOrderNames
  - getInsertionOrderPacing
  - uploadFile
  - delete_6
generated: '2026-08-12'
method: generated
source: >-
  Grounded in openapi/adready-cpxi-kickstart-openapi.yml (harvested from
  https://platform.digitalremedy.com/v3/api-docs). Every operationId was verified verbatim in the spec.
---

# Manage insertion-order line items and push to Xandr

Run **Establish and maintain a Digital Remedy Platform session** first.

## The one operation to be careful with

`PUT /api/ios/{insertionOrderId}/lineItems/{lineItemId}/pushToXandr` (`pushToXandr`) hands the line
item to Xandr, an external ad-serving platform. **This is the point where an action in this API
becomes real money in an external system.** It has no idempotency key, no documented response
semantics beyond the generic envelope, and no documented way to reverse it. Require human
confirmation before calling it, and never call it as part of an exploratory or retry loop. The
agentic-access classification in `agentic-access/adready-cpxi-agentic-access.yml` marks operations of
this shape as requiring a human in the loop for exactly this reason.

## Steps

1. **Find the insertion orders.** `GET /api/advertisers/{advertiserId}/io_names`
   (`getInsertionOrderNames`) returns unique insertion order names for an advertiser.
   `POST /api/advertisers/{advertiserId}/io_pacing` (`getInsertionOrderPacing`) returns overall pacing
   across selected insertion orders — use it to understand delivery before you change anything.

2. **List line items.** `GET /api/ios/{insertionOrderId}/lineItems` (`index`) for one insertion order,
   or `GET /api/ios/lineItems` (`getAll_7`) across the account. Paginate with `pageNumber` / `perPage`
   / `sortKey` / `sortOrder`; the response shape is `PageResponseLineItem`.

3. **Read one line item.** `GET /api/ios/lineItem/{lineItemId}` (`getLineItem`).
   The `LineItem` schema is the largest entity in this API — 58 properties.

4. **Create.** `POST /api/ios/{insertionOrderId}/lineItems` (`create_8`).
   Capture the id and immediately re-read it: there is no idempotency, so a timed-out create must be
   resolved by listing, never by retrying.

5. **Price it.** Both are `POST` operations that compute rather than mutate:
   - `POST /api/ios/{insertionOrderId}/lineItems/{lineItemId}/calculateCpm` (`calculateCpm`)
   - `POST /api/ios/{insertionOrderId}/lineItems/{lineItemId}/calculatePriceSpend` (`calculatePriceSpend`)

6. **Update.** `PUT /api/ios/{insertionOrderId}/lineItems/{lineItemId}` (`update_6`) for the whole
   line item; `PUT /api/ios/{insertionOrderId}/lineItems/{lineItemId}/updateFlag` (`updateFlag_1`) for
   the flag.

7. **Attach files.** `POST /api/files/{orderId}/lineItems/{lineItemId}` (`uploadFile`).

8. **Track workflow.** `GET /api/ios/plan/lineItemStatusCount` (`lineItemStatusCount`),
   `GET /api/ios/plan/lineItemStatusCountForPlans` (`lineItemStatusCountForPlans`),
   `GET /api/ios/workflowStatusCount` (`workflowStatusCount`) and
   `GET /api/ios/plan/workflowStatsCount` (`workflowStatsCount`) return status rollups. The status
   values themselves are not enumerated anywhere in the description — discover them from live data.

9. **Push to Xandr** — only after human confirmation. See the warning above.

10. **Delete.** `DELETE /api/ios/{insertionOrderId}/lineItems/{lineItemId}` (`delete_6`).

## Do not call while exploring

`emailNotification_1`, `customEmailNotification` and `sendResubmitEmailNotification` all send email to
real recipients.

## Related

- Media planning: `adready-cpxi-build-media-plan.md`
- Agentic access contracts: `agentic-access/adready-cpxi-agentic-access.yml`
