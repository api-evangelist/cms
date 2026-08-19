---
name: cms-authorize-beneficiary-bluebutton
description: >-
  Obtain a Medicare enrollee's consent through the CMS Blue Button 2.0 OAuth 2.0 flow and read their
  own claims data as FHIR R4. Covers mandatory PKCE, the FHIR scope set, token and grant expiry,
  enrollee revocation, and the case where the enrollee blocks access to personal information.
api: CMS Blue Button 2.0 API
spec: >-
  openapi/cms-patient-api-openapi.yml, openapi/cms-coverage-api-openapi.yml,
  openapi/cms-explanationofbenefit-api-openapi.yml, openapi/cms-capability-api-openapi.yml
operations:
  - readPatient
  - searchPatient
  - searchCoverage
  - readCoverage
  - searchExplanationOfBenefit
  - readExplanationOfBenefit
  - getCapabilityStatement
generated: '2026-08-15'
method: generated
source: >-
  https://bluebutton.cms.gov/api-documentation/authorization/,
  https://bluebutton.cms.gov/api-documentation/calling-the-api/,
  https://api.bluebutton.cms.gov/.well-known/openid-configuration, openapi/cms-*-api-openapi.yml
---

# Authorize a Medicare enrollee and read their claims (Blue Button 2.0)

Blue Button 2.0 is the only CMS claims API driven by **individual consent** rather than by programme
attribution. The enrollee authenticates at Medicare.gov and grants your application access to their
own data, and they can take it away at any time.

## Endpoints

| Environment | Base FHIR URL | OAuth base |
|---|---|---|
| Sandbox | `https://sandbox.bluebutton.cms.gov/v2/fhir/` | `https://sandbox.bluebutton.cms.gov/v2/o/` |
| Production | `https://api.bluebutton.cms.gov/v2/fhir/` | `https://api.bluebutton.cms.gov/v2/o/` |

Discovery: `https://api.bluebutton.cms.gov/.well-known/openid-configuration` (authorize, token,
revoke, userinfo, and a SMART `fhir_metadata_uri`).

## Client requirements — these are hard constraints

- **Confidential clients only.** Public client types are not supported.
- **Authorization code grant only.** The implicit grant is not supported.
- **PKCE is mandatory**, `S256` only: `code_challenge = BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))`.
  Send `code_challenge` and `code_challenge_method=S256` on the authorization request.
- For native and mobile apps, CMS requires a **Backend-For-Frontend proxy**: all code and refresh-token
  exchanges happen on your server, never on the device.

## Scopes

| Scope | Grants |
|---|---|
| `patient/Patient.rs` (or `.read`) | Read and search the enrollee's demographic information |
| `patient/Coverage.rs` (or `.read`) | Read and search their Medicare and supplemental coverage |
| `patient/ExplanationOfBenefit.rs` (or `.read`) | Read and search their Medicare claims |
| `launch/patient` | SMART patient launch context |
| `openid` | Identify the logged-in user |
| `profile` | Access `/v2/connect/userinfo` |

All read-only. There is no write scope anywhere in the CMS estate. Full detail in
`scopes/cms-scopes.yml`.

## Steps

1. **Send the enrollee to authorize.** `GET {oauth_base}/authorize/` with `client_id`, `redirect_uri`,
   `response_type=code`, URL-encoded `scope` (use `%20` between scopes), a `state` with **at least 122
   bits of entropy**, `code_challenge` and `code_challenge_method=S256`. Optional `lang=en|es`.
   POST-style authorization is also supported for SMART App Launch compliance.

2. **Exchange the code for a token.** `POST {oauth_base}/token/` with the code, your client
   credentials and the `code_verifier`. You get an access token valid for **1 hour** plus a refresh
   token.

3. **Read the data.** Against the base FHIR URL:
   - `readPatient` / `searchPatient` — `GET /Patient` returns a bundle with a single Patient entry;
     take `Bundle.entry.resource.id` for later queries. In sandbox synthetic data this id is a
     **negative number**.
   - `searchCoverage` — `GET /Coverage?beneficiary={id}`.
   - `searchExplanationOfBenefit` — `GET /ExplanationOfBenefit?patient={id}`. Synthetic EOB ids look
     like `carrier--10114937820`.
   - `getCapabilityStatement` — `GET /metadata` for the FHIR R4 CapabilityStatement.

   Page through results with `Bundle.link` (`next` / `previous`), not offset arithmetic.

4. **Refresh before expiry.** `POST {oauth_base}/token/` with the refresh token.

5. **Revoke on user request.** `POST {oauth_base}/revoke_token/`.

## Handle the blocked-personal-information case

Two independent things can leave you unable to call `/Patient` or `/UserInfo` even with the scope
granted: you declined to collect enrollee personal information at production approval, or the enrollee
chose during Medicare.gov authentication not to share it. **Design for this** — CMS's own guidance is
to take the patient id from the initial authorization response, or from the `ExplanationOfBenefit` or
`Coverage` bundles, rather than from `/Patient`.

## Error handling

| Status | Body | Meaning |
|---|---|---|
| 400 | `{"error":"invalid_grant"}` | The data-access authorization expired; the enrollee must re-authenticate and re-consent |
| 403 | FORBIDDEN | Client credentials or permissions problem |
| 404 | Data Access Grant was not found | The enrollee has not granted (or has revoked) access to this patient id |

## Testing expiry without waiting

The sandbox exposes `POST /v2/o/expire_authenticated_user/{patientId}/` (client id + secret as Basic
auth, `Content-Length: 0`). It force-expires a grant so you can test three conditions immediately:
access-token expiry, enrollee revocation, and data-access-grant expiry. This is the only time
simulation facility on the CMS surface — see `sandbox/cms-sandbox.yml`.
