# Chioma

**Chioma** is an open‑source platform built on the **Stellar blockchain** that connects **landlords (property owners), house agents, and tenants** through transparent, low‑cost, and programmable rental payments.

Chioma focuses on what blockchains do best: **money movement, trust minimization, and verifiable agreements**, while keeping complex business logic off‑chain for usability and scale.

Our goal is to modernize rental transactions—especially in emerging markets—by reducing friction, disputes, and payment inefficiencies using Stellar’s fast settlement and low fees.

---

Telegram: https://t.me/chiomagroup

> **New contributor?** Jump to the [Quickstart](#quickstart-clone--running-full-stack) to go from clone to a running stack.

## Figma

https://www.figma.com/design/2iA2B5gmRupQtzjYv3LSx4/Chioma?node-id=0-1&t=KEwTSTGKUDT0ievI-1

## Why Chioma?

Rental systems today suffer from:

- Manual and opaque rent collection
- Delayed settlements between tenants, agents, and landlords
- Trust issues around deposits and commissions
- High transaction costs for cross‑border or multi‑currency payments

**Chioma solves this by combining Stellar’s payment rails with a clean, open‑source marketplace layer.**

---

## Why Stellar?

Chioma is designed **specifically for Stellar**, not just deployed on it.

Stellar enables:

- **Fast finality** (seconds)
- **Ultra‑low transaction fees**
- **Built‑in decentralized exchange (DEX)**
- **Anchor‑based fiat on/off‑ramps**
- **Multi‑sig & conditional transactions** for escrow‑like flows

This makes Stellar ideal for:

- Monthly rent payments
- Security deposit escrow
- Automated agent commissions
- Multi‑currency rent settlement

---

## What Chioma Does (High Level)

Chioma is **not** a monolithic on‑chain app.

Instead, it uses a **hybrid architecture**:

### On‑Chain (Stellar)

- Rent payments
- Security deposit holding & release
- Agent commission distribution
- Tokenized rent obligations
- Immutable transaction records

### Off‑Chain

- Property listings
- User profiles (landlords, agents, tenants)
- Matching & discovery
- Messaging & notifications
- Compliance & moderation

This keeps costs low while preserving decentralization where it matters most.

---

## Core Concepts

### 1. Rent Payment Flows

- Tenants pay rent using Stellar assets (USDC, local fiat tokens via anchors, or project tokens)
- Payments settle instantly
- Funds are split automatically between landlord and agent

### 2. Security Deposits (Escrow‑Like)

- Deposits are locked using Stellar’s multi‑sig or pre‑authorized transactions
- Released on move‑out approval
- Transparent and verifiable by all parties

### 3. Agent Commissions

- Agents receive commissions automatically
- No manual tracking or disputes
- Commission logic is enforced at payment time

---

## Architecture Overview

```text
Frontend (Web / Mobile)
   │
   ├── Property Listings
   ├── User Dashboards
   ├── Payment UI
   │
Backend (API + Indexer)
   │
   ├── Business Logic
   ├── Stellar SDK Integration
   ├── Compliance Hooks
   │
Stellar Network
   │
   ├── Asset Issuance
   ├── Payments & Escrow
   ├── DEX Swaps
```

---

## Quickstart (clone → running full stack)

This is the single source of truth for local setup. The `backend/`, `frontend/` and `contract/` docs link here instead of repeating it.

### 1. Prerequisites

| Tool | Version | Used by |
| --- | --- | --- |
| Git | any recent | all |
| Node.js | 20 LTS or newer | backend, frontend |
| pnpm | 10 (`corepack enable`) | backend, frontend |
| Docker + Docker Compose | any recent | Postgres & Redis |
| Rust + `wasm32v1-none` target | stable | contracts (optional) |
| Stellar CLI | latest | contracts (optional) |

### 2. Clone and install

```bash
git clone https://github.com/chioma-housing-protocol-I/chioma.git
cd chioma
corepack enable
pnpm install                       # root tooling (husky, prettier)
pnpm --dir backend install
pnpm --dir frontend install
```

### 3. Start Postgres and Redis

```bash
docker compose -f backend/docker-compose.yml up -d db redis
```

This exposes Postgres on **localhost:5433** (user `postgres`, password `postgres`, db `chioma_db`) and Redis on **localhost:6379**.

### 4. Configure environment

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env.local
```

Then edit `backend/.env` so it matches the Docker database:

```dotenv
DB_HOST=localhost
DB_PORT=5433
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_NAME=chioma_db
```

The remaining defaults (`PAYMENT_GATEWAY=mock`, `PORT=3000`, etc.) work for local development. The frontend defaults already point at `http://localhost:3000`.

### 5. Run the backend (port 3000)

```bash
cd backend
pnpm run migration:run
pnpm run seed:all                  # optional: admin/agent/tenant/landlord users
pnpm run start:dev
```

API: <http://localhost:3000/api> · Swagger docs: <http://localhost:3000/api/docs>

### 6. Run the frontend (port 3001)

In a second terminal:

```bash
cd frontend
pnpm dev -p 3001
```

Open <http://localhost:3001>.

### 7. Smart contracts (optional)

The app runs without deploying contracts. To build and test them:

```bash
rustup target add wasm32v1-none
cd contract
cargo test
stellar contract build
```

Deployed contract IDs go in `backend/.env` (`ESCROW_CONTRACT_ID`, `DISPUTE_CONTRACT_ID`) and `frontend/.env.local` (`NEXT_PUBLIC_CHIOMA_CONTRACT_ID`).

### 8. Before opening a PR

Each package mirrors its CI pipeline locally: `make ci` in `backend/` or `frontend/`, and `./check-all.sh` in `contract/`. Package-specific conventions live in [`backend/CONTRIBUTING.md`](backend/CONTRIBUTING.md), [`frontend/CONTRIBUTING.md`](frontend/CONTRIBUTING.md) and [`contract/CONTRIBUTING.md`](contract/CONTRIBUTING.md).

### Troubleshooting

- **`ECONNREFUSED 127.0.0.1:5432`**: `DB_PORT` is still `5432`; the Docker database listens on `5433`.
- **Port 3000 already in use**: the frontend was started without `-p 3001`, or another process owns the port.
- **`docker compose` not found**: use `docker-compose` (v1) or upgrade Docker.

---

## Open Source First

Chioma is being built **fully open‑source**

We welcome:

- Contributors
- Reviewers
- Anchor operators
- Protocol researchers

---

## You Should Know

Chioma is not just a rental app.

It is **open financial infrastructure for housing**, built on Stellar, and designed to scale across borders, currencies, and communities.

## Property Listing Wizard

The Property Listing Wizard is a guided 8-step flow for landlords to create high-quality rental listings with AI-assisted content generation and automated draft saving.

### API Endpoints

- `POST /property-listings/wizard/start` - Initialize a new property draft.
- `GET /property-listings/wizard/:id/draft` - Resume an existing draft.
- `PATCH /property-listings/wizard/:id/step` - Save current step data and run validation.
- `DELETE /property-listings/wizard/:id/draft` - Discard a draft.
- `POST /property-listings/wizard/:id/publish` - Finalize and publish the property.

### AI Features

The wizard includes server-side AI helpers for:

- **Pricing Suggestions**: Recommended rent ranges based on property type and location.
- **Description Generation**: Compelling property and neighborhood blurbs.
- **Completeness Scoring**: Real-time analysis of listing quality with improvement tips.

### Draft Expiry

Drafts automatically expire after **30 days** of inactivity. A cleanup task runs periodically to remove expired drafts.
