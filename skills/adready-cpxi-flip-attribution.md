---
name: Set up Flip pixels and read attribution
description: >-
  Create and manage Flip attribution pixels for an advertiser on the Digital Remedy Platform, then
  read pixel fires, conversion events, rollup touch reports and performance measurement.
api: openapi/adready-cpxi-kickstart-openapi.yml
operations:
  - getFlipPixels
  - addFlipPixel
  - fetchFlipPixelDetails
  - updateFlipPixel
  - fetchPixelCategories
  - fetchPixelFires
  - fetchDailyPixelFires
  - fetchFlipPixelFiresSummary
  - getConversionEvents
  - getPixelSummary
  - getPixelDistribution
  - getDailyConversions
  - getLastTouches
  - getAllTouchesForConversion
  - getEventPerformanceReport
  - getPerformanceReport
generated: '2026-08-12'
method: generated
source: >-
  Grounded in openapi/adready-cpxi-kickstart-openapi.yml (harvested from
  https://platform.digitalremedy.com/v3/api-docs). These are among the few operations in this API that
  carry provider-written summaries; every operationId was verified verbatim in the spec.
---

# Set up Flip pixels and read attribution

Run **Establish and maintain a Digital Remedy Platform session** first. Everything here is scoped to a
single `advertiserId` — it is the most-used path parameter in the whole API (101 operations).

## Steps

### Pixels

1. **List pixels.** `GET /api/advertisers/{advertiserId}/flipPixels` (`getFlipPixels`) —
   *Get FLIP pixels by advertiser id*.
2. **Create a pixel.** `POST /api/advertisers/{advertiserId}/flipPixels` (`addFlipPixel`) —
   *Add new FLIP pixel for an advertiser*.
3. **Read one.** `GET /api/advertisers/{advertiserId}/flipPixels/{pixelId}` (`fetchFlipPixelDetails`).
4. **Update.** `PUT /api/advertisers/{advertiserId}/flipPixels/{pixelId}` (`updateFlipPixel`) — this
   is one of only three operations in the entire API that documents a `404`, so a missing pixel is at
   least detectable here.
5. **Categories.** `GET /api/advertisers/{advertiserId}/flipPixels/{pixelId}/categories`
   (`fetchPixelCategories`).

### Pixel fires

6. `GET .../flipPixels/{pixelId}/fires` (`fetchPixelFires`) — raw fires.
7. `GET .../flipPixels/{pixelId}/fires/daily` (`fetchDailyPixelFires`) — daily series.
8. `GET .../flipPixels/{pixelId}/fires/summary` (`fetchFlipPixelFiresSummary`) — summary.

Most reporting operations take `startDate` and `endDate` query parameters (59 operations do), and many
also accept `compareStartDate` / `compareEndDate` for period-over-period comparison (16 operations).
No date format is stated in the description — confirm it against a known-good response before relying
on a range.

### Conversions and attribution

9. **Conversion events.** `GET /api/advertisers/{advertiserId}/events` (`getConversionEvents`) —
   *Get conversion events for an advertiser*.
10. **Conversion summary and distribution.**
    `POST /api/advertisers/{advertiserId}/conversions/summary` (`getPixelSummary`),
    `POST /api/advertisers/{advertiserId}/conversions/distribution` (`getPixelDistribution`),
    `POST /api/advertisers/{advertiserId}/conversions/daily` (`getDailyConversions`).
11. **Rollup touches.**
    `POST /api/advertisers/{advertiserId}/reports/rollup` (`getLastTouches`) —
    *Get rollup report with only last touches for each conversion*; then
    `POST /api/advertisers/{advertiserId}/reports/rollup/touches` (`getAllTouchesForConversion`) —
    *Get all touches (except last touch) for a conversion*. Read both to reconstruct a full path;
    neither returns the whole picture alone.

### Performance reporting

12. `POST /api/advertisers/{advertiserId}/reports/performance` (`getPerformanceReport`) and
    `POST /api/advertisers/{advertiserId}/reports/eventPerformance` (`getEventPerformanceReport`) —
    dimension-wise reports by creative, publisher, channel and daypart.
13. `POST /api/advertisers/{advertiserId}/performance/daily` (`getDailyPerformance`) — daily
    CPM/CPA/ROAS/VCR/CTR.
14. Reach: `POST /api/advertisers/{advertiserId}/performance/metrics/reach`
    (`getReachFrequencyMetric`) and
    `GET /api/advertisers/{advertiserId}/performance/metrics/incrementalreach` (`getIncrementalReach`).

Note that several reporting operations are duplicated under an `/mdsp/` prefix (`getDailyImpressions`,
`getImpressionsSummary`, `getImpressionsDistribution`, `getDailyPerformance_1`) alongside their
non-prefixed twins (`getDailyImpressions_1`, `getImpressionsSummary_1`, `getImpressionsDistribution_1`,
`getDailyPerformance`). The description does not say which data source each reads. Pick one family and
stay in it rather than mixing them in a single analysis.

## Reading the results

Reports come back wrapped in `ApiResponse {status, message, result}`. Paged analytics use a third
shape, `PagedResult` — `{records[], recordCount, headers[]}` — which is not the same as the
`PageResponse` shape used elsewhere. Detect the shape before parsing.

## Related

- Brand lift and incrementality operations live alongside these under
  `/api/advertisers/{advertiserId}/brand_lift/*` and `/api/advertisers/{advertiserId}/incrementality/*`.
- Conventions: `conventions/adready-cpxi-conventions.yml`
