# Google Business Lead Research

A production-oriented Python automation pipeline for discovering, researching, validating, and exporting structured business leads from public Google Business / Google Maps results.

The project combines browser automation, deterministic data processing, evidence provenance, semantic retrieval, controlled AI reasoning, validation guardrails, and structured exports.

It is designed as a portfolio demonstration of practical **Python automation, web research, RAG, AI-assisted decision support, QA engineering, Docker, and Azure deployment**.

---

## Project Overview

Google Business Lead Research converts a business category and target location into structured lead records.

Example:

```text
Keyword: plumber
Location: Antipolo City, Rizal, Philippines


The core workflow:
1. Plans the search request.
2. Discovers matching businesses.
3. Extracts available public business information.
4. Normalizes names, addresses, phone numbers, and website domains.
5. Deduplicates candidate records.
6. Researches available website information.
7. Preserves evidence and source provenance.
8. Generates embeddings for evidence.
9. Stores evidence in a ChromaDB vector store.
10. Retrieves semantically relevant evidence.
11. Supports controlled AI tool routing.
12. Produces evidence-grounded AI assessments.
13. Applies deterministic validation and QA guardrails.
14. Exports structured results.
15. Supports standalone Docker execution and Azure Container Apps Job deployment.
The architecture deliberately separates AI reasoning from deterministic business rules. AI-generated assessments are not treated as automatically trustworthy; evidence references, confidence thresholds, deterministic checks, and REVIEW fallbacks are used to keep decisions auditable.


Architecture

Search Request
     |
     v
Query Planning
     |
     v
Google Business Discovery
     |
     v
Raw Business Candidates
     |
     v
Normalization + Deduplication
     |
     v
Website Research
     |
     v
Evidence Documents
     |
     v
Evidence Chunks + Provenance
     |
     v
Ollama Embeddings
     |
     v
ChromaDB Vector Store
     |
     v
Semantic Evidence Retrieval
     |
     +--------------------------+
     |                          |
     v                          v
Controlled AI Routing     Deterministic Lead QA
     |
     v
Evidence-Grounded AI Assessment
     |
     v
Deterministic Assessment Validation
     |
     v
SUPPORTED / REVIEW
     |
     v
Structured Export
CSV / XLSX / JSON


Core Capabilities
- Google Business / Google Maps discovery
- Playwright browser automation
- Location-based business research
- Configurable result limits
- Business-name normalization
- Address normalization
- Phone normalization
- E.164 phone representation when available
- Website and domain normalization
- Lead deduplication
- Website research
- Evidence-document modeling
- Evidence chunking with provenance
- Local embedding generation with Ollama
- ChromaDB vector storage
- Semantic evidence retrieval
- RAG retrieval foundation
- Controlled AI tool routing
- Evidence-grounded AI assessment
- Confidence-based deterministic validation
- Human-review fallback
- Lead quality scoring
- CSV export
- XLSX export
- JSON export
- Docker containerization
- Azure Container Apps Job deployment


Evidence and RAG Architecture
The project preserves source information rather than passing untraceable text directly to an AI model.
Research content is represented as evidence documents and evidence chunks with metadata such as:
- Business name
- Source URL
- Final URL
- Source type
- Chunk index
- Page title
Evidence is embedded locally using:
nomic-embed-text
The embeddings are stored in ChromaDB and can be queried through semantic retrieval.
This provides the retrieval foundation for evidence-grounded AI assessment.


Controlled AI Reasoning
The project uses local Ollama models for AI-assisted workflow components.
Verified local models used by the project include:

qwen2.5
nomic-embed-text

qwen2.5 is used for controlled reasoning and structured JSON responses.
The controlled agent restricts routing decisions to an approved tool allowlist:

plan_search
retrieve_evidence
assess_lead
run_workflow

A model-generated tool name outside the allowlist is not automatically executed and is instead routed to REVIEW.
This implementation demonstrates controlled AI tool routing. It should not be interpreted as unrestricted autonomous agent execution or native LangChain function calling.


Evidence-Grounded Assessment
AI assessments operate on supplied retrieved evidence rather than unrestricted model knowledge.

An assessment contains:

answer
confidence
evidence_ids
source_urls
status


The assessment layer verifies that referenced evidence IDs exist in the supplied retrieval context and derives source URLs from evidence metadata.
When evidence is unavailable or an assessment cannot be sufficiently supported, the workflow can return:

REVIEW

rather than fabricating certainty.


Deterministic Validation and Guardrails
Critical acceptance rules are implemented in deterministic Python rather than delegated entirely to an LLM.
For supported AI assessments, the validator requires:
- Valid assessment status
- Confidence between 0 and 1
- Minimum supported confidence of 0.85
- Evidence IDs
- Source URLs
An assessment that does not satisfy the support requirements is downgraded to REVIEW.
This architecture follows a hybrid approach:

AI reasoning
+
retrieved evidence
+
deterministic validation
+
human review when needed


Lead Quality Assessment
Business leads also pass through deterministic lead-quality scoring.
The scoring system evaluates available fields such as:
- Business name
- Phone
- Address
- Website
- Source URL
- Search location
Lead records can be classified into quality states such as:

approved
review
rejected

This QA layer is separate from the evidence-grounded AI assessment validator.


Structured Lead Output
The primary lead export contract contains 21 fields:

| Field | Description |
|---|---|
| `business_name` | Normalized business name |
| `category` | Business category |
| `industry` | Business industry classification |
| `phone` | Normalized business phone |
| `phone_e164` | E.164 phone representation when available |
| `email` | Business email when available |
| `address` | Normalized full business address |
| `address_line1` | Primary street address |
| `city` | Business city |
| `state` | Business state or region |
| `postal_code` | Postal or ZIP code |
| `country_code` | Country code |
| `website` | Business website URL |
| `website_domain` | Normalized website domain |
| `source` | Discovery source |
| `source_url` | Source URL associated with the lead |
| `search_keyword` | Search keyword |
| `search_location` | Search location |
| `qa_status` | Lead QA status |
| `qa_score` | Lead QA score |
| `qa_flags` | QA flags |

Some values may be empty when the public source does not provide the information.


AI Assessment Export
Evidence-grounded assessments use a separate five-field export contract:

answer
confidence
evidence_ids
source_urls
status

Assessment results can be exported to:

JSON
CSV
XLSX

JSON preserves evidence IDs and source URLs as arrays.
CSV and XLSX serialize multi-value evidence fields using a pipe (|) delimiter.


Local CLI Usage
The standalone application entry point is:

python -m src.main \
  --keyword "plumber" \
  --location "Antipolo City, Rizal, Philippines" \
  --max-results 3 \
  --output-dir output \
  --output-name antipolo_plumbers

On Windows Command Prompt, the same command can be entered on one line:

python -m src.main --keyword "plumber" --location "Antipolo City, Rizal, Philippines" --max-results 3 --output-dir output --output-name antipolo_plumbers

The workflow produces:

antipolo_plumbers.csv
antipolo_plumbers.xlsx
antipolo_plumbers.json


Docker
The project includes a standalone Docker deployment using the Microsoft Playwright Python image.
The container starts the application with:

python -m src.main

Build:

docker build -t google-business-lead-research:step15 .

CLI verification:

docker run --rm google-business-lead-research:step15 --help

A real end-to-end Docker smoke test was completed with:

keyword: plumber
location: Antipolo City, Rizal, Philippines
max_results: 1

The container completed the lead-research workflow and produced one lead with CSV, XLSX, and JSON exports persisted to the host through a mounted output directory.


Azure Deployment
The standalone Docker image was also deployed and validated using:
- Azure Container Registry
- Azure Container Apps Jobs
- Consumption workload profile
- Managed identity for registry authentication
- AcrPull role-based access
Registry authentication uses the Container Apps environment's system-assigned managed identity rather than enabling ACR administrator credentials.
A real Azure Container Apps Job execution completed successfully using:

keyword: plumber
location: Antipolo City, Rizal, Philippines
max_results: 1

Verified application output:

Lead research completed: 1 leads
CSV: output/azure_smoke_test.csv
XLSX: output/azure_smoke_test.xlsx
JSON: output/azure_smoke_test.json

The successful Azure execution completed in approximately 44 seconds.
Azure storage note
The current Azure smoke test writes exports to the job container's local filesystem.
Azure Container Apps Job container storage is ephemeral unless persistent storage or an external storage service is configured. Therefore, the Azure validation demonstrates successful cloud execution of the workflow, but does not claim persistent Azure storage of the generated export files.
The local Docker validation separately verifies persisted host-mounted CSV, XLSX, and JSON output.


Deployment Scope
The current portfolio deployment target is:

GitHub
   |
   v
Docker
   |
   v
Azure Container Registry
   |
   v
Azure Container Apps Job

The repository retains earlier Apify-compatible runtime components, but this project is not being published as an additional Apify Store Actor.
The standalone deployment path uses:

python -m src.main

rather than the legacy Apify Actor entry point.


AI / Cloud Scope
The RAG, embedding, retrieval, controlled-agent, and evidence-assessment components have been validated locally with the project's Ollama-based architecture.
The Docker and Azure deployment smoke tests validate the standalone business lead research and structured export workflow.
The current Azure deployment does not claim to host an Ollama service or execute the complete local Ollama-based AI/RAG stack in Azure.
This distinction is intentional so that local AI validation and cloud workflow validation remain accurately documented.


Testing
The project includes regression tests covering the major workflow components, including:
- Discovery and extraction behavior
- Normalization
- Deduplication
- Lead QA
- Export contracts
- Evidence models
- Embedding integration
- ChromaDB vector storage
- Semantic retrieval
- Controlled AI routing
- Evidence-grounded assessment
- Deterministic assessment validation
- Structured assessment exports
Final regression result after Docker and Azure deployment validation:

119 passed in 47.39s


Technology Stack
Core
- Python 3.11
- Playwright
- OpenPyXL
- ChromaDB
AI / RAG
- Ollama
- qwen2.5
- nomic-embed-text
- LangChain Ollama integration
Deployment
- Docker
- Microsoft Playwright Python container image
- Azure Container Registry
- Azure Container Apps Jobs
- Azure managed identity / RBAC
Legacy Compatibility
- Apify Python SDK
- Existing Apify runtime modules


Engineering Principles
The project follows several practical reliability principles:
Evidence first
AI conclusions should be traceable to supplied evidence whenever evidence-based assessment is required.
Deterministic critical rules
Important acceptance conditions are implemented in Python rather than delegated entirely to an LLM.
Structured outputs
AI and automation components use defined schemas and predictable export contracts.
Provenance
Source URLs and evidence identifiers are retained so results can be traced back to supporting material.
Review instead of fabrication
Ambiguous or insufficiently supported results are routed to REVIEW rather than being presented as certain.
Regression protection
New capabilities are validated against the existing test suite before being accepted into the project.


Responsible Use
This project is intended for legitimate business research and automation using publicly accessible information.
Data availability and completeness depend on the source pages. A business listing may not contain every expected field.
Users are responsible for complying with applicable laws, privacy requirements, contractual obligations, and source-site terms when collecting or using business information.
The system does not guarantee that every discovered business is a qualified sales prospect. QA and AI-assessment mechanisms are decision-support tools and should be reviewed appropriately.


Project Status
Portfolio-ready engineering implementation completed.
Verified checkpoints include:

Core lead research pipeline          PASSED
Normalization and deduplication      PASSED
Evidence/provenance architecture     PASSED
Embeddings and vector storage        PASSED
Semantic retrieval                   PASSED
Controlled AI routing                PASSED
Evidence-grounded assessment         PASSED
Deterministic validation             PASSED
Structured exports                   PASSED
Docker build and execution           PASSED
Azure cloud execution                PASSED
Final regression suite               119/119 PASSED

The remaining release activity is repository documentation and GitHub publication.

