# Requirements: Multi-Tenant FinTech Escrow, Double-Entry Ledger & IAM Engine

**Defined:** 2026-10-28
**Core Value:** Mathematically provable financial integrity under high concurrency: every monetary movement is recorded as balanced, immutable double-entry ledger rows (`SUM(debits) == SUM(credits)`) inside deadlock-free ACID transactions with strict idempotency and tenant isolation.

## v1 Requirements

Requirements for initial release. Each maps to the locked 6-phase roadmap.

### Infrastructure & Core Setup

- [ ] **INFRA-01**: Project initializes with clean modular layout (`app/api`, `app/core`, `app/models`, `app/schemas`, `app/services`, `tests/`) and `requirements.txt` for Python 3.11+
- [ ] **INFRA-02**: `app/core/config.py` validates environment settings via `pydantic-settings` (PostgreSQL 16 DSN, Redis 7 URL, JWT secrets, rate-limit parameters)
- [ ] **INFRA-03**: `app/core/database.py` configures SQLAlchemy 2.0 `asyncpg` engine, session factory (`READ COMMITTED` default), and Redis 7 async client
- [ ] **INFRA-04**: `Dockerfile` (running as non-root user) and `docker-compose.yml` orchestrate FastAPI, PostgreSQL 16, and Redis 7 (`noeviction` policy)
- [ ] **INFRA-05**: `app/main.py` exposes `/health` liveness/readiness probe verifying active PostgreSQL and Redis connectivity

### Multi-Tenant IAM, RBAC & Session Revocation

- [ ] **IAM-01**: Alembic migrations initialize and manage `Organization`, `User`, `Membership`, and `AuditLog` schemas
- [ ] **IAM-02**: User authentication hashes passwords using `Argon2id` (`argon2-cffi`) and issues `PyJWT` access and refresh tokens with unique `jti` claims
- [ ] **IAM-03**: Token refresh endpoint rotates refresh tokens and revokes used/logged-out token `jti`s in a Redis 7 `noeviction` blocklist
- [ ] **IAM-04**: Organizations can provision and authenticate via SHA-256 hashed API keys scoped to their workspace
- [ ] **IAM-05**: FastAPI dependencies strictly scope all queries by `organization_id` and enforce four RBAC roles (`OWNER`, `FINANCE_ADMIN`, `AUDITOR` read-only, `SUPPORT`)
- [ ] **IAM-06**: Sensitive IAM and administrative mutations append immutable records to `audit_logs`

### Append-Only Double-Entry Ledger & ACID Transfer Engine

- [ ] **LEDGER-01**: `Wallet`, `Transaction`, and `LedgerEntry` models store monetary amounts in `BIGINT` minor units scoped by ISO 4217 `currency` (`NPR`, `INR`, `KRW`, `USD`, etc.)
- [ ] **LEDGER-02**: `ledger_entries` enforces SQL `CHECK (amount > 0)` and append-only immutability (no `UPDATE` or `DELETE` on ledger rows)
- [ ] **LEDGER-03**: Every transfer writes matched `DEBIT` and `CREDIT` rows summing to `0` and updates `wallets.cached_balance` atomically in the same ACID transaction
- [ ] **LEDGER-04**: Transfer engine acquires deterministic row-level locks (`SELECT ... FOR UPDATE` ordered by `wallet_id` ASC) under `READ COMMITTED` isolation to guarantee deadlock freedom
- [ ] **LEDGER-05**: `/deposits` and `/payouts` endpoints book balanced double-entry rows against per-currency system omnibus contra accounts and support escrow hold/release states

### Idempotency Middleware & Sliding-Window Rate Limiter

- [ ] **IDEM-01**: Mutation endpoints (`/transfers`, `/deposits`, `/payouts`) require an `Idempotency-Key` header and reject missing keys with HTTP `400`/`422`
- [ ] **IDEM-02**: Idempotency service acquires an atomic Redis `SET NX EX` lock for in-flight keys (returning `409 Conflict` on concurrent duplicates) and persists completed status/response payloads in PostgreSQL `idempotency_records` for deterministic replay
- [ ] **RATE-01**: Redis 7 Lua-script sliding-window rate limiter enforces per-tenant/IP/user request quotas and returns HTTP `429 Too Many Requests` with `Retry-After` headers

### Automated Reconciliation & Concurrency Stress Tests

- [ ] **RECON-01**: Background reconciliation worker/service verifies `SUM(debits) == SUM(credits)` globally and per currency, plus `wallets.cached_balance == net(ledger_entries)` across all wallets
- [ ] **TEST-01**: Async Pytest concurrency suite fires 50 simultaneous transfers across overlapping wallets to prove zero deadlocks, zero race conditions, and exact balance integrity
- [ ] **TEST-02**: Async Pytest idempotency race suite fires concurrent requests with identical `Idempotency-Key` headers to verify single execution and deterministic replay

### Swagger Polish, Locust Benchmark & Architecture README

- [ ] **DOCS-01**: FastAPI OpenAPI/Swagger documentation includes rich request/response schemas, error models, authentication security schemes, and `Idempotency-Key` header specs
- [ ] **BENCH-01**: Locust load testing suite (`locustfile.py`) benchmarks high-throughput `/transfers`, `/deposits`, and `/payouts` under concurrent virtual users
- [ ] **DOCS-02**: Comprehensive `README.md` documents system architecture, the 5 engineering invariants, ERD/data flow diagrams, setup instructions, and benchmark metrics

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Cross-Currency FX & Webhooks

- **FX-01**: Cross-currency liquidity pools and automated FX conversion between multi-currency wallets (e.g., `USD` to `NPR`)
- **HOOK-01**: Outbound HMAC-signed webhook delivery worker with exponential backoff for transaction settlement events

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Mutable balance-only columns without ledger entries | Violates Engineering Invariant 1 (Zero Mutable-Only Balance Columns) |
| Floating-point (`FLOAT` / `REAL`) currency columns | Causes IEEE 754 precision loss; `BIGINT` minor units enforced |
| Third-party live bank wire / card rail integrations | External rails simulated via omnibus contra accounts to keep core engine self-contained |
| Frontend web/mobile application | Backend API engine focused on ledger, concurrency, and IAM architecture |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| INFRA-01 | Phase 1 | Pending |
| INFRA-02 | Phase 1 | Pending |
| INFRA-03 | Phase 1 | Pending |
| INFRA-04 | Phase 1 | Pending |
| INFRA-05 | Phase 1 | Pending |
| IAM-01 | Phase 2 | Pending |
| IAM-02 | Phase 2 | Pending |
| IAM-03 | Phase 2 | Pending |
| IAM-04 | Phase 2 | Pending |
| IAM-05 | Phase 2 | Pending |
| IAM-06 | Phase 2 | Pending |
| LEDGER-01 | Phase 3 | Pending |
| LEDGER-02 | Phase 3 | Pending |
| LEDGER-03 | Phase 3 | Pending |
| LEDGER-04 | Phase 3 | Pending |
| LEDGER-05 | Phase 3 | Pending |
| IDEM-01 | Phase 4 | Pending |
| IDEM-02 | Phase 4 | Pending |
| RATE-01 | Phase 4 | Pending |
| RECON-01 | Phase 5 | Pending |
| TEST-01 | Phase 5 | Pending |
| TEST-02 | Phase 5 | Pending |
| DOCS-01 | Phase 6 | Pending |
| BENCH-01 | Phase 6 | Pending |
| DOCS-02 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 25 total
- Mapped to phases: 25
- Unmapped: 0 ✓

---
*Requirements defined: 2026-10-28*
*Last updated: 2026-10-28 after initial definition*
