# Contract Intelligence Assistant — Product Requirement Prompt (PRP)

## Vision
Deliver a contract intelligence web application that ingests production contracts from SharePoint, classifies them (MSA, SOW, software agreements, etc.), extracts rich metadata, and exposes the information through dashboards and an AI-assisted chat experience. The tool must reduce manual review time, surface renewal risk, and answer natural-language questions using structured data maintained in the core platform.

## Primary Users & Goals
- **Legal & Contract Operations:** Monitor contract pipeline, confirm key terms, and triage renewal actions without opening each file.
- **Account / Delivery Managers:** Track customer-specific obligations, renewal dates, payment terms, and milestones.
- **Executives / Finance:** Roll up metrics (contract value, renewals due, compliance coverage) for decision support.
- **Data Ops / Admins:** Configure connectors, extraction schemas, quality rules, and oversee retraining.

## Problem Statements
- Contracts are stored in SharePoint with inconsistent metadata, requiring manual download and review.
- Existing tooling focuses on structured data; unstructured contract documents remain a blind spot.
- Users need a guided assistant that can answer questions (“Which contracts renew in 90 days?”) and justify answers with links back to the source document.
- Contract types have distinct attributes; schema updates must be easy without schema migrations per change.

## Core Solution Pillars
1. **Automated Ingestion & Refresh** from SharePoint (and future repositories) with delta detection.
2. **Contract Classification & Metadata Extraction** tailored to contract categories with configurable fields.
3. **Contract Intelligence Workspace** delivering dashboards, filtering, and exports without exposing raw documents.
4. **AI Copilot Chat** capable of answering metric queries via SQL over structured data and optionally surfacing semantic insights from embeddings.
5. **Governance & Quality Framework** leveraging rule outcomes, audit logging, and soft deletes.

## Functional Requirements
### 1. Connectors & Ingestion
- Admins provide SharePoint connection details (site, library, credentials/secrets) via secure settings UI.
- Support scheduled polling plus manual “ingest now” trigger.
- Detect added/updated/deleted files since last run using SharePoint metadata; download new/changed files to managed storage.
- Provide interim manual file upload UI (single/multi file) that creates a batch and mimics SharePoint ingestion flow for testing before the connector is live.
- Persist ingestion context in `Batch` table; include external identifier, directory, timestamps.
- Maintain original SharePoint URL for click-through; do not serve raw document content from the app.
- Extendable connector interface (future: S3, Google Drive).

### 2. Classification & Extraction
- Identify contract category automatically (MSA, SOW, software license). Allow manual override in UI.
- Map detected category to `Category` table; link documents via `BatchDocument`.
- Configure extraction fields per category via `ExtractField` + `CategoryExtractField`; support types from `ExtractFieldType` (string, integer, date, boolean, JSON).
- Extraction pipeline outputs values into `BatchExtractField` with confidence and provenance.
- Capture customer/party as `Entity`; maintain many-to-many `EntityCategory` for category-specific context.
- Store high-value clauses or findings as `BatchDocumentFact` (subject, predicate, object) for semantic insights.
- Allow enrichment/override via UI edits; track updates and who made them.

### 3. Data Quality & Rules
- Enable configuration of rule expressions per category in `Rule` table; categorize by `RuleType` (field validation, semantic check, anomaly detection).
- Record outcomes per batch in `BatchRuleOutcome` along with notes.
- Support existing unstructured data quality rule definitions; admins choose which rules execute per ingestion.
- Provide summary of pass/fail counts and drill-down per contract.
- Apply `DeletedOn` timestamps instead of hard deletes for auditability.

### 4. Dashboard & Insights
- Landing dashboard summarizing:
  - Contracts ingested, categorized counts, recent activity.
  - Renewals due (30/60/90 days), active obligations, data quality status.
  - Filter by customer, contract type, rule status, payment terms.
- Provide visualizations (bar/line charts for renewal pipeline, pie for contract types, KPI tiles).
- Present contract list view with key metadata columns and quick filters.
- Contract detail page displays extracted fields grouped logically (Party Info, Terms, Financials, Obligations), rule outcomes, and link to original SharePoint file.
- Export filtered results to CSV/Excel and share saved views per user.

### 5. Chat Assistant
- Embed chat panel (persistent drawer or side panel) accessible from dashboard and contract detail.
- Support user prompts:
  - **Structured queries:** translate to SQL against core tables to answer questions like “How many MSAs expire this quarter?”
  - **Document-specific queries:** allow selection of contract to focus context.
  - **Semantic search:** optional retrieval from embeddings/vector index for clause discovery (“Show contracts mentioning limited liability”).
- Responses must reference supporting metadata (contract name, field, SharePoint link) and confidence.
- Maintain chat history in `Chat`/`Message` tables tied to the requesting user.
- Provide guarded execution: review generated SQL before execution; enforce query templates/allow-list.

### 6. User & Access Model
- Reuse existing authentication (FastAPI + JWT + React auth flows).
- Roles:
  - **Admin:** manage connectors, fields, rules, users.
  - **Analyst:** view dashboards, run chat, edit metadata.
  - **Viewer:** read-only access.
- Audit key actions (metadata overrides, rule configuration changes, chat query logs).

### 7. Notifications & Workflow
- Optional email/slack notifications for upcoming renewals or failed ingestions.
- Provide tasks/to-do list for exceptions (e.g., manual review required) on dashboard.

## Data Model Alignment (per ERD)
- `ExtractFieldType`, `ExtractField`, `CategoryExtractField` define schema for metadata fields per contract type.
- `Category` enumerates contract types (MSA, SOW, Software) and can expand.
- `Entity` stores organizations (customers); `EntityCategory` links customers to contract categories.
- `Batch` represents ingestion runs; `BatchDocument` captures each file with location, category, timestamps.
- `BatchDocumentTag` stores dynamic key/value tags from ingestion pipeline.
- `BatchExtractField` stores extracted field values; `BatchDocumentFact` holds semantic triples or clause summaries.
- `BatchRuleOutcome` records rule validation results.
- `Chat` and `Message` persist user/assistant conversations.
- Enforce soft delete via nullable `DeletedOn` column on mutable tables to preserve referential integrity.

## Processing Pipeline (High Level)
1. **Batch Initialization:** Scheduler/trigger creates `Batch` row and pulls SharePoint delta list.
2. **Document Acquisition:** Download updated contracts, log as `BatchDocument` with raw text path.
3. **Classification:** Run model/heuristics to assign `Category`; fallback to manual queue on low confidence.
4. **Extraction:** Execute LLM/regex/workflow per `CategoryExtractField`; store values with provenance and confidence.
5. **Rule Evaluation:** Evaluate configured `Rule`s; populate `BatchRuleOutcome` and attach alerts.
6. **Vectorization (optional for MVP):** Generate embeddings for clause-level search; index in vector store linked via `BatchDocument`.
7. **Publish:** Update dashboards, refresh chat retrieval context, notify stakeholders.

## Frontend Experience (React / Chakra UI)
- **Authentication:** Keep existing login/register flows.
- **Main layout:** Left nav (Dashboard, Contracts, Rules, Connectors, Chats, Settings).
- **Dashboard page:** KPIs, charts, widgets, alerts.
- **Contracts page:** table with filter panel, bulk export, quick actions.
- **Contract detail drawer/page:** structured sections, rule status, chat pivot to “ask about this contract”.
- **Rules & Fields admin:** forms to manage extraction fields, rule scripts, toggles.
- **Connectors page:** manage SharePoint credentials, sync schedule, last run logs, and provide manual upload workflow for ad-hoc testing.
- **Chat workspace:** full-screen or drawer UI showing history, response citations, follow-up suggestions.

## AI & Data Services
- Use existing OpenAI key (per transcript) for LLM calls; wrap in service with usage tracking to avoid runaway costs.
- Support parameterization per environment (dev/test/prod) and caching of extraction results.
- Ensure models can run in batch mode; design to plug in alternative providers.
- Provide guardrails for prompt injection, rate limits, and logging of prompts/responses.

## Integration & External Systems
- **SharePoint:** Primary ingestion source; store credentials securely (e.g., Azure AD app + OAuth client secret or delegated tokens).
- **Customer master (Salesforce or import):** Provide API/import to sync customer list for `Entity` table if not derived from contracts.
- **Notification channels:** Email via existing SMTP service; Slack/Teams webhooks (optional backlog).

## Non-Functional Requirements
- **Performance:** Support 10k contracts with sub-second dashboard filtering and <5s chat responses (excluding LLM latency).
- **Scalability:** Batch ingestion runs should be resumable and parallelizable (per directory/bucket).
- **Reliability:** Retry failed document ingestions; track status per batch.
- **Security:** Enforce least privilege, encrypt stored files/credentials, rely on SSO (future) aligned with Sense SSO patterns.
- **Compliance:** Log access to contract metadata; support export of audit logs.

## Implementation Roadmap (Suggested)
1. **Foundations (Sprint 0-1):** Extend database models per ERD, migrate schema, scaffold admin UI for categories/entities/fields, implement manual upload-based ingestion path, and stub SharePoint connector.
2. **Ingestion & Extraction MVP (Sprint 2-3):** Implement batch ingestion, automatic classification, baseline extraction for key fields (parties, dates, renewal terms), UI for contract list & detail, manual override flows.
3. **Dashboard & Metrics (Sprint 4):** Build dashboard visualizations, filters, exports, notification hooks.
4. **Chat Assistant v1 (Sprint 5):** Structured SQL query generation, chat UI, citation linking.
5. **Quality & Semantic Enhancements (Sprint 6+):** Rule configuration UI, BatchRuleOutcome surfacing, optional vector search, advanced clause metadata.

## Success Metrics
- Time to answer contract questions reduced by ≥50%.
- ≥90% automatic classification accuracy for supported contract types.
- Extraction accuracy (critical fields) ≥85% with manual correction workflow.
- Renewal alerts delivered at least 30 days before due date.
- Chat answer satisfaction ≥4/5 in user feedback.

## Open Questions & Assumptions
- Confirm authentication/SSO plan (reuse Sense SSO vs. existing template auth). Let's use template auth for now and will integrate with Azure later.
- Determine source of authoritative customer list (Salesforce integration vs. manual upload). This can be manual upload.
- Decide on initial set of extraction fields per contract type (legal to provide prioritized list, e.g., 25+ fields). You can generate these for now.
- Clarify approach for storing raw contract text (filesystem vs. blob storage) and retention policy. For now let's store files locally.
- Confirm vector database selection (e.g., PostgreSQL pgvector vs. external service) and MVP timeline. I want to use postresQL for all of this.
- Need final decision on notification channels and escalation workflow for failed ingestions. Not needed at the moment.

## Appendices
- **Design Considerations:** Hide contract files from the app; surface sharepoint links only. Support manual reprocessing per document. Provide sandbox mode for model testing without committing results.
- **Risk Mitigations:** Implement cost monitoring for LLM usage, guard against long-running extraction loops, ensure connector handles permission errors gracefully.
- **Testing Strategy:** Unit tests for ingestion parsers, contract classification evaluation harness, end-to-end Playwright flow for dashboard and chat, synthetic contract set for regression.
