# Centers for Medicare and Medicaid Services (CMS) APIs

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

The Centers for Medicare and Medicaid Services (CMS) provides a suite of public REST APIs enabling developers to access Medicare provider data, quality measures, drug spending, health plan finder, beneficiary claims, and public health insurance datasets. CMS APIs support interoperability standards including HL7 FHIR and OAuth 2.0 to power healthcare applications across the US health system.

## Developer Portal

https://developer.cms.gov/

## APIs

### Marketplace API
Delivers data that helps users find and evaluate health care insurance plans, providers, and coverage information on the ACA Health Insurance Marketplace. Powers Healthcare.gov and third-party services.

- Documentation: https://developer.cms.gov/marketplace-api/
- Base URL: https://marketplace.api.healthcare.gov/api/v1
- Auth: API key (query parameter)

### Blue Button 2.0 API
A standards-based FHIR API delivering Medicare Part A, B, and D claims data for over 60 million Medicare beneficiaries via OAuth 2.0.

- Documentation: https://bluebutton.cms.gov/api-documentation
- Sandbox: https://sandbox.bluebutton.cms.gov/
- Auth: OAuth 2.0

### Beneficiary Claims Data API (BCDA)
Enables Medicare ACOs and alternative payment model participants to retrieve bulk Medicare claims data via FHIR R4.

- Documentation: https://bcda.cms.gov/api-documentation.html
- Auth: OAuth 2.0 bearer token

### AB2D API (Claims Data to Part D Sponsors)
Enables Medicare Part D plan sponsors to retrieve bulk Part A and B claims data via FHIR.

- Documentation: https://ab2d.cms.gov/api-documentation
- Auth: OAuth 2.0 (Okta)

### Data at the Point of Care (DPC) API
Provides Medicare claims data to fee-for-service providers at the point of care via FHIR.

- Documentation: https://dpc.cms.gov/docsV1
- Auth: OAuth 2.0

### Finder API
Powers Finder.Healthcare.gov to help users find private health plans outside the Marketplace.

- Documentation: https://finder.healthcare.gov/#services
- Auth: API key
- Rate Limit: 1000 requests/minute

### Procedure Price Lookup (PPL) API
Provides bulk CPT/HCPCS procedure cost data for hospital outpatient settings and ASCs.

- Documentation: https://developer.cms.gov/ppl-api/
- Auth: API key + AMA CPT license

### Quality Payment Program (QPP) Submissions API
Enables QPP participants to submit quality performance data and receive real-time scoring.

- Documentation: https://cmsgov.github.io/qpp-submissions-docs/
- Auth: OAuth 2.0

## Data Portal

https://data.cms.gov/ — Free public datasets with no API key required, covering Medicare provider data, quality measures, drug spending, and more.

## GitHub

https://github.com/cmsgov

## APIs.json

https://raw.githubusercontent.com/api-evangelist/cms/refs/heads/main/apis.yml
