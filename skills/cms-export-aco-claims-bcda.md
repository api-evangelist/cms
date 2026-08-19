---
name: cms-export-aco-claims-bcda
description: >-
  Export Medicare Part A, B and D claims for the enrollees attributed to a Medicare ACO or alternative
  payment model entity, using the CMS Beneficiary Claims Data API (BCDA) Bulk FHIR workflow. Handles
  the token exchange, the asynchronous kickoff-poll-download cycle, the 24-hour output expiry, and the
  concurrency limit.
api: CMS Beneficiary Claims Data API (BCDA)
spec: openapi/cms-bcda-openapi.yml
operations:
  - GetAuthToken
  - bulkGroupRequestV2
  - bulkPatientRequestv2
  - jobStatusidv2
  - jobsStatusV2
  - deleteJobv2
  - serveData
  - attributionStatusv2
generated: '2026-08-15'
method: generated
source: openapi/cms-bcda-openapi.yml, https://bcda.cms.gov/api-documentation.html
---

# Export ACO claims from BCDA

BCDA returns a **job**, not data. Every successful export call is asynchronous. Budget minutes to
hours, and download inside the 24-hour window or the files are gone.

## Preconditions

- Production requires being an eligible model entity (a Medicare ACO or alternative payment model
  participant) with credentials managed in 4i / ACO-MS. Anyone can run this whole flow against the
  sandbox with synthetic data.
- BCDA v3 is current. **v1 and v2 are removed on 2027-07-30** — target v2 or v3 operations, never v1,
  and note that no response header warns you about this. See `lifecycle/cms-lifecycle.yml`.

## Steps

1. **Get a bearer token.** `GetAuthToken` — `POST /auth/token` with your client id and secret as HTTP
   Basic credentials. A 400 here means missing credentials, not a bad request body.

2. **Check attribution freshness (optional but cheap).** `attributionStatusv2` —
   `GET /api/v2/attribution_status` returns when your enrollee list and runout files were last
   refreshed. Attribution updates monthly; there is no point re-exporting between refreshes unless you
   are pulling new claims with `_since`.

3. **Kick off the export.** Choose one:
   - `bulkGroupRequestV2` — `GET /api/v2/Group/{groupId}/$export`. Use `groupId=all` for currently
     attributed enrollees, or `groupId=runout` for enrollees attributed during the previous year with
     service dates no later than 31 December of that year.
   - `bulkPatientRequestv2` — `GET /api/v2/Patient/$export` for all currently attributed enrollees.

   Apply the parameters, because the defaults are expensive:
   - `_type` — restrict to the resource types you need (ExplanationOfBenefit, Patient, Coverage, and
     on v2+ Claim and ClaimResponse for partially adjudicated data).
   - `_since` — **without this BCDA returns claims back to 2014.** With it you get resources updated
     after the instant supplied, plus everything for newly attributed enrollees, which is exactly the
     incremental pull you want on a schedule.

   A success is **HTTP 202** with a `Content-Location` header carrying the job status URL. Save that
   URL — the job UUID in it is the only handle you can quote to CMS support.

4. **Handle 429 correctly.** A 429 here does **not** mean you are calling too fast. BCDA allows one
   in-flight export per resource type per model entity. The body is a FHIR OperationOutcome saying a
   job of this resource type is already in progress. Do not back off and retry — either wait for the
   running job, or cancel it with `deleteJobv2` and re-submit. If you have lost the job id, list jobs
   with `jobsStatusV2` (`GET /api/v2/jobs`), which returns FHIR Task resources.

5. **Poll for completion.** `jobStatusidv2` — `GET /api/v2/jobs/{jobId}`. While running it returns 202
   with an `X-Progress` header. On completion it returns 200 with a manifest of NDJSON file URLs.
   Poll every few minutes, not every few seconds.

6. **Download the files.** `serveData` — `GET /data/{jobId}/{filename}` for each entry in the
   manifest. Output is `application/fhir+ndjson`: one FHIR resource per line, so stream it rather than
   parsing the whole file into memory.

7. **Respect the expiry.** Completed job output expires **24 hours** after completion. After that both
   the job status and the data URLs return **HTTP 410 Gone**. There is no extension and no recovery
   other than re-running the export.

## Error handling

All errors return a FHIR `OperationOutcome` in `application/fhir+json` — **not** RFC 9457
problem+json. Full catalog in `errors/cms-problem-types.yml`.

| Status | Meaning | What to do |
|---|---|---|
| 400 | Malformed request | Check `_type` / `_since` and the resource path; the OperationOutcome names the parameter |
| 401 | Credentials invalid for the resource | Re-request a token; do not retry the old one |
| 404 | Path or job not found | Confirm the job UUID from `Content-Location` |
| 410 | Output expired | Re-run the export; the 24-hour window has closed |
| 429 | Export already in progress | Wait or cancel — this is concurrency, not rate |
| 500 | Server error | Retry the kickoff |

## What this skill will not do

- There is **no idempotency key**. If you are unsure whether a kickoff succeeded, list jobs with
  `jobsStatusV2` before submitting again; do not blind-retry.
- There is **no per-patient request**. BCDA deliberately has no way to ask for one enrollee's claims —
  the unit of access is the attributed population.
