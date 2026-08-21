# Purchase Price Check — Proposed technical foundation

**Status:** Conceptual architecture for production discovery  
**Evidence boundary:** Stampli production repositories, services, schemas, and APIs were not available. Names below are domain concepts, not prescribed services, endpoint paths, or database tables.  
**Updated:** 2026-08-21

## 1. Architecture goals

The implementation must support a durable purchase-bidding lifecycle, isolate external marketplace failures from the core workflow, preserve immutable source evidence, and connect the effective award to downstream PO and invoice outcomes.

The design should optimize for:

- reuse of Stampli's existing purchase, policy, vendor, authorization, document, messaging, PO, invoice, and audit capabilities where verified;
- authoritative server-side state and permission enforcement;
- provider-neutral benchmark integration;
- versioned data lineage from source fact through derived value and decision;
- asynchronous work for vendor communication, document extraction, benchmark retrieval, and downstream handoff; and
- safe tenant- and category-level rollout.

## 2. Conceptual system boundaries

| Capability boundary | Responsibility | Reuse question to resolve |
|---|---|---|
| Purchase request | Own the business need, entity, requester, lines, quantity, destination, required date, estimate, and links | Which existing object is authoritative? |
| Policy evaluation | Determine bidding, benchmark, exception, approval, and segregation rules | Can existing workflow policy express both bid and benchmark outcomes? |
| Bid orchestration | Own event lifecycle, current specification version, participants, deadlines, response channels, readiness, improvement requests, and effective award link | New domain or extension of an existing purchase object? |
| Vendor access and messaging | Deliver scoped invitations, authenticate/verify respondents, save drafts, and send amendments/reminders | Can the existing vendor portal support quote-only identities? |
| Document and AI processing | Extract candidate fields, confidence, source locations, normalized values, and explanations | Which Billy capabilities support human confirmation and immutable lineage? |
| Benchmark orchestration | Evaluate provider coverage, create attempts, call adapters, validate results, deduplicate, normalize, and manage freshness | New provider-agnostic capability; no production baseline available |
| Decision and approval | Enforce award authority, rationale, exception approval, and stale-context checks | Can existing approval infrastructure provide idempotent, version-aware decisions? |
| PO and invoice linkage | Create PO input from award and calculate downstream variance | Which existing mapping and matching services can be reused? |
| Audit and analytics | Preserve material state changes and emit operational/outcome events | What retention and immutability guarantees already exist? |

## 3. Proposed domain model

### 3.1 Core entities

| Entity | Purpose | Critical fields or relationships |
|---|---|---|
| PurchaseRequest | Existing or conceptual parent business need | Tenant/entity, requester, lines, category, estimate, currency, destination, required date, contract status |
| PolicyDecision | Versioned control result | Input version, evaluated rules, winning rule, bidding outcome, benchmark outcome, exception requirements, timestamp |
| BidEvent | One private competitive check | Parent request, owner, lifecycle state, deadline, event currency, current specification version, effective award |
| SpecificationVersion | Common requirement sent to participants | Version, lines, identifiers, attributes, substitutions, quantities, destination, dates, response fields, amendment reason |
| EventParticipant | One invited vendor/contact | Vendor or quote-only identity, invitation status, access state, response status, delivery metadata |
| VendorResponseVersion | Vendor's submitted or buyer-entered source record | Participant, version, response channel, entering/confirming actor, source attachments/evidence, structured source fields, confirmations, submission state, submitted timestamp |
| SourceFact | Immutable vendor- or provider-originated value | Owner source, field semantics, original value/unit/currency, source location, capture time, confirming actor |
| DerivedValue | A calculation or AI transformation | Inputs, formula/model version, output, confidence, assumptions, creator, timestamp |
| BenchmarkAttempt | One provider search for one specification version/line | Applicability basis, provider, query inputs, state, started/completed time, failure category, retry count |
| BenchmarkCandidate | One marketplace offer or reference | Provider/seller IDs, product match, condition, quantity/MOQ, price components, delivery, availability, URL/reference, capture time |
| BenchmarkAssessment | Comparability decision for a candidate or result set | Match factors, exclusions, confidence band, landed-cost status, freshness, rule/model version |
| ImprovementRequest | One controlled “Ask to improve” request for a revised offer | Participant, recipient response version, specification version, final reviewed message, disclosure policy/result, public-evidence version and claim type when used, sender, deadline, send state, resulting response version |
| ExceptionRequest | Controlled waiver | Requirement waived, reason code, explanation, evidence, policy, requested/approved actors, state |
| AwardVersion | Human commercial decision | Selected response version, rationale, evidence versions, benchmark context, approvals, effective flag, timestamp |
| HandoffAttempt | Delivery of award to downstream PO preparation | Award version, state, target reference, idempotency key, failures/retries |
| VarianceAssessment | Award-versus-PO/invoice result | Basis, compared object versions, field deltas, materiality rules, state, review outcome |
| AuditEvent | Durable record of a material action | Tenant, actor/service, correlation, object/version, action, timestamp, before/after or immutable payload reference |

### 3.2 Relationships

```text
PurchaseRequest
  ├─ PolicyDecision (versioned)
  └─ BidEvent
       ├─ SpecificationVersion (versioned)
       ├─ EventParticipant ─ VendorResponseVersion ─ SourceFact
       │                    └─ ImprovementRequest ─ revised VendorResponseVersion
       ├─ BenchmarkAttempt ─ BenchmarkCandidate ─ BenchmarkAssessment
       ├─ ExceptionRequest
       ├─ AwardVersion ─ HandoffAttempt ─ PO
       ├─ VarianceAssessment ─ PO / Invoice
       └─ AuditEvent

SourceFact ─ DerivedValue ─ Comparison / AwardVersion
```

The production design may embed or rename these concepts, but it must preserve equivalent versioning, relationships, lineage, and permissions.

## 4. Lifecycle models

### 4.1 Bid event

```text
Draft
  → Need approved
  → Ready to publish
  → Collecting responses
  → Closed / Ready to compare
  → Improvement pending (optional, per vendor)
  → Award pending approval
  → Awarded
  → PO handoff pending
  → Completed
```

Side states:

- Cancelled
- Exception approved
- Incomplete — insufficient valid responses
- Reopened — only through an authorized, audited transition
- Handoff failed — award remains effective while delivery retries

Every transition must define permitted source states, required role, validations, side effects, idempotency behavior, and audit event.

### 4.2 Vendor response

```text
Not invited → Invitation pending → Delivered → Opened → Draft → Submitted
                                                         ↘ Declined
Submitted → Ask to improve drafted → Improvement requested → Revised response submitted / Request expired
Submitted → Vendor revision before deadline → Submitted new version
Any active state → Access revoked / Event cancelled / Expired
```

### 4.3 Benchmark attempt

```text
Not evaluated
  → Not applicable
  → Eligible → Queued → Retrieving → Validating → Normalizing
  → Comparable / Indicative / No reliable benchmark
  → Retryable failure → Queued
  → Unavailable
```

The event workflow must not depend on a synchronous provider response. Customer policy decides whether an unresolved required benchmark blocks award, accepts fallback evidence, or requires exception approval.

### 4.4 Readiness model

The implementation must keep these states separate:

- **Policy minimum satisfied:** the configured count of valid vendor responses is present.
- **Collection complete:** every invited vendor has responded or declined, the deadline has passed, or an authorized early-close rule has committed.
- **Ready to compare:** policy minimum and collection-complete rules pass, with no blocking response or specification conflicts.
- **Ready to award:** comparison is current, required benchmark/exception rules pass, and no blocking revised response is pending.

Adding a vendor after publication must version participant membership and apply a customer-defined deadline fairness rule.

## 5. Benchmark provider abstraction

### 5.1 Required adapter capabilities

The internal provider contract should support capabilities rather than assume one universal marketplace API:

- provider metadata, geographic/category coverage, terms version, and health;
- search by stable identifier, structured attributes, or permitted query;
- retrieve offer/listing/RFQ evidence available under the integration;
- normalize provider-specific pagination, rate-limit, error, and freshness metadata;
- return raw permitted evidence and a provider reference without applying business conclusions;
- declare whether price, shipping, tax, duty, fees, MOQ, condition, availability, delivery, seller, and validity are available; and
- expose storage/display restrictions required by provider terms.

### 5.2 Adapter output principles

- Provider adapters translate transport and provider semantics; they do not decide comparability or award relevance.
- Raw permitted evidence is immutable after capture.
- Provider secrets and raw error bodies never reach end users or analytics.
- Every attempt has an idempotent correlation key derived from tenant, provider, specification version, item, destination basis, and configured freshness window.
- Timeouts, bounded retries, rate limiting, circuit breaking, and a provider kill switch are mandatory.
- A provider failure must not cascade into purchase request or vendor response availability.

### 5.3 Launch-source selection criteria

Select the first provider by a scored feasibility review:

| Criterion | Required evidence |
|---|---|
| Pilot-category coverage | Recall and result availability on representative customer items |
| Exact-match support | Stable identifiers and variant/condition data |
| Commercial basis | Unit, quantity, MOQ, availability, delivery, and cost components |
| Access rights | Approved API/feed/partner/customer-authorized method; no scraping assumption |
| Display and retention rights | Permission to show source data and preserve decision-time evidence |
| Geographic fit | Destination, currency, tax/duty, and seller coverage for pilot customers |
| Operational fit | Rate limits, latency, uptime, support, versioning, and cost |
| Security/privacy | Credential model, subprocessors, PII/commercial-data handling, and deletion |

Amazon Business publicly documents an Integrated Quoting relationship with Fairmarkit, demonstrating that partner integration is possible for eligible high-volume requests. Alibaba.com publicly documents its RFQ marketplace, but this assessment did not verify a general product-price API suitable for this use. Neither brand should be committed as the launch connector before feasibility review.

## 6. Match and normalization pipeline

### 6.1 Proposed stages

1. **Eligibility:** confirm category, geography, provider, and legal access.
2. **Query construction:** use exact identifiers first; then structured attributes only when allowed.
3. **Candidate validation:** reject malformed, unavailable, prohibited, or clearly mismatched results.
4. **Identity match:** compare manufacturer, part/model, variant, condition, pack/unit, and substitutions.
5. **Commercial normalization:** map quantity, MOQ, unit basis, currency, shipping, tax, duty, fees, delivery, and validity.
6. **Confidence assessment:** produce versioned factors and a comparable/indicative/reject classification.
7. **Result-set calculation:** calculate a range only from qualifying offers; identify inclusions and exclusions.
8. **Evidence publication:** expose source, timestamp, assumptions, unknowns, and confidence.
9. **Freshness management:** refresh or flag stale evidence before award according to policy.

### 6.2 Confidence dimensions

The score or rule set should be category-specific and test-set validated. Dimensions should include:

- identifier exactness;
- manufacturer/model/variant match;
- condition and warranty equivalence;
- unit and pack equivalence;
- requested quantity and MOQ compatibility;
- geographic and delivery feasibility;
- landed-cost completeness;
- offer freshness and validity; and
- seller/fulfillment evidence available from the provider.

The implementation must support abstention. A numeric score alone must not hide the factors that caused inclusion, qualification, or rejection.

### 6.3 Landed-cost representation

Use a component model rather than one opaque total:

```text
Comparable cost = item subtotal
                + shipping
                + taxes
                + duties
                + provider or payment fees
                - explicit discounts
```

Each component has a state: source-confirmed, calculated, estimated under an approved rule, excluded by policy, or unknown. If a material component is unknown, the result cannot be labeled fully landed.

## 7. AI data lineage and guardrails

AI capabilities may classify, extract, map, explain, and draft. They must operate on a lineage model:

```text
Source fact → extraction proposal → human/provider confirmation
            → derived normalization → comparison explanation
            → human decision
```

Requirements:

- store the model/prompt or rule version necessary for reproducibility within Stampli policy;
- link each extracted field to source location where technically possible;
- keep confidence at field and assessment level;
- separate deterministic financial calculations from generated narrative;
- require review of low-confidence material fields;
- forbid generated values for missing price, quantity, tax, shipping, duty, delivery, warranty, or payment terms;
- record corrections and overrides with actor and reason where required; and
- redact or minimize vendor and user data sent to model providers according to approved data policy.

For “Ask to improve” requests, the final outbound language—including buyer edits—must pass deterministic disclosure validation at send time. Authorized buyers may see cross-vendor gaps internally, but those values cannot populate the vendor message. When customer policy allows, the message may use current comparable public-market evidence and a delta calculated only against the recipient’s own response. Indicative, no-result, unavailable, expired, or superseded evidence cannot support a quantitative outbound market claim. The message must never reveal another vendor's identity, exact response, rank, or a difference from which a sealed response can be derived.

## 8. Authorization model

The production permission map must be validated against Stampli roles. The minimum conceptual matrix is:

| Capability | Requester | Buyer/finance | Approver | Vendor | Auditor/AP |
|---|---:|---:|---:|---:|---:|
| Edit need before publication | Allowed by policy | Allowed | View | No | View |
| Configure/publish event | No | Allowed | View | No | View |
| View all responses | No by default | Allowed | Allowed | Never | As authorized |
| Edit vendor source value | No | Correction with audit | No | Own response | No |
| Review benchmark | Limited/status | Allowed | Allowed | Never | As authorized |
| Request exception | As policy permits | Allowed | As policy permits | No | View |
| Approve exception/award | No unless separately authorized | Only if role permits | Allowed | Never | No unless authorized |
| View audit record | Limited | Allowed | Allowed | Own actions only | Allowed |

All protected actions and fields require authoritative server-side checks. Vendor principals must be isolated by tenant, event, and participant. Identifier secrecy alone is insufficient.

## 9. Security and privacy requirements

### 9.1 Vendor access

- Use event-and-participant-scoped, expiring, revocable access.
- Apply the approved verification level based on customer risk and response confidentiality.
- Do not expose other participants, response counts beyond policy, prices, benchmark data, or internal analysis.
- Protect attachments with authorization on every access, not only at link generation.
- Scan supported uploads before processing or distribution.

### 9.2 Marketplace credentials and data

- Store provider credentials in approved secret management with rotation and least privilege.
- Separate tenant-specific and platform credentials where provider agreements require it.
- Maintain a source-by-source data inventory covering permitted use, display, cache, snapshot, analytics, retention, and deletion.
- Do not include marketplace source content or vendor PII in logs unless explicitly approved and protected.

### 9.3 Commercial confidentiality

- Encrypt data in transit and at rest.
- Audit reads, downloads, exports, corrections, and high-consequence actions.
- Validate retention, legal hold, deletion, and customer export behavior.
- Threat-model cross-tenant access, cross-vendor access, token forwarding, attachment enumeration, stale authorization, and prompt/data exfiltration.

## 10. Consistency and concurrency

The following operations require optimistic concurrency, version checks, or equivalent protection:

- specification amendment after publication;
- vendor response revision;
- buyer entry or correction of a response received outside the portal;
- improvement-request send and revised-response receipt;
- extraction correction while processing completes;
- benchmark refresh while a decision is open;
- award confirmation against evidence versions;
- award replacement;
- PO handoff retry; and
- PO/invoice variance recalculation after source changes.

Award confirmation must bind to the exact specification, response, benchmark, exception, and approval versions the approver reviewed. A stale confirmation is rejected and returned for review.

## 11. Reliability and failure isolation

- Use durable jobs for invitation delivery, extraction, benchmark retrieval, refresh, notifications, PO handoff, and variance calculation.
- Define timeouts, retry classes, maximum attempts, dead-letter/operational review, and idempotency per job type.
- Treat validation/authentication errors as non-retryable unless inputs change.
- Preserve effective award state when downstream handoff fails.
- Maintain provider health, error-rate, latency, rate-limit, retry, and circuit-breaker signals.
- Provide tenant and provider kill switches that stop new external calls without deleting existing evidence.
- Rollback must preserve committed event, response, exception, award, and audit records.

## 12. Observability

Minimum operational signals:

- policy evaluation count, latency, incomplete state, and rule conflicts;
- invitations attempted/delivered/failed and time to first response;
- response save/submission/error and attachment-processing outcomes;
- response channel, buyer-entry validation, missing durable evidence, and source-conflict outcomes;
- extraction latency, field confidence, correction rate, and material error;
- benchmark applicability, provider coverage, latency, rate limit, failure, candidate count, confidence, and result state;
- comparison opened, stale context, unresolved material field, and recommendation override;
- improvement request drafted/sent/failed/expired, disclosure validation, and revised-response outcome;
- exception and award attempts, authorization failures, idempotent duplicates, and completion;
- PO handoff latency, retries, failures, and duplicate prevention;
- variance calculation coverage, material exceptions, and realized-value completeness; and
- audit write/read failure and restricted export activity.

Use tenant-safe correlation identifiers across request, event, provider call, processing job, award, PO, and invoice. Alerts and dashboards require ownership before beta.

## 13. Proposed integration sequence

| Step | Boundary | Input | Output |
|---|---|---|---|
| 1 | Purchase/policy | Current request version | Versioned bid and benchmark outcomes |
| 2 | Bid orchestration | Approved need and confirmed specification | Published private event and participants |
| 3 | Vendor communication | Participant and specification version | Delivery state and scoped access |
| 4 | Vendor response/document processing | Portal response or buyer-entered source evidence | Versioned source facts, channel provenance, and extraction proposals |
| 5 | Benchmark orchestration | Confirmed item, quantity, destination, provider coverage | Versioned candidates and assessments; may run in parallel with step 4 |
| 6 | Comparison | Current valid response versions and benchmark assessments | Source-linked decision context and event-readiness state |
| 7 | Ask to improve (optional) | Recipient’s current response, final reviewed draft, disclosure policy, and current comparable-market evidence when used | Audited external message, improvement-request state, and revised response version |
| 8 | Decision/approval | Current decision-context version, selection, rationale | Effective award version |
| 9 | PO domain | Effective award | Idempotent PO draft/handoff reference |
| 10 | Invoice/matching | Award, PO, invoice/receipt data | Versioned variance and realized outcome |

## 14. Delivery sequence

1. Production-domain discovery and data/permission mapping.
2. Event, specification, participant, response, and audit lifecycle without AI or benchmark automation.
3. Portal and buyer-entered response capture with human-confirmed document extraction and normalized bid comparison.
4. Benchmark provider proof of concept and labeled match-quality evaluation.
5. Provider abstraction, asynchronous retrieval, evidence storage, and failure states.
6. Combined comparison, one controlled improvement-request path, and stale-context-safe human award.
7. PO handoff, invoice variance, analytics, and controlled rollout.

This sequence protects the core competitive-bidding loop from marketplace uncertainty while keeping the benchmark in the Phase 1 product contract.

### 14.1 Twelve-week build-to-pilot planning frame

Plan against a capped 12-week path to a production pilot, subject to the discovery gates below. The timebox covers a narrow category allowlist, one validated benchmark provider, tenant-level controls, production-pilot instrumentation, and human-controlled awards. It does not include category expansion, multiple provider integrations, autonomous supplier negotiation, or strategic-sourcing breadth.

The plan remains valid only if production discovery confirms reusable request, policy, authorization, vendor-access, messaging, PO, invoice, audit, and rollout capabilities early enough to avoid rebuilding those foundations. Provider access, display and retention rights, secure quote-only vendor access, and award-to-invoice linkage are explicit schedule gates. If a gate fails, the team must narrow the pilot, sequence bidding before automated benchmarking, or re-estimate rather than silently reducing control integrity.

Production-pilot readiness requires:

- the five-stage operating flow from definition through handoff works end to end;
- usage and outcome instrumentation is live;
- metric definitions, baselines, cohort rules, and sample thresholds are locked;
- the initial design-partner cohort and category allowlist are confirmed;
- zero material benchmark matching or normalization errors reach an award; and
- provider and tenant kill switches, rollback, alert ownership, and evidence-preserving failure behavior are operational.

## 15. Validation required before engineering commitment

- Locate authoritative production domains and owners.
- Confirm whether existing workflow policy supports the required state and rule model.
- Confirm quote-only vendor identity and secure external-response capabilities.
- Confirm durable evidence rules and authorization for buyer-entered responses.
- Validate document extraction provenance and correction patterns.
- Complete provider legal/commercial/API feasibility and benchmark evidence retention design.
- Define category-specific labeled data and confidence thresholds.
- Define event early-close rules and improvement-request disclosure policy.
- Map award fields to PO and invoice schemas.
- Approve authorization, segregation of duties, retention, PII, and commercial-confidentiality controls.
- Set production SLOs, alert owners, rollout controls, and support runbooks.
