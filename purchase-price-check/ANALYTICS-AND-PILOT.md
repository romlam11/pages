# Purchase Price Check — Analytics and pilot plan

**Purpose:** Prove customer value, workflow viability, benchmark trust, and downstream realization before broad expansion.  
**Updated:** 2026-08-21

## 1. Measurement principles

1. Measure realized value at the PO/invoice outcome when possible.
2. Label every baseline; request estimate, first quote, last-paid price, benchmark, average bid, and budget are not interchangeable.
3. Separate identified opportunity, awarded improvement, and realized improvement.
4. Measure benchmark abstention and failure as first-class outcomes; high coverage with incorrect matches is failure.
5. Segment by customer, category, policy outcome, benchmark provider, match method, quantity band, and vendor status.
6. Report denominators and exclusions with every percentage.
7. Keep provider content, vendor PII, and confidential commercial values out of analytics unless explicitly approved.

## 2. Metric tree

### North star

**Verified invoice savings per $1 million of eligible spend checked.**

The pilot hypothesis is **$20,000–$50,000 per $1 million**, equivalent to 2–5%. The numerator includes only savings with a locked pre-check counterfactual, effective award, and matched downstream invoice outcome. The denominator is the eligible request value included in the matured checked-spend cohort; reports must state the cohort window, completion rule, exclusions, and downstream-evidence maturity. Non-price value such as faster delivery or better terms is reported separately unless an approved financial conversion exists.

### Input and guardrail metrics

| Layer | Metric | Definition | Proposed pilot interpretation |
|---|---|---|---|
| Eligibility | Eligible spend | Non-contracted request amount that policy classifies required/recommended | Establish baseline by customer/category |
| Adoption | Valid-check completion | Events reaching effective award or approved exception ÷ eligible requests | ≥60% in controlled beta |
| Supplier engagement | Valid response rate | Vendors submitting a valid response ÷ vendors invited | ≥60% |
| Invitation operations | Invitation delivery rate | Successfully delivered invitations ÷ invitation attempts | Diagnose contact and channel failures separately from vendor participation |
| Response channel | Buyer-entered response rate | Valid responses recorded by an authorized buyer ÷ all valid responses | Establish baseline and compare correction/error rates by channel |
| Competition | Valid bids per event | Count of valid invited-vendor responses at decision | Track distribution; no benchmark included |
| Offer improvement | Improvement response rate | Vendors submitting a revised response ÷ improvement requests delivered | Establish baseline by category and vendor type |
| Offer improvement | Incremental improvement | Prior response economics minus current revised response economics | Diagnostic; report only with the same specification and labeled basis |
| Efficiency | Active buyer time | Measured active time for specification, event, review, and award; excludes waiting | Median <15 minutes |
| Cycle time | Vendor invitation to recorded decision | Elapsed business time | Median ≤3 business days in pilot categories |
| Benchmark applicability | Benchmarkable rate | Items classified benchmarkable ÷ evaluated items | Diagnostic, segmented by category |
| Benchmark coverage | Useful-result rate | Comparable or indicative results ÷ benchmark attempts completed | Establish category/provider baseline |
| Benchmark precision | Material match-error rate | Reviewed results with wrong product/variant/condition/quantity basis ÷ reviewed results | Zero errors reaching award; minimize pre-award errors |
| Benchmark completeness | Landed-cost completeness | Useful results with all material cost components known ÷ useful results | Diagnostic by provider/category |
| AI quality | Field acceptance | Extracted material fields accepted without correction ÷ reviewed extracted material fields | ≥90% before expansion |
| Trust | Recommendation override | Eligible decisions selecting a different vendor ÷ recommendations shown | Diagnostic; requires rationale analysis, not a “lower is better” target |
| Control | Approved exception rate | Approved exceptions ÷ required checks | Monitor by reason, customer, category, requester |
| Outcome | Awarded improvement | Locked pre-check counterfactual minus effective award, normalized for quantity | Diagnostic precursor to the $20K–$50K realized-value hypothesis |
| Realization | Realized improvement | Approved baseline minus matched invoice economics | Primary value proof |
| Integrity | Material award variance | Awards with unexplained material PO/invoice variance ÷ awards with downstream data | Establish baseline and reduce by cohort |

The core operating scorecard has four gates: valid-check completion, valid vendor response, active buyer time, and vendor-invitation-to-recorded-decision cycle time. Expansion additionally requires verified value and zero material benchmark errors at award; the remaining measures diagnose quality, control integrity, and scalability.

## 3. Savings and value contract

### 3.1 Required fields

Every calculated value record must include:

- tenant and event identifiers;
- category and event currency;
- baseline type and source version;
- baseline unit, quantity, price components, and amount;
- award version and amount;
- PO/invoice versions and realized amount, when available;
- normalization/calculation version;
- excluded or unknown components;
- calculation timestamp; and
- state: identified, awarded, partially realized, realized, invalidated, or unavailable.

### 3.2 Baseline hierarchy

No one baseline fits every decision. Use an explicitly approved basis:

| Baseline | Appropriate use | Limitation |
|---|---|---|
| Initial vendor quote | Negotiation or competition from a known starting offer | Can reward inflated anchors |
| Last-paid price | Repeat purchases with comparable quantity/specification | May be stale or affected by prior terms |
| Comparable public-market benchmark | Standardized goods with credible landed-cost evidence | Evidence is not a committed supplier bid |
| Average valid bid | Dispersion and selected-price comparison | Sensitive to invited-vendor set |
| Request estimate/budget | Budget variance | Not evidence of market value |
| Contract price | Compliance and leakage | Applies only when contract and item match |

Do not add savings calculated from different baselines. Report each basis separately or apply an approved precedence rule.

For pilot value reporting, lock the approved counterfactual before the decision is recorded. Later marketplace movements, revised estimates, or post-award baseline changes must not rewrite the value basis.

### 3.3 Realization

```text
Awarded improvement = normalized baseline − effective award
Realized improvement = normalized baseline for fulfilled quantity − invoiced economic cost
```

Shipping, tax, duty, rebates, credits, partial receipts, returns, and split invoices must follow a documented inclusion policy. If they cannot be reconciled, report partial or unavailable realization rather than a complete savings claim.

## 4. Event taxonomy

Names are conceptual; production naming must follow Stampli conventions.

| Event | Trigger | Minimum analytical properties |
|---|---|---|
| purchase_check_evaluated | Bidding and benchmark applicability committed | Request, policy/version, bidding outcome, benchmark outcome, reason codes, category, amount band |
| purchase_check_created | Event created | Event/request, owner type, policy outcome, line count, vendor target count |
| specification_confirmed | Specification version confirmed | Event, version, completeness, identifier types, benchmarkable line count |
| invitation_attempted | Vendor invitation queued | Event, participant type, channel, attempt number |
| invitation_delivered | Delivery confirmed | Event, participant, latency |
| invitation_failed | Delivery fails terminally or retryably | Event, participant, reason class, retryable flag |
| vendor_response_saved | Vendor draft persists | Event, response version, structured/uploaded mode |
| vendor_response_submitted | Valid response committed | Event, response version, time since delivery, completeness |
| vendor_response_recorded_by_buyer | Authorized buyer records a response from another approved channel | Event, response version, channel, evidence type, entering role, completeness |
| quote_extraction_completed | Extraction finishes | Document type, material-field count, confidence bands, latency |
| quote_field_corrected | Human/vendor changes proposal | Field class, prior confidence, corrector role, material flag |
| benchmark_attempt_started | Eligible item queued | Event/line, provider, match method, category, destination region |
| benchmark_attempt_completed | Attempt reaches terminal useful/no-result state | Provider, latency, candidate count, result state, confidence, freshness |
| benchmark_attempt_failed | Attempt fails | Provider, stage, error class, retry count, terminal flag |
| benchmark_candidate_rejected | Candidate fails validation/match | Provider, rejection reasons, match method |
| comparison_viewed | Authorized user opens current decision context | Event, evidence versions, valid-bid count, benchmark state |
| collection_closed | Event becomes ready for comparison | Event, close basis, pending/declined counts, early-close actor/reason when applicable |
| recommendation_generated | Analysis created | Event, model/rule version, confidence, unresolved-material-field count |
| improvement_request_sent | Final reviewed “Ask to improve” request is delivered to a vendor | Event, participant, prior response version, benchmark state and evidence version when used, outbound claim type, disclosure-validation result, deadline, sender role |
| improvement_request_resolved | Revised response arrives or request expires/cancels | Event, participant, result, response version, elapsed time, commercial delta basis |
| improvement_request_blocked | Final message fails disclosure validation | Event, participant, policy/rule version, prohibited-claim category, actor role; exclude message text and commercial values from analytics payloads |
| exception_requested | User requests waiver | Requirement, reason code, requester role |
| exception_resolved | Exception approved/rejected | Requirement, reason, approver role, latency |
| award_attempted | User confirms decision | Event, selected-response version, lowest-bid flag, benchmark state |
| award_recorded | Effective award commits | Event, award version, rationale category, selected rank, baseline types |
| po_handoff_completed | Downstream PO reference commits | Event, award version, latency, mapped-field completeness |
| po_handoff_failed | Handoff fails | Event, stage, retry count, terminal flag |
| variance_assessed | Award compared to PO/invoice | Event, object type, material flag, variance components, calculation version |
| realized_value_updated | Outcome state changes | Event, state, baseline type, fulfilled quantity band, amount band |

## 5. Funnel definitions

```text
Purchase evaluated
  → eligible/recommended
  → event created
  → specification confirmed
  → invitations delivered
  → minimum valid responses received
  → comparison reviewed
  → award or approved exception
  → PO created
  → invoice/receipt matched
  → realized outcome measured
```

Each conversion must use a declared cohort rule:

- event-time cohort for product funnel and cycle-time metrics;
- award-time cohort for award and handoff quality;
- invoice-time or matured event cohort for realized value; and
- separate treatment for cancellations, policy changes, test tenants, and incomplete data.

## 6. Pilot design

### 6.1 Participants

- Five to ten design partners with finance-led purchasing teams.
- Two or three standardized-goods categories per partner.
- Categories should have stable identifiers, repeat volume, multiple viable vendors, and a feasible benchmark provider.
- Exclude categories with regulatory, safety, counterfeit, installation, or complex-service risk until dedicated controls exist.

### 6.2 Baseline period

Measure at least the following before feature exposure:

- eligible request volume and spend;
- current quote attachments and valid bids per purchase;
- policy compliance and exception rate;
- vendor response rate and time;
- portal versus buyer-entered response volume, completeness, correction, and source-conflict rates;
- buyer active time and source-to-award cycle;
- available last-paid/contract/estimate baselines;
- PO/invoice variance; and
- current marketplace checking behavior, if observable through research.

### 6.3 Rollout cells

Use a staggered or matched-cohort design where volume permits:

- **Control/current workflow:** existing process and policy.
- **Bidding-only:** structured event and vendor comparison, benchmark not shown.
- **Bidding + benchmark:** full relevant benchmark experience.

If sample size is too small for causal inference, use within-customer matched categories and qualitative review. Do not present directional pilot observations as statistically proven lift.

### 6.4 Review cadence

- Daily during initial alpha for material match, confidentiality, or award errors.
- Weekly for funnel, response, benchmark coverage, field corrections, cycle time, exceptions, and support burden.
- Weekly for early-close use, improvement-request disclosure validation, revised-response rate, and incremental value.
- Monthly or after a defined matured cohort for award-to-invoice realized value.

## 7. Expansion gates

### Expand

- zero material benchmark match/normalization errors reach award;
- at least 60% of invited vendors submit a valid response;
- at least 60% eligible-request completion by award or approved exception;
- median active buyer time below 15 minutes;
- median vendor-invitation-to-recorded-decision cycle at or below three business days;
- at least 90% material extracted fields accepted without correction; and
- zero “Ask to improve” messages reveal or make a sealed competing bid derivable, and zero quantitative market claims use a benchmark state other than current comparable evidence; and
- evidence supports $20,000–$50,000 of verified invoice savings per $1 million of eligible spend checked in selected categories, or another validated, material non-price outcome.

### Iterate

Value is visible but one or more of benchmark coverage, vendor participation, buyer effort, cycle time, exception abuse, or downstream realization misses target. Limit iteration to named categories and root causes.

### Stop or reposition

- any unresolved confidentiality or material benchmark-integrity failure;
- verified improvement remains below 1% while the process adds more than three business days after two category-specific iterations;
- benchmark access cannot be made legally and operationally reliable for target categories; or
- supplier and buyer participation is insufficient to produce credible competition.

Potential repositioning: policy evidence and structured bidding without automated external benchmarking, or internal/contract price assurance where customer data is stronger.

## 8. Reporting views

### Customer operational view

- eligible, active, completed, exception, and overdue checks;
- vendor response and cycle-time distribution;
- response channel, early-close, improvement-request, and revised-response outcomes;
- benchmark result states and category coverage;
- award rank, non-lowest rationale, and unresolved variance;
- identified, awarded, partially realized, and realized value by baseline.

### Product pilot view

- activation and funnel by cohort;
- provider/category coverage and error taxonomy;
- extraction and benchmark correction quality;
- buyer and vendor friction;
- buyer-entered response quality and improvement-request effectiveness;
- support incidents and provider operational health;
- result maturity and realized-value completeness.

## 9. Analytics quality requirements

- Version event and metric schemas.
- Validate required properties at emission and ingestion.
- Use idempotent event identifiers for retried jobs and state commits.
- Reconcile critical events to source-of-truth lifecycle counts.
- Track missing-property and late-arrival rates.
- Document timezone and business-day handling.
- Restrict exact commercial values and PII to approved analytics stores and roles.
- Maintain a deletion/retention strategy consistent with customer and provider obligations.
