# Soroban Loyalty

A modular, production-grade on-chain loyalty platform built on the **Stellar** network using **Soroban** smart contracts. Businesses create reward campaigns, users earn tokenized incentives (LYT), and everything is stored transparently on-chain.

- **Repository:** https://github.com/milah-247/Soroban_loyalty
- **Live preview:** https://wave-lyt-vault.lovable.app
- **Latest release in history:** `v1.57.0`. See the [Changelog](CHANGELOG.md).
- **Terminology:** see the [Glossary](docs/glossary.md)

---

## Table of Contents

- [Project History & Maintainership](#project-history--maintainership)
- [Screenshots](#screenshots)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Smart Contracts](#smart-contracts)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Deploy Contracts](#deploy-contracts)
- [Testing](#testing)
- [API Reference](#api-reference)
- [Project Structure](#project-structure)
- [Operations: CI/CD, Monitoring & Infrastructure](#operations-cicd-monitoring--infrastructure)
- [PostgreSQL Connection Pool](#postgresql-connection-pool)
- [Security](#security)
- [Documentation Index](#documentation-index)
- [Contributing](#contributing)
- [Code of Conduct](#code-of-conduct)

---

## Project History & Maintainership

This project was created in April 2026 by **[Dev-Odun](https://github.com/Dev-Odun)** under the [`soroban-loyalty`](https://github.com/soroban-loyalty) GitHub organization (`soroban-loyalty/Soroban-loyalty`). It grew through open-source contributions from more than 80 developers, with 66 tagged releases (`v1.0.0` → `v1.57.0`).

The original maintainer account (Dev-Odun) has since been suspended and is no longer accessible. The same maintainer now continues the project under the **[milah-247](https://github.com/milah-247)** account, in this repository.

- **History preserved:** the full Git history was carried over unchanged. That includes the original root commit, every contributor's authorship, and all release tags.
- **Nothing rewritten:** no commits were rewritten and no authors were changed.
- **Credit:** all past contributions remain credited to their original authors. Run `git shortlog -sne` to see them.

---

## Screenshots

<img width="1920" height="1080" alt="Soroban Loyalty screenshot 1" src="https://github.com/user-attachments/assets/3f6a7f4f-e559-49a7-b5fa-88942d3a93e2" />
<img width="1920" height="1080" alt="Soroban Loyalty screenshot 2" src="https://github.com/user-attachments/assets/a668bb79-bd31-47fc-96a2-2c12989ff050" />
<img width="1920" height="1080" alt="Soroban Loyalty screenshot 3" src="https://github.com/user-attachments/assets/03958210-a869-4455-8097-5bf231ec5335" />
<img width="1920" height="1080" alt="Soroban Loyalty screenshot 4" src="https://github.com/user-attachments/assets/3619774a-0088-4217-913a-f069c8c0bd31" />

---

## Features

**For users**
- Connect a [Freighter](https://www.freighter.app/) wallet and sign transactions in the browser
- Browse active campaigns, claim rewards (mints LYT), and redeem LYT (burns tokens)
- Referral claims and vested reward claims
- Dashboard with animated balance, transaction history, CSV export, and a print view
- Profile page, global search, and an in-app help center with a searchable FAQ

**For merchants**
- Create, edit, pause/resume, soft-delete/restore, and drag-and-drop reorder campaigns
- Campaign image upload and share links
- Analytics dashboard with charts, stat cards, date-range filters, and A/B experiment stats

**Platform**
- On-chain event indexer with checkpointing and deduplication
- JWT authentication using Stellar wallet challenge/verify
- Rate limiting, Zod input validation, XSS sanitization, and an audit log for sensitive operations
- Redis caching, pagination, search, and filtering on campaign listings
- Internationalization (English and Spanish), dark mode, onboarding flow, and WCAG 2.1 AA accessibility work
- Network status indicator and Soroban error boundaries
- Governance contract for token-weighted proposals and voting

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Stellar Network                          │
│                                                                 │
│  ┌──────────────┐   ┌──────────────────┐   ┌────────────────┐   │
│  │ Token (LYT)  │◄──│ Rewards Contract │──►│Campaign Contract│  │
│  │  mint/burn   │   │  claim/redeem    │   │ create/manage  │   │
│  └──────────────┘   └──────────────────┘   └────────────────┘   │
│                     ┌──────────────────┐                        │
│                     │   Governance     │ propose / vote / exec  │
│                     └──────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
          ▲                    ▲
          │ Soroban RPC        │ Events
          │                    │
┌─────────┴────────────────────┴──────────────────────────────────┐
│                  Backend (Node.js / Express 5)                  │
│                                                                 │
│  ┌──────────────┐   ┌──────────────────┐   ┌────────────────┐   │
│  │   Indexer    │   │ Campaign Service │   │ Reward Service │   │
│  │ (event poll) │   │  (DB read/write) │   │ (DB read/write)│   │
│  └──────┬───────┘   └────────┬─────────┘   └───────┬────────┘   │
│         └────────────────────┼─────────────────────┘            │
│                              ▼                                  │
│                PostgreSQL  +  Redis (cache)                     │
└─────────────────────────────────────────────────────────────────┘
          ▲
          │ REST API
          │
┌─────────┴───────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                          │
│                                                                 │
│  /dashboard     — claim rewards, view balance                   │
│  /merchant      — create & manage campaigns                     │
│  /analytics     — campaign performance stats                    │
│  /campaigns, /transactions, /profile, /help                     │
│                                                                 │
│  Freighter wallet integration (sign & submit transactions)      │
└─────────────────────────────────────────────────────────────────┘
```

Claim and redeem transactions are signed in the browser by Freighter and submitted directly to Soroban RPC. The backend never holds user keys. It indexes the resulting on-chain events into PostgreSQL.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Smart contracts | Rust `1.85.0` (pinned in `rust-toolchain.toml`), Soroban SDK, `wasm32-unknown-unknown` |
| Backend | Node.js 20+, TypeScript, Express 5, `pg`, `ioredis`, Zod, `node-pg-migrate`, `prom-client`, Sentry, Swagger/OpenAPI |
| Frontend | Next.js 16, React 19, TypeScript, `next-intl`, Recharts, `@stellar/stellar-sdk`, `@stellar/freighter-api` |
| Data | PostgreSQL, Redis |
| Observability | Prometheus, Alertmanager, Grafana, ELK (Elasticsearch, Logstash, Filebeat, Kibana), Sentry |
| Infrastructure | Docker Compose, Nginx, Kubernetes manifests, Terraform, cert-manager, blue-green deployment |
| Release | semantic-release (Conventional Commits → `CHANGELOG.md` + GitHub Releases) |

---

## Smart Contracts

The Cargo workspace (`Cargo.toml`) contains four contracts:

| Contract | Crate | Description |
|---|---|---|
| `token` | `soroban-loyalty-token` | Fungible LYT token: mint, burn, transfer, allowance/`transfer_from`. Minting is restricted, and admin changes go through multisig propose/approve/execute. |
| `campaign` | `soroban-loyalty-campaign` | Merchants create campaigns with a reward amount and an expiration date. Supports pause/resume, role-based access, an emergency pause, and time-locked upgrades. |
| `rewards` | `soroban-loyalty-rewards` | Users claim rewards (mints LYT) and redeem them (burns LYT). Double-claims are prevented. Also supports referral claims, vesting, an emergency pause, and schema migration. |
| `governance` | `soroban-loyalty-governance` | Token-holder proposals: propose, vote, finalize, execute, and cancel. |

See [contracts/README.md](contracts/README.md), [docs/contracts.md](docs/contracts.md), and [GAS_BENCHMARKS.md](GAS_BENCHMARKS.md).

---

## Quick Start

### Prerequisites

- [Rust](https://rustup.rs/) (the toolchain in `rust-toolchain.toml` is installed automatically) + `wasm32-unknown-unknown` target
- [Stellar CLI](https://developers.stellar.org/docs/tools/developer-tools/cli/install-stellar-cli)
- [Docker + Docker Compose](https://docs.docker.com/get-docker/)
- [Node.js 20+](https://nodejs.org/)
- [Freighter](https://www.freighter.app/) browser extension (to use the frontend)

### 1. Clone & configure

```bash
git clone https://github.com/milah-247/Soroban_loyalty.git
cd Soroban_loyalty
cp .env.example .env   # then fill in the values. See Configuration below.
```

### 2. Run with Docker

```bash
docker compose up --build
```

This command automatically picks up `docker-compose.override.yml` for development-only settings, such as local source mounts and dev commands.

For production-style startup:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up --build
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:3001 |
| Soroban local node | http://localhost:8000 |
| PostgreSQL | `localhost:5432` |
| Redis | `localhost:6379` |
| Prometheus | http://localhost:9090 |
| Alertmanager | http://localhost:9093 |
| Grafana | http://localhost:3002 |
| Elasticsearch | http://localhost:9200 |
| Kibana | http://localhost:5601 |

### 3. Run locally (without Docker)

**Start PostgreSQL and Redis** (or use Docker just for these):
```bash
docker compose up postgres redis -d
```

**Backend:**
```bash
cd backend
npm install
npm run migrate:up
npm run dev
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

---

## Configuration

All configuration is read from environment variables. [`.env.example`](.env.example) documents every variable. Each one is marked **REQUIRED**, **OPTIONAL**, or **SECRET**. The most important ones are:

| Variable | Purpose |
|---|---|
| `SOROBAN_RPC_URL`, `NETWORK_PASSPHRASE` | Soroban RPC endpoint and network (local, testnet, or mainnet) |
| `DATABASE_URL` | PostgreSQL connection string (secret) |
| `REDIS_URL` | Redis cache |
| `REWARDS_CONTRACT_ID`, `CAMPAIGN_CONTRACT_ID`, `TOKEN_CONTRACT_ID` | Deployed contract IDs (set automatically by the deploy script) |
| `JWT_SECRET` | Signs session tokens. Generate one with `openssl rand -hex 32`. |
| `ENABLE_INDEXER`, `START_LEDGER` | Indexer toggle and the ledger to start from on a cold start |
| `RATE_LIMIT_*` | Rate-limit window and maximum request counts |
| `SECRETS_ARN`, `AWS_REGION` | Optional: load secrets from AWS Secrets Manager at startup |
| `SENTRY_DSN`, `SLACK_WEBHOOK_URL` | Error monitoring and alerting |
| `NEXT_PUBLIC_*` | Browser-visible frontend config (API URL, RPC URL, passphrase, contract IDs) |

Never commit real secrets. For production secret handling, see [docs/secrets-management-runbook.md](docs/secrets-management-runbook.md).

---

## Deploy Contracts

```bash
# Add the Rust wasm target (once)
rustup target add wasm32-unknown-unknown

# Deploy to the local network
./scripts/deploy-contracts.sh local <YOUR_SECRET_KEY>

# Deploy to testnet
./scripts/deploy-contracts.sh testnet <YOUR_SECRET_KEY>
```

The script builds the contracts, deploys them, initializes them with the correct cross-contract references, and updates your `.env` automatically. For full environment deployments, see [docs/deployment-guide.md](docs/deployment-guide.md).

---

## Testing

**Contracts**
```bash
cargo test                                  # all contracts
cargo test -p soroban-loyalty-token
cargo test -p soroban-loyalty-campaign
cargo test -p soroban-loyalty-rewards
cargo test -p soroban-loyalty-governance
```

The contract tests cover:
- **Token:** mint, transfer, burn, and overflow/underflow guards
- **Campaign:** creation, expiry validation, deactivation, and time-based expiry
- **Rewards:** claims, double-claim prevention, rejection of inactive or expired campaigns, and redeem burning tokens

**Backend**
```bash
cd backend
npm test                     # unit tests with coverage
npm run test:integration     # integration tests
npm run lint && npm run typecheck
npm run test:perf:campaigns  # k6 load test (requires k6)
```

**Frontend**
```bash
cd frontend
npm test
npm run lint
```

**End-to-end:** Playwright tests live in [`e2e/`](e2e) and run in CI (`.github/workflows/e2e.yml`). A Postman collection is available in [`postman/`](postman).

---

## API Reference

The full reference is in [docs/api-reference.md](docs/api-reference.md). The backend also generates an OpenAPI spec from `backend/src/openapi.ts`.

### Campaigns

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/campaigns` | - | List campaigns (pagination, search, filters). Returns `{ campaigns: Campaign[]; total: number }`. |
| `GET` | `/campaigns/:id` | - | Get a campaign by ID. Returns `{ campaign: Campaign }`. |
| `POST` | `/campaigns` | JWT | Create a campaign |
| `PATCH` | `/campaigns/:id` | JWT | Update a campaign |
| `PATCH` | `/campaigns/reorder` | - | Reorder campaigns |
| `DELETE` | `/campaigns/:id` | - | Soft-delete a campaign |
| `POST` | `/campaigns/:id/restore` | - | Restore a soft-deleted campaign |

The same router is also mounted at `/merchant/campaigns`, where every route requires a JWT.

### Rewards

| Method | Path | Description |
|---|---|---|
| `GET` | `/user/:address/rewards` | Get a user's rewards. Returns `{ data: Reward[]; total; limit; offset }`. |
| `POST` | `/user/:address/rewards/claim` | Record a reward claim |

### Analytics

| Method | Path | Description |
|---|---|---|
| `GET` | `/analytics?days=N` | Aggregated reward metrics (`AnalyticsData`) |
| `GET` | `/analytics/campaigns` | Per-campaign analytics |

### Auth

| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/challenge` | Request a wallet sign-in challenge |
| `POST` | `/auth/verify` | Verify the signed challenge and receive a JWT |

### System

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | Health of the database and RPC. Returns 200 if healthy, 503 otherwise. |
| `GET` | `/metrics` | Prometheus metrics |

> **Note:** route handlers for `GET /user/:address/transactions` (`transaction.routes.ts`) and `GET /audit-logs` (`admin.routes.ts`) exist, but they are not currently mounted in `backend/src/app.ts`.

### TypeScript Types

| File | Types |
|---|---|
| `backend/src/types/index.ts` | `Campaign`, `CampaignFilters`, `Reward`, `TransactionRecord`, `AnalyticsData`, `CampaignAnalyticsData`, `AuditLogEntry`, `AuditAction`, `AuditLogFilters` |
| `frontend/src/types/index.ts` | `Campaign`, `Reward`, `TransactionRecord`, `AnalyticsData` |

---

## Project Structure

```
Soroban_loyalty/
├── contracts/            # Soroban smart contracts (Rust workspace)
│   ├── token/            # LYT fungible token
│   ├── campaign/         # Campaign management
│   ├── rewards/          # Claim, redeem, referral, vesting
│   └── governance/       # Proposals & voting
├── backend/              # Node.js / Express API + event indexer
│   └── src/
│       ├── app.ts, index.ts       # App setup & server entry
│       ├── indexer/               # On-chain event indexer
│       ├── routes/                # REST route handlers
│       ├── services/              # Business logic (campaign, reward, analytics, audit, …)
│       ├── middleware/            # Rate limiting, validation, sanitization, errors
│       └── db.ts, soroban.ts, auth.ts, secrets.ts, metrics.ts, sentry.ts
├── frontend/             # Next.js app
│   └── src/
│       ├── app/                   # Pages (dashboard, merchant, analytics, campaigns, …)
│       ├── components/            # UI components
│       ├── context/, hooks/, lib/ # Wallet state, hooks, API & Soroban clients
│       └── i18n/, locales/        # English & Spanish translations
├── database/             # schema.sql, migrations, query-plan notes
├── e2e/                  # Playwright end-to-end tests
├── benchmarks/           # k6 performance tests
├── postman/              # API collection
├── monitoring/           # Prometheus, Alertmanager, Grafana, ELK, uptime
├── infra/                # Kubernetes, Terraform, cert-manager, CDN, blue-green, secrets
├── nginx/                # Reverse proxy
├── scripts/              # Deploy, backup, blue-green switch, secret scan, …
├── docs/                 # Guides, runbooks, ADRs, audits
├── docker-compose*.yml
└── .env.example
```

---

## Operations: CI/CD, Monitoring & Infrastructure

**GitHub Actions workflows** (`.github/workflows/`):

| Area | Workflows |
|---|---|
| CI | `ci.yml`, `backend-ci.yml`, `frontend-ci.yml`, `rust-ci.yml`, `contracts-ci.yml`, `e2e.yml`, `changelog-check.yml` |
| Security | `contracts-security.yml` (plus gitleaks and Trivy configuration in the repo root) |
| Release | `release.yml` (semantic-release) |
| Deploy | `cd-staging.yml`, `deploy-staging.yml`, `blue-green-deploy.yml`, `cleanup-staging.yml`, `refresh-staging-data.yml`, `terraform.yml` |

Dependabot keeps npm, Cargo, and GitHub Actions dependencies up to date.

**Releases.** Commits follow [Conventional Commits](https://www.conventionalcommits.org/). On every push to `main`, semantic-release picks the next version from the existing tags, then updates `CHANGELOG.md` and publishes a GitHub Release.

**Monitoring and infrastructure.** See [docs/deployment-guide.md](docs/deployment-guide.md), [docs/blue-green-runbook.md](docs/blue-green-runbook.md), [docs/cdn-setup.md](docs/cdn-setup.md), [docs/incident-response-playbook.md](docs/incident-response-playbook.md), and [docs/runbooks/](docs/runbooks).

---

## PostgreSQL Connection Pool

The backend uses `pg.Pool`, and all four sizing settings are configurable through environment variables:

| Variable | Default | Description |
|---|---|---|
| `DB_POOL_MAX` | `10` | Maximum concurrent connections. |
| `DB_POOL_MIN` | `2` | Minimum idle connections kept alive. |
| `DB_POOL_IDLE_TIMEOUT_MS` | `30000` | Milliseconds before an idle connection is closed. |
| `DB_POOL_CONNECTION_TIMEOUT_MS` | `5000` | Milliseconds to wait for a free connection before throwing. |

**Sizing guidance**

- **`DB_POOL_MAX`:** a common starting formula is `(vCPUs × 2) + effective_spindle_count`. For a 2-vCPU app server talking to a `db.t3.medium` (2 vCPU), `10` is a safe default. Across all app instances combined, never exceed the database's `max_connections` (100 by default on RDS).
- **`DB_POOL_MIN`:** keep this at `2` so the first request after an idle period doesn't pay connection-setup latency. Set it to `0` in serverless or ephemeral environments.
- **`DB_POOL_IDLE_TIMEOUT_MS`:** lower it to `10000` in low-traffic or serverless deployments to release connections back to the database sooner.
- **`DB_POOL_CONNECTION_TIMEOUT_MS`:** `5000` is a reasonable upper bound. Lower it to `2000` if you prefer to fail fast when the pool is exhausted.

Pool exhaustion errors are logged at `error` level with `totalCount`, `idleCount`, and `waitingCount`, so they're easy to spot.

---

## Security

To report a vulnerability, see [SECURITY.md](./SECURITY.md) for the process and response timeline. The threat model is in [docs/security-model.md](docs/security-model.md), and audit material is in [docs/audits/](docs/audits).

- All sensitive contract functions use `require_auth()`
- Double-claim prevention: claimed state is written **before** external calls (reentrancy guard)
- Overflow-safe arithmetic through `checked_add`, plus Rust's `overflow-checks = true` in release builds
- Token minting is restricted to the Rewards contract (set as admin during deploy)
- Emergency pause on the campaign and rewards contracts
- No secret keys in code: all keys come from environment variables or a secrets manager. A pre-commit secret scan is available (`scripts/pre-commit-secret-scan.sh`).

---

## Documentation Index

| Topic | Document |
|---|---|
| New contributor walkthrough | [docs/onboarding.md](docs/onboarding.md) |
| End-user guide | [docs/user-guide.md](docs/user-guide.md) |
| Merchant guide | [docs/merchant-guide.md](docs/merchant-guide.md) |
| API reference | [docs/api-reference.md](docs/api-reference.md) |
| Contracts | [docs/contracts.md](docs/contracts.md) |
| Database & ERD | [docs/database.md](docs/database.md) |
| Indexer | [docs/indexer.md](docs/indexer.md) |
| Freighter integration | [docs/freighter-integration.md](docs/freighter-integration.md) |
| Folder structure | [docs/structure.md](docs/structure.md) |
| Architecture decisions | [docs/adr/](docs/adr) |
| Glossary | [docs/glossary.md](docs/glossary.md) |

---

## Contributing

We welcome contributions from developers of all skill levels!

- **New to the project?** The [Onboarding Guide](docs/onboarding.md) walks you through it step by step.
- **Ready to contribute?** [CONTRIBUTING.md](CONTRIBUTING.md) explains how to find issues, set up your environment, follow the code style, and open pull requests.
- **Looking for somewhere to start?** Try an issue labeled [`good first issue`](https://github.com/milah-247/Soroban_loyalty/labels/good%20first%20issue).

Please use [Conventional Commit](https://www.conventionalcommits.org/) messages (`feat:`, `fix:`, `docs:`, …) so releases and the changelog are generated correctly.

---

## Code of Conduct

This project follows the [Code of Conduct](./CODE_OF_CONDUCT.md). By participating, you are expected to uphold these guidelines.
