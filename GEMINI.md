<!-- GSD:project-start source:PROJECT.md -->

## Project

**Multi-Tenant FinTech Escrow, Double-Entry Ledger & IAM Engine**

A production-grade B2B treasury, escrow, and digital wallet backend built with Python 3.11+, FastAPI (async), PostgreSQL 16 (SQLAlchemy 2.0 + asyncpg), and Redis 7. It provides multi-tenant workspace isolation, fine-grained RBAC, session & API-key authentication, and a mathematically verifiable append-only double-entry ledger supporting multi-currency integer minor units (`NPR`, `INR`, `KRW`, `USD`, etc.).

**Core Value:** Mathematically provable financial integrity under high concurrency: every monetary movement is recorded as balanced, immutable double-entry ledger rows (`SUM(debits) == SUM(credits)`) inside deadlock-free ACID transactions with strict idempotency and tenant isolation.

### Constraints

- **Tech Stack**: Python 3.11+, FastAPI (Async), Pydantic v2, `pydantic-settings`, PostgreSQL 16, SQLAlchemy 2.0 (`asyncpg`), Alembic, Redis 7 — Locked architecture specification
- **Security**: Argon2id (`argon2-cffi`), PyJWT (Access + Refresh rotation), Redis `jti` blocklist (`noeviction`), SHA-256 API keys — Zero-trust authentication & session control
- **Financial Invariant 1 (Zero Mutable-Only Balances)**: Append-only `ledger_entries` with SQL `CHECK (amount > 0)` synchronized atomically with `wallets.cached_balance`
- **Financial Invariant 2 (Deadlock-Free Locking)**: Deterministic `SELECT ... FOR UPDATE` ordered by `wallet_id` ASC under `READ COMMITTED` isolation
- **Financial Invariant 3 (Strict Idempotency)**: Mandatory `Idempotency-Key` header on `/transfers`, `/deposits`, `/payouts` backed by Redis lock + PostgreSQL persistence
- **Financial Invariant 4 (Multi-Tenancy & RBAC)**: Strict `organization_id` scoping and 4 roles (`OWNER`, `FINANCE_ADMIN`, `AUDITOR`, `SUPPORT`)
- **Financial Invariant 5 (Audit & Reconciliation)**: Append-only `audit_logs` and automated zero-sum reconciliation checker (`SUM(debits) == SUM(credits)`)
- **Roadmap Structure**: Locked to the exact 6-phase progression specified by the user

<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->

## Technology Stack

Technology stack not yet documented. Will populate after codebase mapping or first phase.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.agents/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-fast` for a trivial task inline, with no subagents and no PLAN.md
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
