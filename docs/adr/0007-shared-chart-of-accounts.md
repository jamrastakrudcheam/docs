---
status: accepted
---

# One chart of accounts shared by ICC and ITK

ICC and ITK post to one chart of accounts, as they do in Oracle EBS today (P26). Segment values, hierarchies and cross-validation rules belong to the chart, so a new natural account is opened once with one running code for both companies. The Company segment (primary balancing) tells the companies apart: each account combination belongs to the company whose value is in that segment, and balances, periods and statements stay per ledger (ADR-0002).

## Considered options
- **One chart per company** (the first cut of ticket 06): the same values would have to be imported and kept identical twice, and the shared running code in the account request process (GL-101) breaks.

## Consequences
- The chart is owned by a company group, not a company. Each company is assigned to exactly one chart; its Company segment value is recorded on the assignment.
- A combination is valid for a ledger only when its Company segment value matches the ledger's company, so one company can never post into the other's books.
- Changing a shared value (description, enabled, posting flags) affects both companies, so it goes through the account request approval.
