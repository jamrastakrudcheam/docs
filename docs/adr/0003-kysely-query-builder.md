# Kysely query builder instead of an ORM

Data access uses Kysely over node-postgres rather than TypeORM or Prisma. ADR-0001 needs one transaction object handed explicitly through several module services, and Kysely's `Transaction` is just that object; money and quantities must stay decimal strings, which node-postgres already returns for `numeric`; and the design relies on row locks in key order, unique source allocations and per-module schemas, all of which are plain SQL that Kysely writes without hiding it.

Kysely is pinned to 0.28.x because 0.29 is ESM-only and the NestJS build is CommonJS.
