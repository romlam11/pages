# Product Requirements Document: Purchase Price Check


## 1. Product decision

Purchase Price Check lets finance and procurement teams run a lightweight, private bid before committing to an operational purchase. Stampli creates one comparable specification, collects invited-vendor offers, and adds a public-market benchmark when the item can be matched reliably. An authorized person selects the vendor and records the rationale; the decision flows into the PO and is checked against invoices.

The product targets operational purchases that merit competition but bypass formal sourcing because it is too manual or heavy. It is not a strategic-sourcing suite.

### Product promise

> Before the company commits to a defined purchase, Stampli shows whether invited bids are commercially reasonable using comparable vendor offers and, when relevant, current public-market evidence.

### Recommendation

Build Phase 1 for standardized goods and repeatable categories. Benchmark only when relevance and matchability rules pass; treat “no reliable benchmark found” as a valid, auditable outcome.

## 2. Problem and opportunity

Purchase approval answers whether an organization is allowed to buy. It does not consistently answer whether the proposed price and terms are competitive.

Today, many mid-market teams collect quotes by email, normalize PDFs in spreadsheets, search marketplaces manually, and attach incomplete evidence after deciding. As a result:

- bidding policy is applied inconsistently;
- vendor responses are difficult to compare;
- public prices are checked without a consistent landed-cost basis; and
- the award rationale is disconnected from the PO and invoice outcome.

Stampli can connect the request, evidence, decision, PO, invoice, and realized result in one auditable flow.

### Primary users

| User | Core need |
|---|---|
| Finance or procurement manager | Run a compliant competitive check without extensive manual normalization |
| Approver or budget owner | Select the strongest total value using concise, source-visible evidence |
| Requester | Obtain an approved purchase through one clear process |
| Vendor respondent | Submit a private quote quickly without full onboarding |
| Auditor or AP stakeholder | Verify that policy and awarded economics were honored |

## 3. Goals and Phase 1 boundary

### Goals

1. Apply customer bidding and benchmarking policy consistently.
2. Make two to five vendor offers easy to request and compare.
3. Add a trustworthy external price signal when a purchase is benchmarkable.
4. Preserve human control over supplier contact, exceptions, award, and commitment.
5. Connect the decision to the PO and invoice so value can be verified.

### Phase 1 scope

Phase 1 supports an invite-only event for a defined purchase, typically with two to five vendors and one bidding round. It includes policy checks, a common specification, secure or buyer-entered responses, offer normalization, conditional public benchmarking, one optional improvement request per vendor, human award and exceptions, PO handoff, audit evidence, and PO/invoice variance detection.


## 4. Phase 1 experience

1. A requester submits the items or scope, quantity, location, required date, and estimated value.
2. Stampli checks customer policy and identifies information needed for comparable offers.
3. Finance or procurement approves the common specification, vendors, deadline, and benchmark attempt.
4. Vendors receive the same private request and respond through a secure, event-scoped path. An authorized buyer may record a response from an approved channel with durable source evidence.
5. Stampli extracts and normalizes offers while preserving original values, and retrieves public-market evidence when benchmark rules pass.
6. The approver compares offers, benchmark evidence, landed cost, delivery, terms, warranty, vendor status, and exceptions. The buyer may ask each vendor to improve once under the confidentiality rule below.
7. An authorized person selects the vendor and confirms or edits the rationale. The award continues to final approval and PO creation without re-entry.
8. Stampli compares the PO and invoice with the award and records material variance. Value remains pending until downstream evidence exists.

### Five-stage operating model

The five stages are: **Define and check policy → Invite 2–5 vendors → Collect offers and check the market → Compare and ask to improve → Award and hand off**.

Award-to-PO readiness and PO/invoice assurance are distinct. Awarded improvement may appear after the award; realized improvement requires linked and assessed PO, invoice, or receipt evidence.


## 5. Prioritized capabilities

| FR | Capability | Priority |
|---|---|---|
| FR-001 | Policy and applicability | Blocker |
| FR-002 | Comparable specification | Blocker |
| FR-003 | Private bid event | Blocker |
| FR-004 | Secure vendor response | Blocker |
| FR-005 | Quote normalization and source integrity | Blocker |
| FR-006 | Relevant public-market retrieval | Blocker |
| FR-007 | Benchmark landed-cost normalization | Blocker |
| FR-008 | Bid and benchmark comparison | Blocker |
| FR-009 | Human-authorized award | Blocker |
| FR-010 | Audit trail and PO handoff | Blocker |
| FR-011 | Policy-governed exceptions | Critical |
| FR-012 | Role access and sealed responses | Critical |
| FR-013 | Award-to-PO/invoice variance | Critical |
| FR-014 | Benchmark uncertainty and failure states | Critical |
| FR-015 | Operational and outcome analytics | Must have |
| FR-016 | One buyer-authorized improvement request per vendor | Must have |

Phase 1 cannot launch without the Blocker and Critical requirements. Full acceptance criteria are in [FUNCTIONAL-REQUIREMENTS.md](./FUNCTIONAL-REQUIREMENTS.md).

## 6. Human authority and essential rules

AI may classify purchases, identify missing specifications, draft requests, extract and normalize fields, explain tradeoffs, and prepare PO fields. A person must authorize the external specification, recipients, corrected or low-confidence values, exceptions, supplier communications, award, rationale, and financial commitment.

The following rules define the Phase 1 product boundary:

1. Every event uses one common specification, deadline, event currency, and materially identical requirements for all vendors.
2. Source facts remain separate from derived values; formulas, inputs, corrections, overrides, and decisions are auditable.
3. Vendor responses remain sealed according to customer policy, and suppliers never receive competing responses.
4. The benchmark does not satisfy bidding policy and never becomes a vendor merely because its price is displayed.
5. A non-lowest award requires a rationale; customer policy may require additional approval.
6. A buyer may ask each vendor to improve once, referencing only that vendor’s response and policy-approved public evidence. The message may not reveal another vendor’s identity, bid, rank, or derivable bid gap.
7. Only one award is effective at a time. A replacement creates a new decision version.
8. Value may be called realized only after the applicable PO, invoice, or receipt evidence has been linked and assessed.

## 7. Success and pilot recommendation

### North-star outcome

**Verified invoice savings per $1 million of eligible spend checked.**

Target: **$20,000–$50,000 per $1 million (2–5%)**, counting only invoice-verified savings against a locked pre-check baseline. Reports must state the denominator, matured-cohort rules, and baseline used (initial quote, request estimate, last-paid price, public benchmark, or average valid bid).

### Pilot gates

| Measure | Proposed gate |
|---|---|
| Workflow completion | At least 60% of eligible requests complete a valid check or approved exception |
| Vendor response | At least 60% of invited vendors submit a valid response |
| Buyer effort | Median active buyer time per completed check is under 15 minutes, excluding supplier waiting |
| Business-day cycle time | Median time from vendor invitation to recorded decision is at most three business days |
| Benchmark trust | Zero material matching or normalization errors reach award |
| Verified value | $20,000–$50,000 of invoice savings per $1 million checked, against a locked pre-check baseline |

Pilot two or three standardized categories using one validated benchmark source, customer-level controls, and human review of matches. Expand only if all operating gates pass, value is verified, and no material benchmark error reaches award.

### Build-to-pilot recommendation

Plan for a capped 12-week build-to-production pilot, subject to early validation of domain ownership, provider feasibility, evidence rights, and reuse of existing authorization and downstream capabilities. If those cannot be resolved within the timebox, narrow, resequence, or re-estimate the pilot. This plan does not approve category expansion.

Metric definitions, instrumentation, category analysis, and go/iterate/stop gates are in [ANALYTICS-AND-PILOT.md](./ANALYTICS-AND-PILOT.md).

## 8. Launch blockers and open decisions

Production discovery is required before confirming architecture, ownership, estimates, or contracts.

| Decision or dependency | Recommended direction | Owner |
|---|---|---|
| Benchmark provider | Select for pilot-category coverage, licensed access, landed-cost data, and evidence-retention rights | Product, Legal, Engineering |
| Comparable-confidence threshold | Validate by category on labeled test sets; abstain below threshold | Product, Data |
| Evidence retention | Store only what provider terms permit; disclose limitations to users | Legal, Security |
| Workflow ownership | Confirm request, vendor, policy, authorization, audit, PO, and invoice integration owners | Architecture, domain owners |
| Improvement-request disclosure | Allow comparable public evidence; prohibit disclosure of competing bids or derivable gaps | Product, Legal |

The complete evidence register, dependency map, risk register, and unresolved decisions are maintained in [ASSUMPTIONS-RISKS-DECISIONS.md](./ASSUMPTIONS-RISKS-DECISIONS.md).

## Supporting documents

- [FUNCTIONAL-REQUIREMENTS.md](./FUNCTIONAL-REQUIREMENTS.md) — testable functional and non-functional requirements
- [TECHNICAL-FOUNDATION.md](./TECHNICAL-FOUNDATION.md) — proposed domain model, integration contract, security, and observability
- [ANALYTICS-AND-PILOT.md](./ANALYTICS-AND-PILOT.md) — metric contracts, instrumentation, and pilot design
- [ASSUMPTIONS-RISKS-DECISIONS.md](./ASSUMPTIONS-RISKS-DECISIONS.md) — evidence, dependencies, risks, and open decisions
