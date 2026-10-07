# Real-World Validation Case Study

## Google Business Lead Research — Antipolo City, Philippines

### Objective

Validate the production-oriented lead research pipeline against live public business listings while preserving source provenance, applying deterministic quality checks, and avoiding unsupported data when extracted information is incomplete.

### Validation Environment

The workflow was executed using the standalone Docker deployment.

Test configuration:

- Search keyword: `dentist`
- Search location: `Antipolo City, Rizal, Philippines`
- Country code: `PH`
- Maximum results: `5`
- Export formats: CSV, XLSX, JSON

The container completed successfully and generated all three structured export formats.

### Verified Results

The live validation produced:

| Result | Verified value |
|---|---:|
| Business records discovered | 5 |
| Country normalization | PH |
| REVIEW records | 4 |
| REJECTED records | 1 |
| CSV export | Generated |
| XLSX export | Generated |
| JSON export | Generated |
| Missing extracted fields | Preserved as missing values |

The five records received deterministic QA scores ranging from `35` to `60`.

Two records scored `60` and were routed to `REVIEW`. Two records scored `50` and were also routed to `REVIEW`. One record scored `35` and was routed to `REJECTED`.

### Evidence of Conservative Data Handling

The extracted records did not contain complete contact information for every discovered business.

In the inspected output, unavailable phone, address, website, and related fields were preserved as missing values rather than populated with unsupported values.

Examples observed during validation included:

- A record with an extracted website but no extracted phone or address.
- A record with an extracted street address but no extracted phone or website.
- A record without an extracted phone, address, or website.

The deterministic QA layer identified these missing fields and produced explicit flags such as:

- `missing_phone`
- `missing_address`
- `missing_website`

This behavior is intentional. Incomplete extracted data is treated as a quality condition rather than silently converted into apparently complete lead data.

### Provenance

Each discovered record retained:

- Discovery source
- Source URL
- Search keyword
- Search location
- Country code
- QA status
- QA score
- QA flags

Raw source URLs and business identities are intentionally omitted from this public case study. The underlying validation artifacts remain in the private development environment.

### QA Outcome

The validation demonstrates that the pipeline does not treat every discovered business as an automatically qualified lead.

Instead, deterministic rules classify records according to the information actually present in the extracted data.

For this validation run:

- `4/5` records required review.
- `1/5` record was rejected.
- `0/5` records were classified as APPROVED.

No record was promoted to `APPROVED` without satisfying the required quality conditions.

This is expected behavior for an evidence-first workflow operating on incomplete real-world extracted data.

### Deployment Evidence

The standalone application architecture has been validated through:

- Local Python execution
- Docker container execution
- Azure Container Apps Job execution

For this case study, the live business validation was executed through Docker.

The Docker validation used a host-mounted output directory, allowing the generated CSV, XLSX, and JSON files to persist outside the container.

Azure validation was performed separately and demonstrated successful cloud execution. Azure job-local export storage was ephemeral during that smoke test, so persistent Azure storage is not claimed.

### Engineering Takeaway

This validation demonstrates more than successful business discovery.

The verified run shows that the system can:

- Discover real public business records.
- Normalize location context using an explicit country code.
- Preserve source provenance.
- Export structured results in multiple formats.
- Detect incomplete extracted records.
- Apply deterministic QA scoring.
- Route uncertain records to `REVIEW`.
- Reject records that do not meet minimum completeness expectations.
- Preserve missing fields rather than populate them with unsupported values.

The result is an auditable lead-research workflow designed around evidence, deterministic validation, and transparent handling of incomplete real-world data.

## Public Portfolio Scope

This case study presents verified system behavior and validation results without publishing the reusable production implementation.

The complete source code, extraction logic, tests, internal workflow implementation, and raw validation artifacts remain private.