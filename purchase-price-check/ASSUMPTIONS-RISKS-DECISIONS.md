# Purchase Price Check — Assumptions, risks, decisions, and evidence

**Purpose:** Keep the assessment honest about what is known, hypothesized, proposed, and unresolved.  
**Updated:** 2026-08-21

## 1. Assessment position

The opportunity is credible, but the winning scope is narrow. Purchase bidding places Stampli closer to Coupa, Zip, Ramp, Fairmarkit, and other procurement platforms. Stampli should not claim a general sourcing advantage without evidence. The proposed wedge is:

> Lightweight, finance-controlled purchase bidding with a relevant public-market price benchmark, connected directly to the PO and invoice outcome.

The benchmark strengthens the feature because it answers a common approver question even when three invited quotes cluster around the same price. It also increases execution risk: marketplace access, product matching, landed-cost completeness, and evidence rights must be treated as product dependencies, not implementation details.

## 2. Evidence classes

| Class | Meaning | How it is used |
|---|---|---|
| Publicly documented fact | A current public source describes a product or market behavior | Supports market validation and feasibility questions |
| Absence inference | Reviewed public pages did not expose a capability | Signals a question; never proves a private feature or roadmap is absent |
| Product hypothesis | A proposed customer behavior or outcome | Requires discovery or pilot validation |
| Technical proposal | A recommended production contract without codebase evidence | Requires architecture and engineering validation |

## 3. Public evidence

### Stampli adjacency

- Stampli publicly documents purchase intake, approval workflows, preferred purchasing behavior, purchase orders, invoice matching, and Billy-assisted procurement capabilities. These establish workflow adjacency, not production implementation details for this feature.
- Stampli publicly teaches a lightweight “three bids and a buy” RFQ process, supporting the target operational-purchase use case.
- Publicly reviewed Stampli product pages did not establish a native multi-vendor bid and benchmark workflow. This is an absence inference only.

### Competitive validation

- Ramp publicly documents Vendor Sourcing with AI-assisted RFx creation, vendor response collection, comparison, and human award. This validates demand and represents a close functional threat.
- Zip, Fairmarkit, Omnea, and Pactum publicly describe AI-assisted sourcing, comparison, intake, or negotiation capabilities. This indicates category convergence, not whitespace.
- The strategic mitigation is scope discipline and downstream financial continuity, not a novelty claim.

### Marketplace and benchmark validation

- Amazon Business publicly documents an RFQ flow for eligible high-value or high-quantity purchases and an Integrated Quoting relationship with Fairmarkit. This demonstrates that a marketplace can participate through an authorized procurement integration.
- Amazon Business's published thresholds and seller-response timing apply to its own offering and should not be generalized.
- Alibaba.com publicly documents an RFQ marketplace in which buyers post needs and sellers respond. This validates supplier-network behavior, not a general public-price API suitable for automatic benchmarking.
- Ramp and Omnea/Tropic publicly document price-intelligence experiences and also illustrate that benchmark coverage and accuracy depend on the underlying data source.

## 4. Product hypotheses

| ID | Hypothesis | Validation method | Failure implication |
|---|---|---|---|
| H-01 | Finance-led mid-market teams have enough non-contracted, quote-appropriate purchases to justify the feature | Customer spend/policy analysis and request sample | Narrow categories/customers or stop |
| H-02 | Two to five vendors, an initial response, and at most one controlled improvement request per vendor cover most valuable lightweight bidding events | Review historical quote attachments, negotiation behavior, and customer interviews | Move improvement requests to Phase 2 or expand only with evidence |
| H-03 | Most responses can use a secure no-account path, while an audited buyer-entry path captures quotes received through approved external channels | Usability test and pilot response funnel by channel | Simplify the portal or formalize external-channel ingestion |
| H-04 | At least 60% of invited vendors submit a valid response | Pilot cohort | Vendor participation becomes the primary product problem |
| H-05 | Standardized goods can be benchmarked with acceptable precision using approved provider data | Labeled pilot-category evaluation | Limit benchmark categories/providers or use manual evidence only |
| H-06 | Showing source, assumptions, unknowns, and confidence creates sufficient trust | Decision usability tests and correction/override data | Rework evidence model; do not increase automation |
| H-07 | Buyers can complete active work in under 15 minutes | Instrumented pilot | Reduce fields, automation, and policy scope |
| H-08 | Selected categories produce $20,000–$50,000 of verified invoice savings per $1 million of eligible spend checked, using a locked pre-check counterfactual | Award and invoice reconciliation | Reposition as control/audit feature or stop expansion |
| H-09 | Connecting the award to PO/invoice creates credible realized-value evidence | Production data mapping and pilot | Limit savings claims and resolve downstream integration first |
| H-10 | Relevant benchmarks improve decisions beyond invited bids alone | Bidding-only versus bidding-plus-benchmark cohort or matched analysis | Keep bidding core; narrow or remove automated benchmark |
| H-11 | The median time from vendor invitation to recorded decision is at most three business days in pilot categories | Instrumented pilot | Narrow policy scope, improve supplier follow-up, or reposition the workflow |

## 5. Technical and organizational assumptions

These are not verified:

- Stampli has an authoritative purchase request object that can own or link a bid event.
- Existing policy and approval capabilities can express bid thresholds, benchmark relevance, exception routing, non-lowest selection, and separation of duties.
- A quote-only vendor/contact can be represented without full onboarding.
- Existing messaging and external access can be extended safely for sealed responses.
- Billy or another document service can return source-linked extraction, field confidence, and human corrections.
- Confirmed award values can flow into the PO domain without re-entry.
- PO, receipt, and invoice data can be linked at sufficient line-level quality for variance and realization.
- The audit platform can retain material versions and evidence under customer and provider obligations.
- A marketplace partner will permit required search, display, and decision-time evidence retention for pilot categories.

The current planning hypothesis is a capped 12-week build-to-production-pilot, not category expansion. It depends on early resolution of the ownership, reuse, provider, evidence-rights, and downstream-linkage assumptions above. Any estimate made before those assumptions are resolved should include explicit discovery contingency and an explicit rescope or re-estimation trigger.

## 6. Risk register

| Risk | Likelihood | Impact | Early signal | Mitigation | Owner |
|---|---|---|---|---|---|
| No viable marketplace access for pilot categories | Medium | Critical | Legal/API review finds no licensed search and retention path | Evaluate multiple providers; use customer-authorized evidence fallback; preserve bidding core | Needs owner |
| Incorrect product match reaches decision | Medium | Critical | Variant/condition corrections or buyer distrust | Exact-ID first; category allowlist; labeled test set; confidence threshold; mandatory review; abstain | Product + Data owners TBD |
| Public price excludes material landed-cost components | High | High | Large shipping/duty/tax corrections | Component model, unknown state, provider/category rules, no automatic rejection | Product owner TBD |
| Benchmark is interpreted as an actual purchasable bid | Medium | High | Approver asks to award marketplace result directly | Separate semantic model and UI treatment; benchmark never counts as vendor or bid | Product + Design owners TBD |
| Vendor response is too low | Medium | High | <60% valid response or long delays | No-account response, known vendors first, reminders, email/document ingestion, focused categories | Product owner TBD |
| Buyer-entered bid is marked valid without durable source evidence | Medium | High | Source conflicts, audit gaps, or higher correction rate | Same completeness rules as portal responses, durable evidence required, channel/actor labeling, version history | Product/Engineering owners TBD |
| “Ask to improve” request exposes sealed pricing | Medium | Critical | Vendor can infer a competitor's response from exact gap or rank | Keep cross-vendor gaps buyer-only; validate the final edited message; allow deltas only between approved comparable-market evidence and the recipient’s own response; run security/legal tests | Product/Legal owners TBD |
| Workflow adds too much cycle time | Medium | High | >3 business days median or exception spikes | Policy thresholds, contract fast path, recommended path, SLA visibility, category tuning | Product owner TBD |
| Confidential response is disclosed | Low | Critical | Security test or incident | Server-side isolation, scoped/revocable access, encryption, threat model, audit, staged rollout | Security owner TBD |
| AI silently changes a commercial fact | Medium | Critical | Material correction after award | Immutable source facts, separate derived values, confirmation, stale-context award check | AI/Engineering owners TBD |
| Production data model cannot connect award to invoice | Medium | High | Missing line-level linkage | Discovery first; make linkage a launch dependency; limit realization claims until resolved | Architecture owner TBD |
| Savings claim is overstated | Medium | High | Baseline disputes or negative invoice variance | Baseline contract, maturity states, invoice reconciliation, transparent exclusions | Product + Data owners TBD |
| Scope expands into sourcing suite | High | High | Requests for RFP, auctions, supplier discovery, contracts | Enforce explicit non-goals and Phase 1 decision gates | Product owner TBD |
| Provider cost/latency harms economics or experience | Medium | Medium | High call cost, rate limits, slow results | Cache within terms, async flow, identifier-first queries, provider health/cost monitoring | Engineering owner TBD |
| Provider terms change | Medium | High | Deprecation or contract notice | Adapter abstraction, terms registry, kill switch, multiple source strategy | Partnerships/Legal TBD |

## 7. Open decisions

Status vocabulary: 🔴 Blocked, 🟠 Open, ⚪ Not yet validated, 🟡 Proposed, 🟢 Decided.

| Decision | Status | Recommended direction | Evidence needed | Owner | Blocks |
|---|---|---|---|---|---|
| Production domain/object that owns a bid event | 🟠 Open | Choose after mapping request, workflow, vendor, PO, invoice, and audit boundaries | Code and architecture review | Needs owner | Estimate and technical design |
| Twelve-week build-to-pilot feasibility | 🟡 Proposed | Treat as a capped planning hypothesis; validate scope and reuse during initial production discovery | Domain mapping, provider feasibility, delivery plan, and staffed owner commitments | Product/Engineering TBD | Pilot commitment |
| First benchmark provider | 🟠 Open | Select by pilot-category fit, licensed access, landed-cost data, retention, and operations | Provider proof of concept and legal/commercial review | Needs owner | FR-006 beta |
| Benchmark comparable-confidence threshold | 🟠 Open | Calibrate by category; prefer abstention over false match | Labeled evaluation set and buyer review | Product/Data TBD | FR-007 beta |
| Rights to preserve decision-time marketplace evidence | 🟠 Open | Retain only permitted fields/content; document limitations | Provider terms and counsel review | Legal/Security TBD | Audit design |
| Failure behavior for a policy-required benchmark | 🟡 Proposed | Bounded retry, evidence fallback, then approved exception; never silently pass | Customer policy interviews and operations review | Product TBD | FR-011/014 |
| Quote-only vendor verification level | 🟠 Open | Risk-tier by customer/event while preserving low friction | Security threat model and vendor usability test | Security/Product TBD | FR-004 beta |
| Minimum valid bid count | 🟡 Proposed | Customer-configurable, normally two valid submissions; invitations alone do not satisfy policy | Customer policy research | Product TBD | FR-001/003 |
| Single currency in Phase 1 | 🟢 Decided | One event currency; conversions only for benchmark evidence with explicit approved rate | Scope decision | Product | None |
| Marketplace evidence counts as a bid | 🟢 Decided | Never; only a formal provider-returned quote intentionally invited into the event could become a response in future scope | Product decision | Product | None |
| Human award authority | 🟢 Decided | Required for every Phase 1 award and exception | Product decision | Product | None |
| Services/custom goods benchmarking | 🟢 Decided | Not benchmarked automatically in Phase 1 unless a later category-specific contract proves comparability | Product decision | Product | None |
| Cross-customer benchmark data | ⚪ Not yet validated | Exclude from Phase 1; require separate consent, privacy, anonymization, quality, and value case | Legal/privacy/data research | Needs owner | Future only |
| Material PO/invoice variance | 🟠 Open | Configurable absolute/percentage/category thresholds with documented basis | Finance customer research and existing matching rules | Product/Finance TBD | FR-013 |
| Event readiness after policy minimum | 🟠 Open | Compare only after all responses/declines, deadline, or an authorized early-close rule with rationale | Customer policy research | Product TBD | FR-003/008 |
| “Ask to improve” disclosure | 🟡 Proposed | Permit a current comparable public-market price/range and its delta to the recipient’s own quote; use non-quantitative language for other benchmark states; prohibit competing identity, bid, rank, or derivable gap; revalidate edits at send | Legal review and vendor-message testing | Product/Legal TBD | FR-012/016 |
| Phase 1 improvement scope | 🟡 Proposed | At most one buyer-authorized improvement request per vendor; no open-ended negotiation | Customer research and pilot cycle-time/value | Product TBD | FR-016 |

## 8. Discovery questions before build commitment

### Customer and business

1. How many requests already contain two or more quote attachments?
2. Which categories and amount thresholds require quotes today?
3. What share of eligible purchases has a valid contract, regulated price, emergency, or sole-source exception?
4. How much active buyer time and elapsed time does the current process consume?
5. Which terms beyond price change the award decision most often?
6. How frequently are invited vendors not yet onboarded?
7. What minimum response count satisfies policy, and are invitations ever sufficient when vendors decline?
8. After the minimum is met, when may the buyer close collection while vendors remain pending?
9. Which baseline do customers accept for savings reporting?
10. How often do awarded, PO, invoiced, and paid economics differ?
11. For which standardized categories do customers already check Amazon, Alibaba, or other marketplaces?
12. How often do buyers request revised quotes, what do they disclose, and what incremental value results?

### Product and architecture

1. Which production object should own event state and versions?
2. Which existing policy, approval, messaging, vendor, attachment, audit, and feature-flag capabilities can be reused?
3. Can quote-only vendor contacts be isolated from approved vendor records?
4. What are the existing role, entity, and separation-of-duties rules?
5. How are line items mapped from request to PO to invoice today?
6. Can the audit system preserve immutable decision evidence and provider-limited references?
7. What Billy extraction and confidence contracts are available?
8. Which analytics and operational standards must this feature inherit?
9. Which approved storage and verification path can preserve quotes received by email or another external channel?

### Marketplace feasibility

1. Which provider offers approved search or quoting access for pilot geographies and categories?
2. Can the integration search by manufacturer part number, GTIN, SKU, or exact model?
3. Which price components, quantity/MOQ, condition, availability, delivery, and seller attributes are returned?
4. What may Stampli display, cache, snapshot, analyze, and retain?
5. Is access tenant-authorized, platform-authorized, or both?
6. What are the rate limits, latency, cost, versioning, support, and deprecation terms?
7. Can a public listing be linked into a formal marketplace RFQ, and if so, when does it become a quote rather than a benchmark?
8. What customer consent or account linking is required?

## 9. Validation artifacts required

Before a beta commitment, produce:

- a production current-state and reuse map;
- category and customer opportunity analysis;
- provider feasibility scorecard and approved terms summary;
- labeled benchmark match-quality set with category-specific thresholds;
- vendor-response usability study;
- buyer-entered response evidence and quality study;
- improvement-message disclosure policy and revised-response pilot;
- authorization and confidentiality threat model;
- data inventory and retention design;
- request-to-award-to-PO/invoice field mapping;
- instrumented pilot plan and baseline report; and
- operational ownership, SLOs, alerts, and support runbooks.

## 10. Sources

All sources were accessed or revalidated in August 2026. Vendor-authored savings and performance figures are treated as vendor claims.

### Stampli

- [Three-bid RFQ guidance](https://www.stampli.com/resources/supplier-selection-rfq-3-bid/)
- [Procure-to-pay](https://www.stampli.com/procure-to-pay/)
- [Billy in Procurement](https://www.stampli.com/billy-in-procurement/)
- [Purchase Orders](https://www.stampli.com/stampli-purchase-orders/)
- [Preferred Items](https://www.stampli.com/preferred-items/)

### Marketplaces and price intelligence

- [Amazon Business Request for Quote](https://business.amazon.com/en/solutions/bulk-buying/request-for-quote)
- [Amazon Business Integrated Quoting](https://business.amazon.com/en/blog/amazon-business-integrated-quoting-feature)
- [Alibaba.com Request for Quotation](https://seller.alibaba.com/rfq)
- [Ramp Price Intelligence](https://support.ramp.com/price-intelligence-on-ramp/)
- [Omnea and Tropic Price Intelligence](https://www.omnea.co/resource/omnea-and-tropic-partner)

### Competitive sourcing

- [Ramp Vendor Sourcing](https://support.ramp.com/vendor-sourcing-in-ramp-procurement/)
- [Fairmarkit](https://www.fairmarkit.com/)
- [Omnea autonomous sourcing](https://www.omnea.co/resource/the-case-for-autonomous-sourcing)
- [Zip AI for RFx](https://ziphq.com/blog/ai-for-rfx)
- [Pactum Requisition Alignment Agent](https://pactum.com/alignment-agents)

### Outcome context

- [APQC sourcing-event savings goal](https://www.apqc.org/what-we-do/benchmarking/open-standards-benchmarking/measures/typical-savings-goal-procurement)
