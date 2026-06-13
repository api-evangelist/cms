# Centers for Medicare and Medicaid Services (CMS) APIs

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
