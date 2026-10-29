---
status: accepted
---

# Background jobs in PostgreSQL for file imports

File imports follow the Oracle FBDI flow (load file → interface rows → import → correct/purge) and can be thousands of rows, so they run as jobs instead of inside the HTTP request: a job table in PostgreSQL, picked up by a worker inside the same NestJS process (`FOR UPDATE SKIP LOCKED`), with status, counts and log visible on an "งานนำเข้า" page. This does not reopen ADR-0001: there is still no message broker or cross-module event outbox, each job step runs in its own transaction, and creating a document (journal, invoice, statement) is still one transaction through the owning module's service.

## Considered options
- **Synchronous import in the request:** simplest, but large files time out and a browser refresh loses the result.
- **External queue (Redis/BullMQ, a broker):** rejected by ADR-0001 and adds infrastructure for a local-first system.

## Consequences
- Jobs must be idempotent per step (re-running LOAD or IMPORT on the same load request does not duplicate rows or documents).
- A crashed worker leaves jobs "Running"; a heartbeat timeout returns them to "Queued".
