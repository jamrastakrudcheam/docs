---
status: accepted
---

# Shipped-not-billed account when billing waits for POD

For Customer Sites whose billing trigger is proof of delivery (P03), Ship Confirm posts Dr *Shipped not billed* / Cr Inventory and the cost moves to COGS when the invoice recognises revenue; sites billed at Ship Confirm post COGS directly. The SLA mapping chooses the path from the site's billing trigger. This keeps cost of sales in the same period as its revenue instead of booking COGS in the month of shipment and revenue in the month the POD arrives.

## Consequences
- Shipped-not-billed is an asset balance that must reconcile to shipments awaiting invoice at every close (ticket 33 precheck).
- Returns and cancelled shipments before invoicing reverse from Shipped-not-billed, not from COGS.
