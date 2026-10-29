# Modular monolith with one synchronous transaction per final command

The Blueprint proposed asynchronous cross-module integration through an outbox. We run one NestJS deployment on one PostgreSQL database instead, and a final command (Ship Confirm, accept receipt, post invoice, finalize payment or count) passes a single transaction through every module service it touches, including SLA/GL, so stock, balances and accounting commit or roll back together. There is no outbox and no message broker; idempotency keys and optimistic versions still guard against double-submits and stale edits (D04, user correction).

## Consequences

- Modules call each other through in-process service interfaces, never HTTP to themselves, and never write another module's schema directly.
- If an SLA mapping is missing, the final command rolls back; the user saves a draft and fixes the exception first.
- File uploads and report generation sit outside DB atomicity and use temporary file → validated metadata → finalize.
