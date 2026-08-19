---
name: cms-search-marketplace-plans
description: >-
  Find Qualified Health Plans for a household on the Federally-Facilitated Marketplace using the
  Healthcare.gov Marketplace API — resolve geography, estimate APTC/CSR eligibility, search plans, and
  check whether a household's doctors and prescriptions are covered.
api: Healthcare.gov Marketplace API
spec: >-
  openapi/cms-geography-api-openapi.yml, openapi/cms-households-eligibility-api-openapi.yml,
  openapi/cms-insurance-plans-api-openapi.yml, openapi/cms-provider-drug-coverage-api-openapi.yml,
  openapi/cms-insurance-issuers-api-openapi.yml, openapi/cms-api-reference-api-openapi.yml
operations:
  - 'GET /counties/by/zip/{zipcode}'
  - 'GET /market-years'
  - 'POST /households/eligibility/estimates'
  - 'POST /plans/search'
  - 'POST /plans/search/stats'
  - 'GET /plans/{plan_id}'
  - 'GET /plans/{plan_id}/quality-ratings'
  - 'GET /providers/search'
  - 'GET /providers/covered'
  - 'GET /drugs/search'
  - 'GET /drugs/covered'
  - 'GET /issuers/{issuer_id}'
generated: '2026-08-15'
method: generated
source: >-
  openapi/cms-*-api-openapi.yml (Marketplace family), https://developer.cms.gov/marketplace-api/,
  https://developer.cms.gov/marketplace-api/key-request.html
---

# Search Marketplace plans for a household

Base URL `https://marketplace.api.healthcare.gov/api/v1`.

> **Operation ids do not exist on this API.** Unlike the CMS claims APIs, no operation in the
> Marketplace OpenAPI declares an `operationId`, so every reference below is by **method + path**. If
> you are generating a client, expect the generator to invent names.

## Authentication

An API key in the **`apikey` query parameter** — not a header. Request one at
<https://developer.cms.gov/marketplace-api/key-request.html>. It is free.

Two things will bite you:
- **Keys expire every 60 days**, auto-renewed by an email notification. An expired key returns **401**,
  not 429. Handle key rotation as a first-class case.
- The key travels in the query string, so it lands in every proxy and access log on the path. Do not
  reuse a Marketplace key anywhere else.

## Steps

1. **Pin the data year.** `GET /market-years` returns the market years the API serves; `GET /versions`
   reports the state of the API. Marketplace data is annual — plans, rate areas and poverty guidelines
   all change on 1 January, so decide the year before anything else.

2. **Resolve geography.** `GET /counties/by/zip/{zipcode}` returns the counties a ZIP maps to. A ZIP
   can span multiple counties with **different rate areas and different plan availability**, so if you
   get more than one back you must ask the household which county they live in rather than picking the
   first. Supporting lookups: `GET /states/{abbrev}`, `GET /states/{abbrev}/medicaid`,
   `GET /states/{abbrev}/poverty-guidelines`, `GET /rate-areas`.

3. **Estimate eligibility.** `POST /households/eligibility/estimates` with the household — its
   `place` (address / county fips / zip), its `people` (age, gender, tobacco use, relationship), and
   income. Returns APTC (premium tax credit), CSR (cost-sharing reduction) eligibility and Medicaid /
   CHIP indications. Related benchmarks: `POST /households/slcsp` (second-lowest-cost silver plan, the
   APTC benchmark), `POST /households/lcbp` (lowest-cost bronze), `POST /households/lcsp` (lowest-cost
   silver), `POST /households/ichra` (individual coverage HRA affordability),
   `GET /households/pcfpl` (percent of federal poverty level).

4. **Search plans.** `POST /plans/search` with the household and a filter (metal level, plan type,
   issuer, premium and deductible ranges). `POST /plans/search/stats` returns aggregate statistics for
   the same filter — use it to show "42 plans from 6 issuers, premiums $180–$740" before rendering a
   list. Detail: `GET /plans/{plan_id}`; quality: `GET /plans/{plan_id}/quality-ratings`.

5. **Check the household's doctors.** `GET /providers/autocomplete` to resolve a name to an NPI, then
   `GET /providers/covered` with the NPIs and plan ids to see which plans have them in network. The NPI
   is the join key to the NPPES NPI Registry API if you need to verify the provider record itself.

6. **Check the household's prescriptions.** `GET /drugs/autocomplete` to resolve a drug name to an
   RxCUI, then `GET /drugs/covered` with the RxCUIs and plan ids for formulary coverage. Aggregate
   coverage statistics: `GET /coverage/stats`, `GET /coverage/search`.

7. **Look up the carrier.** `GET /issuers/{issuer_id}`.

## Bulk alternative

If you need the whole dataset rather than a household-specific answer, the Bulk Data API serves whole
files: `GET /data/apt`, `/data/decile-mapping`, `/data/state-medicaid`, `/data/rate-areas`,
`/data/county-zips`, `/data/crosswalk`. Pull these once per market year and cache them rather than
calling the geography endpoints per request.

## Error handling

Errors are a Marketplace `ApplicationError` JSON object — **not** FHIR OperationOutcome and not RFC
9457 problem+json. Schemas in `json-schema/marketplace-applicationerror400.json` and
`json-schema/marketplace-applicationerror404.json`.

| Status | Meaning |
|---|---|
| 400 | Invalid request — commonly an unsupported market year |
| 401 | Missing, invalid or **expired** apikey |
| 404 | No data for the requested code or identifier |
| 500 | Server error |

Rate limits exist and are signalled in response headers, but CMS publishes no numbers and names no
header. Read whatever headers come back rather than assuming `X-RateLimit-*`. See
`rate-limits/cms-rate-limits.yml`.
