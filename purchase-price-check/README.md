# Purchase Price Check — PRD package

This package defines a greenfield Stampli feature for lightweight purchase bidding with a relevant public-market price benchmark. It is intended as an assessment submission and as a starting contract for Product, Design, Engineering, Data, Security, Legal, and QA.

## Product thesis

Purchase Price Check turns a defined purchase need into a private competitive-bidding workflow inside Stampli. It collects comparable vendor offers and, when the purchase is benchmarkable, adds a source-visible public-market price benchmark before a human authorizes the award. The request, bids, benchmark, decision, PO, and invoice remain connected.

The benchmark is evidence, not a bid. The system may validly conclude that no reliable benchmark exists.

## Documents

| Document | Purpose | Primary audience |
|---|---|---|
| [PRD.md](./PRD.md) | Product problem, scope, workflow, priorities, business rules, launch plan, and success criteria | Product, Design, leadership |
| [FUNCTIONAL-REQUIREMENTS.md](./FUNCTIONAL-REQUIREMENTS.md) | Testable Phase 1 functional and non-functional requirements | Engineering, QA, Design |
| [TECHNICAL-FOUNDATION.md](./TECHNICAL-FOUNDATION.md) | Proposed domain model, service boundaries, lifecycle, integration contract, security, and observability | Engineering, Architecture, Security |
| [ANALYTICS-AND-PILOT.md](./ANALYTICS-AND-PILOT.md) | Event taxonomy, metric definitions, pilot design, and go/iterate/stop gates | Product, Data, Finance |
| [ASSUMPTIONS-RISKS-DECISIONS.md](./ASSUMPTIONS-RISKS-DECISIONS.md) | Evidence register, risks, dependencies, unresolved decisions, and discovery questions | Cross-functional reviewers |

## Scope boundary

Phase 1 is an invite-only bid for a defined purchase, normally involving two to five vendors. It includes policy triggering, comparable specifications, secure or buyer-entered vendor responses, quote normalization, conditional public-market benchmarking, one optional buyer-authorized “Ask to improve” request per vendor, human award, exceptions, PO handoff, audit evidence, and PO/invoice variance detection.

It is not a strategic-sourcing suite, supplier marketplace, reverse auction, contract-management product, or autonomous purchasing agent.

## Pilot recommendation

Use a capped 12-week build-to-production-pilot for two or three standardized categories, one validated benchmark provider, tenant and category controls, and human-controlled awards. This is a planning hypothesis contingent on early production discovery, not approval for category expansion or an unconditional engineering estimate.

The operating flow has five stages: define and check policy; invite two to five vendors; collect offers while checking the market; compare and optionally ask a vendor to improve; and award and hand off. Verified invoice savings are evaluated against a locked pre-check counterfactual, with a hypothesis of $20,000–$50,000 per $1 million of eligible spend checked.

## Evidence boundary

The production Stampli codebase and architecture were not available for this assessment. The documents therefore:

- make no claim about undisclosed Stampli features or roadmap;
- describe architecture and ownership as proposed, not verified;
- avoid invented endpoint names or production schemas;
- require implementation discovery before estimates or contracts are finalized; and
- treat all example customer, vendor, provider, transaction, price, availability, identity, and timing data as illustrative rather than live evidence.

## Recommended reading order

1. Read the PRD for the product decision and Phase 1 boundary.
2. Review the requirements contract and open decisions together.
3. Validate the technical foundation against Stampli's production architecture.
4. Agree on the pilot cohort and metric definitions before development begins.
