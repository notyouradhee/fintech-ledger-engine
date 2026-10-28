# Multi-Tenant FinTech Escrow, Double-Entry Ledger & IAM Engine

## What This Is

A production-grade B2B treasury, escrow, and digital wallet backend built with Python 3.11+, FastAPI (async), PostgreSQL 16 (SQLAlchemy 2.0 + asyncpg), and Redis 7. It provides multi-tenant workspace isolation, fine-grained RBAC, session & API-key authentication, and a mathematically verifiable append-only double-entry ledger supporting multi-currency integer minor units (`NPR`, `INR`, `KRW`, `USD`, etc.).

## Core Value

Mathematically provable financial integrity under high concurrency: every monetary movement is recorded as balanced, immutable double-entry ledger rows (`SUM(debits) == SUM(credits)`) inside deadlock-free ACID transactions with strict idempotency and tenant isolation.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Containerized async FastAPI + PostgreSQL 16 + Redis 7 infrastructure with non-root Docker execution and `/health` probes
- [ ] Workspace-scoped Multi-Tenant IAM (`Organization`, `User`, `Membership`, `AuditLog`) with Argon2id password hashing, JWT access/refresh rotation, Redis `jti` revocation blocklist (`noeviction`), and SHA-256 API keys
- [ ] Strict four-role RBAC enforcement (`OWNER`, `FINANCE_ADMIN`, `AUDITOR`, `SUPPORT`) across all tenant-scoped queries
- [ ] Append-only Double-Entry Ledger (`Wallet`, `Transaction`, `LedgerEntry`) using ISO 4217 multi-currency `BIGINT` minor units, SQL `CHECK (amount > 0)`, and system omnibus contra accounts for `/deposits` and `/payouts`
- [ ] Deadlock-free ACID transfer engine acquiring deterministic row locks (`SELECT ... FOR UPDATE` ordered by `wallet_id` ASC) under `READ COMMITTED` isolation while synchronizing `wallets.cached_balance`
- [ ] Strict `Idempotency-Key` middleware (Redis `SET NX EX` lock + PostgreSQL `idempotency_records` replay) on `/transfers`, `/deposits`, and `/payouts`
- [ ] Redis 7 Lua-script sliding-window rate limiter protecting API endpoints
- [ ] Immutable append-only `audit_logs` for sensitive operations and a background reconciliation worker verifying `SUM(debits) == SUM(credits)` across all accounts
- [ ] Concurrency stress test suite (50 simultaneous transfers & duplicate idempotency race tests), Locust load benchmark, OpenAPI/Swagger polish, and architecture documentation

### Out of Scope

- Direct third-party bank wire / card network integration (Stripe/Swift live webhooks) — Simulated via system omnibus contra accounts for deterministic ledger verification
- Cross-currency FX conversion market-making — Transfers occur between wallets of matching ISO 4217 currency in v1
- Frontend web/mobile UI — Pure backend API engine with Swagger/OpenAPI interactive documentation

## Context

- Designed as a high-assurance B2B treasury and escrow ledger capable of operating across international markets (e.g., Nepal `NPR`, India `INR`, Korea `KRW`, `USD`) by storing amounts in integer minor units (`BIGINT`) scoped by ISO 4217 currency code per wallet.
- External `/deposits` and `/payouts` maintain the global zero-sum invariant (`SUM(debits) == SUM(credits)`) by booking balanced double-entry rows against per-currency system omnibus/settlement contra accounts.
- `SUPPORT` role can view wallets/transactions and initiate support/refund workflows without permission to execute payouts or mutate IAM.

## Constraints

- **Tech Stack**: Python 3.11+, FastAPI (Async), Pydantic v2, `pydantic-settings`, PostgreSQL 16, SQLAlchemy 2.0 (`asyncpg`), Alembic, Redis 7 — Locked architecture specification
- **Security**: Argon2id (`argon2-cffi`), PyJWT (Access + Refresh rotation), Redis `jti` blocklist (`noeviction`), SHA-256 API keys — Zero-trust authentication & session control
- **Financial Invariant 1 (Zero Mutable-Only Balances)**: Append-only `ledger_entries` with SQL `CHECK (amount > 0)` synchronized atomically with `wallets.cached_balance`
- **Financial Invariant 2 (Deadlock-Free Locking)**: Deterministic `SELECT ... FOR UPDATE` ordered by `wallet_id` ASC under `READ COMMITTED` isolation
- **Financial Invariant 3 (Strict Idempotency)**: Mandatory `Idempotency-Key` header on `/transfers`, `/deposits`, `/payouts` backed by Redis lock + PostgreSQL persistence
- **Financial Invariant 4 (Multi-Tenancy & RBAC)**: Strict `organization_id` scoping and 4 roles (`OWNER`, `FINANCE_ADMIN`, `AUDITOR`, `SUPPORT`)
- **Financial Invariant 5 (Audit & Reconciliation)**: Append-only `audit_logs` and automated zero-sum reconciliation checker (`SUM(debits) == SUM(credits)`)
- **Roadmap Structure**: Locked to the exact 6-phase progression specified by the user

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Multi-currency ISO 4217 + `BIGINT` minor units | Supports global currencies (`NPR`, `INR`, `KRW`, `USD`) with zero floating-point rounding risk | — Pending |
| System omnibus contra accounts for deposits/payouts | Ensures every external deposit and payout is a balanced double-entry transaction so `SUM(debits) == SUM(credits)` holds globally | — Pending |
| Deterministic `wallet_id` ASC locking under `READ COMMITTED` | Prevents AB-BA deadlocks when concurrent transfers lock overlapping wallet pairs | — Pending |
| Dual-layer Idempotency (Redis `SET NX EX` + Postgres table) | Blocks in-flight race conditions via Redis while persisting deterministic response payloads in Postgres | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-10-28 after initialization*
