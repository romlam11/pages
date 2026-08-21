# Purchase Price Check — Functional and quality requirements

**Scope:** Phase 1 purchase bidding with relevant public-market price benchmarking  
**Implementation status:** Greenfield proposal; production implementation not inspected  
**Updated:** 2026-08-21

## 1. Requirement conventions

- **Blocker:** Phase 1 cannot ship without the capability.
- **Critical:** Required for launch and control integrity.
- **Must have:** Required for a measurable pilot, but not the core transaction loop.
- Acceptance criteria describe observable behavior and avoid prescribing a specific interface.
- “Benchmark” means public-market evidence, not a participating vendor bid.
- “Source fact” means a value received from a vendor or benchmark provider before Stampli transformations.
- “Derived value” means a calculation, normalization, classification, or AI-generated explanation.
- “Authoritative enforcement” means the server rejects an unauthorized or invalid state change even if the client is bypassed.

## 2. Requirements summary

| FR | Title | Priority | Phase | Serves | Group |
|---|---|---|---|---|---|
| FR-001 | Determine bidding and benchmark applicability | Blocker | 1 | Finance/procurement, requester | Policy |
| FR-002 | Produce a comparable purchase specification | Blocker | 1 | Requester, finance/procurement | Definition |
| FR-003 | Manage a private bid event | Blocker | 1 | Finance/procurement | Bidding |
| FR-004 | Collect secure vendor responses | Blocker | 1 | Vendor respondent | Bidding |
| FR-005 | Normalize offers without changing source facts | Blocker | 1 | Finance/procurement, vendor | Comparison data |
| FR-006 | Retrieve a relevant public-market benchmark | Blocker | 1 | Finance/procurement, approver | Benchmark |
| FR-007 | Normalize benchmark evidence and confidence | Blocker | 1 | Finance/procurement, approver | Benchmark |
| FR-008 | Compare bids with benchmark evidence | Blocker | 1 | Approver | Decision |
| FR-009 | Record a human-authorized award | Blocker | 1 | Approver | Decision |
| FR-010 | Preserve evidence and continue to PO | Blocker | 1 | Auditor/AP, requester | Downstream |
| FR-011 | Support approved exceptions | Critical | 1 | Finance/procurement, approver | Policy |
| FR-012 | Enforce roles and sealed responses | Critical | 1 | All personas | Security |
| FR-013 | Detect award-to-PO/invoice variance | Critical | 1 | Auditor/AP | Downstream |
| FR-014 | Handle benchmark uncertainty and failure | Critical | 1 | Finance/procurement, approver | Benchmark |
| FR-015 | Measure operation and realized outcomes | Must have | 1 | Product, finance/procurement | Analytics |
| FR-016 | Ask a vendor to improve once | Must have | 1 | Finance/procurement, vendor respondent | Bidding |

## 3. Functional requirements

### FR-001 — Determine bidding and benchmark applicability

**Priority:** Blocker  
**Serves:** Finance/procurement, requester  
**Description:** Evaluate customer policy and purchase attributes to decide whether a competitive check is clear, recommended, required, or requires an exception. Independently determine whether a public-market benchmark is relevant and technically available.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A purchase request contains amount, category, entity, requester, contract status, and item information | The request reaches the policy evaluation point | The system records one bidding outcome and one benchmark-applicability outcome, including the governing rule identifiers and evaluation timestamp |
| A valid contract or preferred-item price satisfies the configured policy | The request is evaluated | The system permits the configured fast path and records why bidding and/or benchmarking was not required |
| Multiple policies match with different outcomes | The system evaluates the request | The authoritative policy precedence produces one result and the record retains all evaluated rules and the winning rule |
| Required inputs are missing | The system attempts evaluation | The result is “incomplete”; the system identifies the missing inputs and does not silently select a permissive outcome |
| The system determines the purchase is not benchmarkable | A user reviews the decision | The system shows the specific relevance reason, such as custom service, ambiguous item, unsupported geography, or insufficient identifier |
| A material request field changes after evaluation | The change is saved | The system re-evaluates policy and applicability, versions the prior result, and identifies any workflow state invalidated by the change |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Policy service unavailable | Critical | Do not default to “clear”; preserve the request, expose retry, and prevent a policy-required commitment until evaluation or an authorized contingency path completes |
| Currency or amount unresolved | Important | Mark amount-dependent rules incomplete and request the missing basis |
| Category changes after bids exist | Important | Re-evaluate policy and mark affected bids/benchmark as requiring review before award |

### FR-002 — Produce a comparable purchase specification

**Priority:** Blocker  
**Serves:** Requester, finance/procurement  
**Description:** Convert the purchase need into one common specification containing the attributes required for like-for-like vendor responses and benchmark matching.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A defined-goods request includes identifiers, quantity, destination, and required date | The buyer prepares the event | The system creates a specification with the same commercial requirements for every invited vendor |
| The request lacks an attribute required for comparison | The specification is evaluated | The system names the missing attribute and prevents publication while a Blocker attribute remains unresolved |
| AI proposes a specification from free text or attachments | A buyer reviews it | Proposed fields are distinguishable from user-confirmed fields and require confirmation before external send |
| A published specification changes | The buyer confirms the amendment | The system versions the specification, records the reason, sends the same amendment to affected vendors, and requires response reconfirmation where the change is material |
| The request contains multiple line items | Benchmark and bid preparation begins | Each line retains its own identifier, quantity, unit, and substitution rules while the event retains shared delivery and response terms |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Bundle cannot be decomposed into comparable items | Critical | Mark the event not benchmarkable and require buyer confirmation that vendor bids can still be evaluated as a bundle |
| Equivalent substitutions are allowed | Important | Record explicit substitution constraints and prevent an alternative from being labeled an exact match |
| Quantity changes after a vendor responds | Important | Mark affected responses stale until the vendor reconfirms or resubmits |

### FR-003 — Manage a private bid event

**Priority:** Blocker  
**Serves:** Finance/procurement  
**Description:** Create an invite-only competitive event linked to the purchase request, with two to five invited vendors, one common specification, one initial response deadline, and at most one controlled improvement request per vendor.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| The need is approved and specification complete | An authorized buyer creates the event | The event receives a stable identifier, request link, lifecycle state, owner, deadline, invited-vendor list, and audit entry |
| Fewer than the policy minimum or more than five vendors are selected | The buyer attempts publication | Authoritative validation rejects publication and identifies the violated rule |
| A prospective vendor is invited | The buyer confirms send | The system creates only the minimum quote-contact identity required by policy; it does not imply full vendor approval |
| The buyer publishes the event | Invitations are sent | Every vendor receives the same current specification and a vendor-scoped response path; delivery status is recorded per recipient |
| The event is already published and fewer than five vendors are participating | An authorized buyer adds a vendor | The new vendor receives the current specification and all applicable amendments; the event records when the vendor was added and applies the configured deadline or extension rule |
| The configured minimum number of valid bids has been received but one or more invited vendors remain pending | A buyer attempts to compare or close collection | The system applies the customer policy for readiness: wait for responses/declines, wait for the deadline, or require authorized early close with a recorded rationale |
| The deadline passes | The event has no approved extension | New submissions are rejected and the event moves to the configured closed or incomplete state |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Invitation delivery fails | Critical | Record the per-recipient failure, allow authorized resend or contact correction, and do not count the vendor as successfully invited for policy evidence |
| Vendor is removed after opening the request | Important | Preserve the access and activity history, revoke future access, and require a reason |
| Late-added vendor has materially less response time | Important | Apply the configured fairness rule and require deadline extension or an authorized exception before its invitation is sent |
| Event is cancelled | Important | Revoke response access, notify invited vendors as configured, and preserve all prior evidence |

### FR-004 — Collect secure vendor responses

**Priority:** Blocker  
**Serves:** Vendor respondent  
**Description:** Allow an invited vendor to submit structured commercial terms or an attached quote without completing full vendor onboarding.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A valid vendor-scoped invitation is opened | The respondent accesses the event | The system exposes only that vendor's request, response, messages, and permitted event information |
| The vendor enters structured data or uploads a supported quote | The response is saved | The system preserves draft data, attachment metadata, uploader identity, and timestamp without exposing it to competing vendors |
| Extraction produces proposed fields | The vendor or authorized buyer reviews them | Each proposed field can be confirmed or corrected and the response records who confirmed each material correction |
| Required response fields are incomplete | The vendor attempts final submission | Submission is rejected with field-specific validation while the draft remains saved |
| A submitted response is revised before the deadline | The vendor confirms revision | The system creates a new response version and preserves the prior version and timestamps |
| A quote arrives by email or another approved channel | An authorized buyer records it | The response is labeled buyer-entered, retains the entering actor, channel, capture time, and durable source evidence, and is evaluated against the same required fields as a portal response |
| A buyer-entered response includes a total but lacks the source attachment or other durable evidence required by policy | The buyer attempts to mark it valid | The server rejects valid status, preserves the draft, and identifies the missing evidence |
| The response link is expired, revoked, or belongs to another vendor | It is used | Access is rejected without disclosing event or vendor-confidential information |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Forwarded or stolen response link | Critical | Apply the approved verification policy, log the attempt, and prevent cross-vendor access |
| Buyer-entered values conflict with the attached source | Critical | Preserve both, prevent valid status, and require an authorized correction or vendor confirmation before comparison |
| Network fails during save or upload | Important | Preserve confirmed saved data, identify unsaved content, and allow retry without creating a duplicate submission |
| Two contacts edit concurrently | Important | Prevent silent overwrite and require the later editor to reconcile against the current version |
| Attachment is unsupported or unsafe | Important | Reject the file, preserve other draft data, and state the accepted formats or remediation |

### FR-005 — Normalize offers without changing source facts

**Priority:** Blocker  
**Serves:** Finance/procurement, vendor respondent  
**Description:** Extract and normalize comparable price and commercial terms while keeping original vendor-confirmed values immutable and separately visible.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A vendor submits a source value | The system normalizes it | The original value, source location, normalized value, formula, assumptions, confidence, actor, and timestamp remain linked |
| A value is missing | Normalization runs | The field remains missing; the system may request clarification but does not infer a commercial fact without labeling it as an assumption |
| Unit, quantity, or price basis differs across vendors | The comparison is calculated | The system converts only when the source basis and conversion are known and shows the calculation |
| A vendor or buyer corrects an extracted value | The correction is saved | The record preserves the prior extraction, correction, correcting actor, reason if required, and downstream recalculation |
| A low-confidence material field affects comparison | A user reviews the event | The field is visibly unresolved and cannot support an unqualified recommendation until confirmed or accepted under policy |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Arithmetic in the source document conflicts with its total | Critical | Show the conflict, preserve both values, and require confirmation of the authoritative amount |
| Tax, freight, or discount basis is ambiguous | Important | Keep the component unknown or conditional and reduce comparability confidence |
| Multiple currencies appear in one response | Important | Reject as unsupported in Phase 1 or require a single confirmed event-currency basis before comparison |

### FR-006 — Retrieve a relevant public-market benchmark

**Priority:** Blocker  
**Serves:** Finance/procurement, approver  
**Description:** Whenever an item passes benchmark relevance and matchability rules, retrieve current public-market evidence through an approved provider or capture authorized user-supplied evidence.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| An item is classified benchmarkable and an enabled provider covers its category and geography | The specification is confirmed or materially changed | The system creates a benchmark attempt automatically with the exact search inputs, provider, time, and status |
| The specification is confirmed and vendor invitations or response collection are in progress | Benchmark retrieval is eligible to run | Retrieval proceeds independently and may complete before or after vendor responses without changing the bid-policy minimum |
| An exact product identifier is present | Search begins | Exact-identifier matching is attempted before broader attribute or keyword matching |
| The provider returns candidate offers | Retrieval completes | The system stores only permitted evidence, associates candidates with the requested item, and sends them to relevance and normalization checks |
| A user supplies a marketplace URL or document under the approved fallback | The evidence is submitted | The system captures provenance, timestamp, submitting actor, and review status and does not imply provider verification beyond the stored evidence |
| The item is not benchmarkable | The event proceeds | No automated search occurs; the non-applicability reason is recorded and visible |
| The benchmark provider is queried repeatedly for unchanged inputs | A current result is still within the configured freshness window | The system may reuse the permitted cached result and records that it was reused, including original capture time |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Provider terms or credentials do not permit the requested operation | Critical | Do not attempt circumvention; return an unavailable state and record the dependency reason |
| Provider rate limit or outage | Important | Enter a retryable pending/unavailable state, apply bounded retry, and preserve the bid workflow |
| No candidate result | Important | Return “no reliable benchmark” with the completed search basis; do not create a synthetic price |
| Multiple marketplaces return duplicates | Nice-to-Have | Deduplicate when stable identifiers and seller/offer identity prove equivalence; otherwise retain separate evidence |

### FR-007 — Normalize benchmark evidence and confidence

**Priority:** Blocker  
**Serves:** Finance/procurement, approver  
**Description:** Determine whether marketplace evidence is comparable, indicative, or unreliable and calculate a landed-cost view without hiding unknown components.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| Candidate evidence contains product identity, condition, quantity basis, currency, and price | Relevance is evaluated | The system records match factors, mismatches, and a confidence classification using a versioned rule/model |
| Shipping, tax, duty, or fees are known | Landed cost is calculated | Each component and formula is visible and linked to its source or approved calculation basis |
| A landed-cost component is unknown | A benchmark is displayed | The component remains explicitly unknown, confidence is reduced, and the total is not represented as fully landed |
| Multiple comparable offers exist | A market range is calculated | The system identifies the included offers, calculation method, currency, capture time, and exclusions |
| Only one credible offer exists | The comparison is prepared | The result is labeled a single indicative reference, not a market range |
| An exact variant, condition, MOQ, geography, or delivery requirement differs materially | Comparability is evaluated | The candidate is excluded or clearly qualified and cannot produce a high-confidence comparable result |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Wrong variant appears cheaper | Critical | Exclude it from comparable benchmark calculations and show the mismatch if retained as evidence |
| Marketplace price is dynamic or expired | Important | Preserve the decision-time capture and mark current validity separately without rewriting history |
| Currency conversion is required | Important | Use an approved timestamped rate, show it, and reduce confidence if the award currency differs; Phase 1 event remains single-currency |
| Seller, warranty, or authenticity data is unavailable | Important | Mark the missing risk dimension and prevent an unqualified “cheaper equivalent” conclusion |

### FR-008 — Compare bids with benchmark evidence

**Priority:** Blocker  
**Serves:** Approver  
**Description:** Present vendor bids and benchmark evidence in one decision context while preserving their different authority and fulfillment status.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| Two or more valid bids exist | The decision context is opened | The system compares total price, unit basis, delivery, payment terms, warranty, vendor status, and unresolved assumptions on a common specification |
| A comparable or indicative benchmark exists | It is shown with bids | The benchmark is visibly labeled as market evidence, not a participating vendor, and does not count toward bid-policy minimums |
| A bid materially exceeds a comparable benchmark | Analysis is produced | The system may flag the difference and explain its basis but does not reject the bid or conclude misconduct automatically |
| The configured minimum bid count is satisfied but invited vendors remain pending | The buyer opens the decision context | The system shows that policy minimum and event readiness are distinct and prevents final comparison or early close unless the configured readiness rule is satisfied |
| Non-price terms change total-value interpretation | A recommendation is shown | The contributing facts and weights/logic are visible; the approver can select a different eligible vendor |
| Evidence is stale, incomplete, or low confidence | The comparison is viewed | The limitation is visible at the affected field and summary level and prevents an unqualified recommendation |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Cheapest benchmark cannot satisfy requested quantity or date | Critical | It cannot be represented as a fulfillable alternative and is qualified or excluded |
| Vendor bid expires during review | Important | Mark it expired and prevent award until policy permits reconfirmation or exception |
| All bids are above the benchmark | Important | Show the evidence and tradeoffs; do not infer collusion or automatically reopen bidding |

### FR-009 — Record a human-authorized award

**Priority:** Blocker  
**Serves:** Approver  
**Description:** Require an authorized person to select an eligible vendor, review source evidence and assumptions, and record the commercial rationale before downstream commitment.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| Valid bids and required controls are complete | An authorized approver selects a vendor | The server records the selected response version, actor, timestamp, rationale, benchmark context, and effective award version |
| The selected bid is not the lowest comparable bid | The approver confirms the award | A rationale is required and any configured additional approval is enforced authoritatively |
| The user has not acknowledged unresolved material assumptions | The user attempts award | The award is rejected and the unresolved assumptions are identified |
| An unauthorized user attempts award | The request reaches the server | The award is rejected, no state changes, and the attempt is logged |
| A completed award is replaced | An authorized change is approved | The prior award remains historical, a new effective version is created, and downstream impacts are identified |
| Award submission is repeated due to retry or double action | The server receives duplicate intent | At most one effective award version is created for the idempotency scope |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Evidence changes between opening and confirmation | Critical | Detect the stale decision context and require the approver to review the new version before award |
| Award succeeds but downstream handoff fails | Critical | Preserve the effective award, mark handoff failed, and retry without duplicating the award |
| Selected vendor becomes ineligible | Important | Prevent new commitment and route to replacement or approved exception while preserving history |

### FR-010 — Preserve evidence and continue to PO

**Priority:** Blocker  
**Serves:** Auditor/AP, requester  
**Description:** Keep the request, event, specification, invitations, responses, benchmark evidence, calculations, exceptions, and award connected and carry approved commercial data into PO preparation.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| An award becomes effective | PO preparation begins | The approved vendor, item, quantity, unit, price, terms, delivery, and source links transfer without manual re-entry |
| A material action or data correction occurs | It is committed | The audit record captures actor/service, timestamp, action, object version, before/after state or immutable event payload, and correlation to the request |
| Benchmark evidence is used in the decision | The decision is recorded | A permitted decision-time snapshot or durable source reference preserves what the approver evaluated and its limitations |
| A user with audit permission reviews the purchase | The record is retrieved | The user can reconstruct policy evaluation, event lifecycle, responses, normalization, benchmark, exception, award, and handoff state |
| PO handoff fails | The failure is recorded | The award remains effective, the handoff is retryable, and duplicate PO creation is prevented |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Marketplace listing disappears | Critical | Historical evidence remains interpretable within permitted retention terms and states that the live source is unavailable |
| Audit retention conflicts with provider terms | Critical | Follow the approved legal design and expose the resulting evidence limitation; do not retain prohibited content |
| PO fields are edited after handoff | Important | Record the change and invoke variance or reapproval rules based on configured materiality |

### FR-011 — Support approved exceptions

**Priority:** Critical  
**Serves:** Finance/procurement, approver  
**Description:** Permit policy-governed continuation when required bidding or benchmarking cannot or should not be completed.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A permitted exception reason applies | An authorized user requests an exception | The system requires reason, explanation, evidence where configured, and the required approval route |
| An exception is approved | The purchase proceeds | The audit trail records which requirement was waived, by whom, when, why, and under which policy |
| An exception reason is not allowed for the category or amount | Submission is attempted | The server rejects it and identifies that the selected reason is not permitted |
| Benchmark retrieval failed after bounded retry | Policy allows failure continuation | The exception record includes provider status and attempted fallback; the system does not characterize the item as market-validated |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Exception approver is also requester where segregation is required | Critical | Enforce the configured separation-of-duties rule authoritatively |
| Emergency exception used repeatedly | Important | Record analytics by customer/category/requester and make the pattern reviewable; do not auto-accuse or auto-block beyond policy |

### FR-012 — Enforce roles and sealed responses

**Priority:** Critical  
**Serves:** All personas  
**Description:** Enforce least-privilege access to event configuration, vendor responses, benchmark evidence, recommendations, exceptions, award, and audit records.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A vendor accesses an event | Authorization is evaluated | Access is limited to the vendor's own current request, messages, and response; competing identities and data are never returned |
| A requester lacks commercial-response permission | The requester views progress | The system shows permitted status without bid values, benchmark-sensitive data, or restricted rationale |
| A buyer, approver, or auditor performs an action | The request reaches the server | Role, entity, event, state, and separation-of-duties rules are enforced authoritatively |
| Access is revoked | A previously valid session or token is used | Further protected access is rejected according to the revocation requirement and logged |
| Restricted evidence is exported or downloaded | The action succeeds | The audit trail records actor, object, time, and permitted purpose/context available to the system |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Vendor guesses another response identifier | Critical | Return no confidential data and log the denied attempt without confirming the target exists |
| User changes entity or role during an open session | Critical | Re-evaluate authorization before each protected operation; stale client state grants no authority |
| Award requires two-person control | Important | The same actor cannot satisfy both approvals when policy requires separation |

### FR-013 — Detect award-to-PO/invoice variance

**Priority:** Critical  
**Serves:** Auditor/AP  
**Description:** Compare the effective award with resulting PO and invoice economics using a documented basis and configurable materiality.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A PO is created or modified from an award | Comparison runs | The system compares item identity, quantity, unit price, total, currency, shipping, tax treatment, and terms where available and records the result |
| An invoice is linked to the awarded purchase | Invoice data becomes available | The system compares the invoice with the award and PO and identifies the exact fields and amounts causing material variance |
| An award is effective but no qualifying PO/invoice/receipt evidence exists | Value is reported | The system may report identified or awarded improvement but shows realized value as pending or unavailable, never confirmed |
| Variance is below configured materiality | The comparison completes | The result is recorded without blocking unless customer policy specifies otherwise |
| Variance exceeds materiality | The comparison completes | The system triggers the configured review or approval path and does not silently label savings as realized |
| Multiple invoices or PO changes apply to one award | Outcome is calculated | The system aggregates or allocates values using a documented method and exposes the basis |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Substitution occurs after award | Critical | Mark identity variance and require the configured approval before treating economics as compliant |
| Freight or tax appears only at invoice | Important | Attribute the variance to that component rather than unit price when data permits |
| Partial delivery and invoice | Important | Measure realized value only for received/invoiced quantity and retain outstanding balance |

### FR-014 — Handle benchmark uncertainty and failure

**Priority:** Critical  
**Serves:** Finance/procurement, approver  
**Description:** Represent pending, unavailable, not applicable, no-result, stale, and low-confidence benchmark states explicitly without blocking unrelated bid work or creating false certainty.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A benchmark attempt is in progress | A user reviews the event | The system shows pending status, attempt time, source, and whether further action is required; bids remain usable |
| Search completes without a credible match | Results are finalized | The system records “no reliable benchmark,” search basis, rejection reasons, and completion time without showing a synthetic price |
| A provider error occurs | Bounded retry is exhausted | The system shows unavailable status, failure category, permitted retry/fallback, and audit entry without exposing secrets |
| Evidence passes freshness limit before award | The decision context is opened | The evidence is marked stale and refreshed or explicitly accepted under policy before it supports a recommendation |
| Match confidence is below the comparable threshold | Evidence is displayed | It is labeled indicative or excluded, with the confidence factors visible |
| Benchmark is required by policy but unavailable | Award is attempted | The server follows the configured rule: block pending retry, accept authorized evidence fallback, or require an approved exception; it never silently passes |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Provider returns malformed or contradictory data | Critical | Reject affected candidates, record validation failure, and do not display them as comparable |
| Some line items have benchmarks and others do not | Important | Show coverage per line and do not imply the full event was benchmarked |
| Refresh returns a materially different price | Important | Preserve both versions, identify the change, and require review before award if it affects the decision |

### FR-015 — Measure operation and realized outcomes

**Priority:** Must have  
**Serves:** Product, finance/procurement  
**Description:** Capture the funnel, quality, cycle-time, benchmark, exception, award, and downstream outcome data required to evaluate the pilot and customer value.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A material lifecycle event occurs | It is committed | A versioned analytics event records non-sensitive identifiers, tenant, timestamp, actor type, state transition, correlation ID, and relevant reason codes |
| Savings or improvement is calculated | A metric is stored or shown | The baseline type, baseline amount, awarded amount, realized amount, currency, quantity basis, and calculation version are retained |
| A benchmark attempt completes | Analytics records it | Category, provider, applicability, result state, confidence band, latency, and failure/rejection reasons are captured without unauthorized listing or vendor PII |
| A pilot report is generated | The cohort and window are selected | Funnel denominators, exclusions, late-arriving outcomes, and category/customer segmentation are documented |
| Pilot gates are evaluated | A matured cohort is reported | The scorecard includes eligible-request completion, valid responses per invited vendor, median active buyer time per completed check, median vendor-invitation-to-recorded-decision cycle, and verified invoice savings per $1 million of eligible spend checked against a locked pre-check counterfactual |
| Consent or privacy rules prohibit an analytics field | Event generation occurs | The field is omitted or transformed according to policy and the metric contract documents the limitation |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Benchmark data cannot be retained for analytics | Critical | Store only the permitted classification and aggregate metadata; exclude prohibited source content |
| Outcome arrives after reporting window | Important | Attribute it using the documented cohort and late-arrival policy; do not silently drop or double count it |
| One event has multiple award versions | Important | Metrics use the effective award and retain revision counts separately |

### FR-016 — Ask a vendor to improve once

**Priority:** Must have  
**Serves:** Finance/procurement, vendor respondent  
**Description:** Allow an authorized buyer to use “Ask to improve” once for an invited vendor without expanding Phase 1 into open-ended negotiation. Billy may draft the request, but a person reviews and authorizes the external message. Any revised offer becomes a new vendor-response version.

#### Acceptance criteria

| Given | When | Then |
|---|---|---|
| A current valid vendor response exists and no improvement request has been sent to that vendor | An authorized buyer prepares an improvement request | The system may draft a message using current response and approved market context, but no external action occurs until the buyer reviews and confirms it |
| An authorized buyer reviews comparison context | Cross-vendor gaps and ranks are displayed | Those values remain restricted to authorized customer users and are not copied into the vendor-facing draft |
| Customer disclosure policy prohibits competing-price disclosure | A draft is generated | The message excludes competing vendor identities, exact bid values, and any difference or ranking detail that makes another vendor's bid derivable |
| Comparable public-market evidence is current and policy permits disclosure | The buyer prepares the message | The message may state the approved public-market price or range and the difference between that evidence and the recipient’s own current quote; it identifies the evidence as public-market context, not a participating bid |
| The benchmark is indicative, has no reliable result, or is unavailable | The buyer prepares the message | The system excludes quantitative public-market claims and may produce only a non-quantitative request to improve |
| The buyer edits the Billy draft | The buyer attempts to send it | The final edited content is revalidated against disclosure policy; prohibited content blocks send without consuming the one-request allowance |
| The buyer confirms a valid request | The message is sent | The system records sender, recipient, exact final message version, disclosed evidence basis, recipient response version, deadline, specification version, and send result |
| The vendor submits an improved offer by the deadline | The response is validated | A new response version is created; the prior version remains historical and every comparison/award uses the selected current version |
| One improvement request has already been sent to the vendor | Another request is attempted in Phase 1 | Authoritative validation rejects it and identifies the one-request limit |
| A revised offer arrives through an approved non-portal channel | An authorized buyer records it | FR-004 buyer-entered response controls apply, and the response links to the improvement request it satisfies |

#### Edge cases

| Case | Severity | Expected behavior |
|---|---|---|
| Draft or buyer edit reveals or makes a sealed competing bid derivable | Critical | Prevent send and identify the prohibited disclosure before any vendor receives it; do not count the blocked attempt as the vendor’s one request |
| A quantitative market claim is based on indicative, no-result, unavailable, expired, or superseded evidence | Critical | Prevent send until the claim is removed or current comparable evidence is selected and retained as its basis |
| Revised offer changes the specification or introduces a substitution | Critical | Do not treat it as directly comparable; require specification/substitution review and apply amendment rules |
| Improvement deadline passes without response | Important | Retain the original valid response as current unless policy says otherwise and mark the improvement request expired |
| Award is attempted while a promised revised offer is pending | Important | Apply the event-readiness policy and show the pending revision before confirmation |
| Provider evidence becomes stale after the request is sent | Important | Preserve what was disclosed and require current evidence for any later recommendation or award conclusion |

## 4. Non-functional requirements

All thresholds are proposed for validation against Stampli production standards.

| NFR | Category | Requirement | Proposed threshold / observable contract |
|---|---|---|---|
| NFR-001 | Evidence integrity | Every displayed benchmark fact is source-traceable | 100% of displayed price and fulfillment fields link to a permitted source or explicit calculation |
| NFR-002 | Performance | Policy and applicability evaluation is responsive | p95 ≤2 seconds excluding external benchmark retrieval |
| NFR-003 | Reliability | Benchmark failure does not take down bidding | Core event, response, comparison, and award functions remain usable during provider failure |
| NFR-004 | Financial clarity | Cost assumptions are explicit | 100% of derived totals identify currency, unit, quantity, and inclusion/unknown state for freight, tax, duty, and fees |
| NFR-005 | Human authority | No autonomous commitment | 100% of awards, exceptions, vendor sends, and financial commitments require the configured authorized action |
| NFR-006 | Explainability | Recommendations expose their evidence | 100% identify contributing source facts, derived values, assumptions, and confidence |
| NFR-007 | Semantic integrity | Benchmark and bids cannot be confused | No benchmark counts as a valid invited-vendor response or award candidate |
| NFR-008 | Consistency | Award creation is idempotent and durable | Duplicate submission creates at most one effective award; committed state survives refresh |
| NFR-009 | Usability | Policy outcome is actionable | Every non-clear outcome states governing reason and next required action |
| NFR-010 | Workflow continuity | Award-to-PO requires no re-entry | All confirmed commercial fields supported by the PO domain transfer from the award record |
| NFR-011 | Validation | Missing information is specific | Validation names every field blocking publication, submission, benchmark comparison, or award |
| NFR-012 | Confidentiality | Restricted commercial data is not exposed to requesters | Server responses omit fields the requester role cannot access |
| NFR-013 | Vendor security | Response access is narrowly scoped | Tokens/sessions are event-and-vendor scoped, revocable, expiring, and protected by the approved verification policy |
| NFR-014 | Sealed bidding | Cross-vendor disclosure is prevented | Zero API or UI paths return another vendor's identity or response to a vendor principal |
| NFR-015 | Draft reliability | Vendor work is recoverable | Confirmed saves survive refresh; failed uploads can retry without duplicating submission |
| NFR-016 | Accessibility/localization | External response is broadly usable | WCAG 2.2 AA; locale-aware dates, numbers, currencies; keyboard and assistive-technology support |
| NFR-017 | Auditability | Material actions are reconstructable | 100% record actor/service, timestamp, correlation, object/version, action, and change evidence |
| NFR-018 | Historical integrity | Decision-time benchmark remains interpretable | Preserve a permitted snapshot/reference and its timestamp, assumptions, and confidence for every benchmark used at award |
| NFR-019 | Calculation integrity | Variance and savings use explicit bases | 100% expose baseline type, formula version, amounts, currency, and quantity basis |
| NFR-020 | Data lifecycle | Evidence follows policy | Retention, legal hold, export, deletion, and access are validated before beta |
| NFR-021 | Reliability | Core workflow availability is isolated from providers | External connector uses timeout, bounded retry, circuit breaking, and a provider kill switch |
| NFR-022 | Performance | Saved comparison is usable promptly | p95 ≤3 seconds for saved event data, excluding in-progress external retrieval |
| NFR-023 | Security | Data is protected and actions authorized | Encryption in transit/at rest, secret management, server-enforced RBAC, access logging, and security review before beta |
| NFR-024 | AI integrity | AI never silently alters source facts | Zero source facts overwritten by extraction/normalization; all corrections are actor-attributed and versioned |
| NFR-025 | Marketplace compliance | Access honors provider terms | 100% of connectors use approved API/feed/partner/customer-authorized evidence; automated scraping excluded |
| NFR-026 | Supportability | Operations are observable | Metrics/logs/traces cover provider latency/error, invitation delivery, submission, normalization, award, handoff, and variance |
| NFR-027 | Privacy | PII and commercial data are minimized | Data inventory, purpose, access, retention, subprocessor, and deletion behavior approved before beta |
| NFR-028 | Release safety | Rollout and rollback are controlled | Tenant feature flag, category allowlist, provider kill switch, staged cohorts, and rollback without loss of committed evidence |
| NFR-029 | Competitive confidentiality | “Ask to improve” messages cannot expose sealed bids | Zero messages reveal a competing vendor's identity, exact bid, or a gap/rank that makes the bid derivable. A permitted delta may compare approved comparable public-market evidence only with the recipient’s own response |
| NFR-030 | Response-channel parity | Buyer-entered responses meet the same evidence contract | 100% of valid buyer-entered responses retain durable source evidence, actor, channel, timestamp, completeness result, and version history |

## 5. Requirement-to-test minimum

Before controlled beta, QA must cover:

- happy paths for required and recommended checks;
- policy precedence and re-evaluation after material changes;
- vendor isolation and attempted cross-vendor access;
- buyer-entered draft, missing-evidence, source-conflict, correction, and valid-response paths;
- exact, indicative, no-result, not-applicable, pending, stale, and unavailable benchmark states;
- wrong-variant, unknown-shipping, MOQ, currency, condition, and delivery mismatches;
- AI extraction correction and immutable source behavior;
- authorized and unauthorized award/exception paths;
- event readiness after policy minimum, all-response, deadline, and authorized early-close conditions;
- “Ask to improve” human review, final-message revalidation, buyer-only cross-vendor context, benchmark-state disclosure rules, one-request limit, revised response, expiry, and non-portal revision paths;
- concurrent updates and duplicate submissions;
- award handoff failure and idempotent retry;
- PO/invoice variance across partial and revised transactions; and
- realized-value pending behavior before qualifying downstream evidence; and
- analytics denominator and late-arriving outcome correctness.
