---
status: accepted
---

# Configurable chart of accounts, populated from Oracle EBS

The chart of accounts is not hard-coded: each COA defines its segments, value sets, delimiter and segment labels the way Oracle Fusion does, with **Company (primary balancing)** and **Natural account** labels mandatory. The actual structure and values are imported from the Oracle E-Business Suite R12 system ICC uses today (its *Chart of Accounts – Segment Values Listing*), because ICC ERP replaces Oracle in phases (D01) and balances, reports and mappings must carry over without re-coding accounts (P08: no invented master data).

## Considered options
- **Fixed segments chosen now** (e.g. Company-CostCenter-Account-SubAccount): simpler tables and screens, but guesses a structure we have not seen and forces a re-map of every EBS account.
- **One free-text account code:** easy to import, but loses balancing and natural-account semantics needed for TB, close and financial statement mapping.

## Consequences
- Journal lines store the account combination id; imports accept the full combination string using the COA's delimiter (the journal template's ACCOUNT_COMBINATION column).
- Cross-validation rules, account hierarchies and statement mapping work on segment labels, not on segment positions.
