# Roadmap: Multi-Tenant FinTech Escrow, Double-Entry Ledger & IAM Engine

## Overview

This roadmap delivers a production-grade B2B treasury, escrow, and digital wallet backend built with Python 3.11+, FastAPI (async), PostgreSQL 16 (SQLAlchemy 2.0 + asyncpg), and Redis 7. Progression follows six locked phases: starting from containerized async infrastructure and configuration validation, moving into multi-tenant workspace IAM with RBAC and Redis-backed session revocation, building the append-only double-entry ledger and deadlock-free ACID transfer engine, layering dual-store idempotency middleware and Lua-scripted sliding-window rate limiting, verifying mathematical invariants through an automated reconciliation worker and 50-way concurrency stress suites, and concluding with OpenAPI documentation polish, Locust load benchmarking, and architectural documentation.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Architecture, Folder Structure & Docker Infrastructure** - Containerized FastAPI, PostgreSQL 16, and Redis 7 runtime with strict environment validation and `/health` probes
- [ ] **Phase 2: Multi-Tenant IAM, RBAC & Session Revocation** - Workspace-scoped organizations, Argon2id/PyJWT auth, Redis `jti` blocklist, SHA-256 API keys, 4-role RBAC, and immutable audit logs
- [ ] **Phase 3: Append-Only Double-Entry Ledger & ACID Transfer Engine** - Multi-currency `BIGINT` ledger models, immutable `DEBIT`/`CREDIT` entries, deterministic `wallet_id` ASC row locking, and omnibus `/deposits` & `/payouts`
- [ ] **Phase 4: Idempotency Middleware & Sliding-Window Redis Rate Limiter** - Mandatory `Idempotency-Key` enforcement (Redis lock + PostgreSQL replay) and Redis Lua sliding-window rate limiting
- [ ] **Phase 5: Automated Reconciliation Worker & Concurrency Stress Tests** - Zero-sum ledger reconciliation service (`SUM(debits) == SUM(credits)`) and 50-way async Pytest concurrency & idempotency race suites
- [ ] **Phase 6: Swagger Documentation Polish, Locust Benchmark & Architecture README** - Enriched OpenAPI schemas, Locust high-throughput load testing suite, and comprehensive architecture README

## Phase Details

### Phase 1: Architecture, Folder Structure & Docker Infrastructure
**Goal**: Developers and operators can boot a complete, containerized async FastAPI + PostgreSQL 16 + Redis 7 stack with validated settings and verified health probes.
**Depends on**: Nothing (first phase)
**Requirements**: INFRA-01, INFRA-02, INFRA-03, INFRA-04, INFRA-05
**Success Criteria** (what must be TRUE):
  1. Repository exposes the modular package hierarchy (`app/api`, `app/core`, `app/models`, `app/schemas`, `app/services`, `tests/`) with pinned Python 3.11+ dependencies in `requirements.txt`.
  2. Application startup validates all required environment variables (PostgreSQL 16 DSN, Redis 7 URL, JWT secrets, rate-limit settings) via `pydantic-settings` and fails fast on invalid configuration.
  3. SQLAlchemy 2.0 `asyncpg` engine and session factory operate with `READ COMMITTED` default isolation alongside an initialized async Redis 7 client.
  4. Running `docker compose up --build` starts FastAPI (running as a non-root container user), PostgreSQL 16, and Redis 7 (`noeviction` policy), and `GET /health` returns HTTP `200 OK` confirming live database and Redis connectivity.
**Plans**: TBD

### Phase 2: Multi-Tenant IAM, RBAC & Session Revocation
**Goal**: Organizations and users can authenticate via JWTs or workspace API keys with strict tenant isolation, four-role RBAC enforcement, immediate token revocation, and immutable audit logging.
**Depends on**: Phase 1
**Requirements**: IAM-01, IAM-02, IAM-03, IAM-04, IAM-05, IAM-06
**Success Criteria** (what must be TRUE):
  1. Alembic migrations cleanly upgrade the database to create `organizations`, `users`, `memberships`, and `audit_logs` tables with proper foreign keys and indexes.
  2. Users can register and authenticate with `Argon2id` password hashing to receive `PyJWT` access and refresh tokens carrying unique `jti` claims, and rotating or logging out a token immediately blocks its `jti` in Redis (`noeviction`).
  3. Organizations can create and authenticate with SHA-256 hashed API keys that resolve strictly to their workspace context.
  4. Every tenant-scoped request enforces `organization_id` isolation and role permissions (`OWNER`, `FINANCE_ADMIN`, `AUDITOR` read-only, `SUPPORT`), rejecting unauthorized or cross-tenant access with HTTP `403`/`404`.
  5. Sensitive IAM and administrative actions (role changes, API key creation/revocation, auth mutations) write append-only records to `audit_logs`.
**Plans**: TBD

### Phase 3: Append-Only Double-Entry Ledger & ACID Transfer Engine
**Goal**: Tenants can execute multi-currency wallet transfers, deposits, and payouts backed by an immutable double-entry ledger, atomic cached balance updates, and deadlock-free row locking.
**Depends on**: Phase 2
**Requirements**: LEDGER-01, LEDGER-02, LEDGER-03, LEDGER-04, LEDGER-05
**Success Criteria** (what must be TRUE):
  1. Wallets, transactions, and ledger entries store all monetary amounts as `BIGINT` minor units scoped by ISO 4217 `currency` (`NPR`, `INR`, `KRW`, `USD`, etc.) with zero floating-point columns.
  2. `ledger_entries` enforces `CHECK (amount > 0)` at the database level and prevents `UPDATE` or `DELETE` mutations on historical ledger rows.
  3. Executing a transfer atomically inserts balanced `DEBIT` and `CREDIT` ledger entries summing to zero and updates `wallets.cached_balance` within a single `READ COMMITTED` database transaction.
  4. Concurrent transfers between overlapping wallet pairs acquire `SELECT ... FOR UPDATE` locks ordered deterministically by `wallet_id` ASC, preventing AB-BA deadlocks and negative balances.
  5. `/deposits` and `/payouts` endpoints book balanced double-entry rows against per-currency system omnibus contra accounts and support escrow hold/release lifecycle states.
**Plans**: TBD

### Phase 4: Idempotency Middleware & Sliding-Window Redis Rate Limiter
**Goal**: Financial mutation endpoints are protected against duplicate execution via atomic Redis-plus-PostgreSQL idempotency and guarded against traffic abuse via sliding-window rate limiting.
**Depends on**: Phase 3
**Requirements**: IDEM-01, IDEM-02, RATE-01
**Success Criteria** (what must be TRUE):
  1. Requests to `/transfers`, `/deposits`, and `/payouts` without an `Idempotency-Key` header are rejected immediately with HTTP `400`/`422`.
  2. In-flight requests acquire an atomic Redis `SET NX EX` lock per tenant and idempotency key (returning HTTP `409 Conflict` to concurrent duplicates), while completed requests persist status codes and payloads in PostgreSQL `idempotency_records` for exact deterministic replay.
  3. High-frequency callers exceeding configured request quotas are throttled by the Redis 7 Lua-script sliding-window rate limiter with HTTP `429 Too Many Requests` and accurate `Retry-After` headers.
**Plans**: TBD

### Phase 5: Automated Reconciliation Worker & Concurrency Stress Tests
**Goal**: Mathematical ledger integrity and concurrency safety are continuously verifiable via an automated reconciliation service and proven under 50-way simultaneous stress tests.
**Depends on**: Phase 4
**Requirements**: RECON-01, TEST-01, TEST-02
**Success Criteria** (what must be TRUE):
  1. Running the reconciliation worker/service verifies that `SUM(debits) == SUM(credits)` holds globally and per currency, and confirms `wallets.cached_balance == net(ledger_entries)` for every wallet, flagging any discrepancy immediately.
  2. The async Pytest concurrency suite executes 50 simultaneous transfers across overlapping wallets (`A -> B`, `B -> A`, `A -> C`) with zero database deadlocks, zero race conditions, and exact ending balances.
  3. The async Pytest idempotency race suite fires concurrent identical `Idempotency-Key` requests and proves that exactly one financial transaction is committed while duplicates receive `409 Conflict` or the cached replay response.
**Plans**: TBD

### Phase 6: Swagger Documentation Polish, Locust Benchmark & Architecture README
**Goal**: API consumers and reviewers have interactive OpenAPI documentation, reproducible Locust throughput benchmarks, and a complete engineering architecture guide.
**Depends on**: Phase 5
**Requirements**: DOCS-01, BENCH-01, DOCS-02
**Success Criteria** (what must be TRUE):
  1. FastAPI `/docs` (Swagger UI) and `/openapi.json` document all endpoints with rich request/response examples, explicit error schemas, JWT/API-Key security schemes, and required `Idempotency-Key` header parameters.
  2. `locustfile.py` executes multi-user load scenarios across `/transfers`, `/deposits`, and `/payouts`, producing latency and throughput metrics without ledger invariant violations.
  3. `README.md` clearly documents the system architecture, the 5 core engineering invariants, database ERD and transaction locking flows, Docker quickstart commands, and benchmark results.
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Architecture, Folder Structure & Docker Infrastructure | 0/TBD | Not started | - |
| 2. Multi-Tenant IAM, RBAC & Session Revocation | 0/TBD | Not started | - |
| 3. Append-Only Double-Entry Ledger & ACID Transfer Engine | 0/TBD | Not started | - |
| 4. Idempotency Middleware & Sliding-Window Redis Rate Limiter | 0/TBD | Not started | - |
| 5. Automated Reconciliation Worker & Concurrency Stress Tests | 0/TBD | Not started | - |
| 6. Swagger Documentation Polish, Locust Benchmark & Architecture README | 0/TBD | Not started | - |
