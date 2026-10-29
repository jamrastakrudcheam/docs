---
status: accepted
---

# Foreign-currency journals with THB accounted amounts

Ledgers keep THB as their currency, but journals and subledger documents may be entered in any enabled currency (P27). Each line stores the entered amount in its currency and the accounted THB amount, computed with the rate type and date on the journal (or a user rate when the rate type is "User"). Balances keep accounted THB per account and period, plus entered amounts per currency for accounts that are revalued.

## Considered options
- **THB only** (D06 as first written): simpler, but ICC buys and sells in USD, EUR, JPY, SGD and MMK and revalues open balances every month (GL-203.3).

## Consequences
- Rates are master data per rate type and date, loaded from a spreadsheet or a simulated bank feed (D03); a missing rate stops the journal from completing.
- A journal balances in accounted THB; rounding differences go to a configured rounding account per ledger.
- Month-end revaluation creates an unrealized gain/loss journal that reverses on the first day of the next period.
- Tax amounts and tax reports stay in THB.
