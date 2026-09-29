# Quickstart: clone → running full stack

This is the **single source of truth** for local setup. The READMEs in `backend/`, `frontend/`, and `contract/` link here instead of repeating these steps.

## 1. Prerequisites

| Tool                          | Version                  | Needed for                     |
| ----------------------------- | ------------------------ | ------------------------------ |
| Git                           | any recent               | cloning                        |
| Node.js                       | 20 LTS or newer          | backend + frontend             |
| pnpm                          | 10.x (`corepack enable`) | backend + frontend             |
| Docker + Docker Compose       | recent                   | Postgres + Redis               |
| Rust + `wasm32v1-none` target | stable                   | smart contracts (optional)     |
| Stellar CLI                   | latest                   | deploying contracts (optional) |
# Quickstart: clone to running stack

This is the single source of truth for local setup. Sub-READMEs link here instead of restating it.

## 1. Prerequisites

| Tool                          | Version                  | Needed for           |
| ----------------------------- | ------------------------ | -------------------- |
| Git                           | any recent               | cloning              |
| Node.js                       | 20 LTS or newer          | backend, frontend    |
| pnpm                          | 10.x (`corepack enable`) | backend, frontend    |
| Docker + Docker Compose       | recent                   | Postgres + Redis     |
| Rust + `wasm32v1-none` target | stable                   | contracts (optional) |
| Stellar CLI                   | latest                   | contracts (optional) |

## 2. Clone

```bash
git clone https://github.com/chioma-housing-protocol-I/chioma.git
cd chioma
```

## 3. Start infrastructure (Postgres + Redis)

```bash
cd backend
docker compose up -d        # Postgres on localhost:5433, Redis on localhost:6379
```

## 4. Backend (NestJS API → http://localhost:3000)

```bash
# still in backend/
cp .env.example .env
```

Edit `.env` so it matches the Docker services:

```dotenv
DB_HOST=localhost
DB_PORT=5433
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_NAME=chioma_db
pnpm install   # root git hooks (husky, lint-staged)
```

## 3. Start Postgres and Redis

```bash
cd backend
docker compose up -d db redis
```

This exposes Postgres on **localhost:5433** (user `postgres`, password `postgres`, database `chioma_db`) and Redis on **localhost:6379**.

## 4. Backend (NestJS API, port 3000)

```bash
cd backend
cp .env.example .env
```

Edit `.env` so the database settings match the Docker container:

```env
DB_PORT=5433
DB_PASSWORD=postgres
```

Then install, migrate, seed, and run:

```bash
pnpm install
pnpm run migration:run
pnpm run seed:all           # optional: admin/agent/tenant/landlord users
pnpm run start:dev
```

- API: http://localhost:3000/api
- Swagger docs: http://localhost:3000/api/docs

## 5. Frontend (Next.js → http://localhost:3001)

In a new terminal from the repo root:

```bash
cd frontend
cp .env.example .env.local  # defaults point at the backend on :3000
pnpm install
pnpm dev -p 3001
```

Open http://localhost:3001. The backend already uses port 3000, so the frontend must run on 3001.

## 6. Smart contracts (optional)

Only needed if you are working on the Soroban contracts.

```bash
rustup target add wasm32v1-none
cd contract
cargo test                  # run contract tests
stellar contract build      # build all contracts to WASM
```

To point the frontend at a deployed contract, set `NEXT_PUBLIC_CHIOMA_CONTRACT_ID` in `frontend/.env.local`.

## 7. Before opening a PR

Each package has a script that mirrors CI:

```bash
cd backend  && ./check-all.sh
cd frontend && make ci
cd contract && ./check-all.sh
```

See each package's `CONTRIBUTING.md` for coding conventions.

## Troubleshooting

- **`ECONNREFUSED` to Postgres** – check `docker compose ps` in `backend/` and that `DB_PORT=5433`.
- **Port 3000 already in use** – the backend owns 3000; run the frontend with `-p 3001`.
- **Reset the database** – `docker compose down -v && docker compose up -d`, then re-run migrations.
pnpm run seed:all        # optional: admin/agent/tenant/landlord demo users
pnpm run start:dev       # http://localhost:3000
```

## 5. Frontend (Next.js, port 3001)

In a new terminal:

```bash
cd frontend
cp .env.example .env.local
pnpm install
pnpm dev -- -p 3001      # http://localhost:3001
```

The defaults in `.env.example` point `NEXT_PUBLIC_API_URL` at `http://localhost:3000` (the backend).

## 6. Smart contracts (optional)

```bash
cd contract
rustup target add wasm32v1-none
cargo build
cargo test
```

Use `stellar contract build` to produce deployable WASM. Deployment and contract details are in [contract/README.md](./contract/README.md).

## 7. Check it works

- API: `curl http://localhost:3000` responds
- App: open http://localhost:3001

## Before opening a PR

Every package has a `check-all.sh` that mirrors CI:

```bash
cd backend && ./check-all.sh
cd frontend && ./check-all.sh   # or: make ci
cd contract && ./check-all.sh
```

See the package-specific `CONTRIBUTING.md` files for conventions in each area.

## Troubleshooting

- **`ECONNREFUSED 127.0.0.1:5432`** – the Docker Postgres listens on `5433`; set `DB_PORT=5433`.
- **`password authentication failed`** – set `DB_PASSWORD=postgres` to match `docker-compose.yml`.
- **Port 3000 already in use** – the frontend must run on `3001` (`pnpm dev -- -p 3001`) so the backend can keep `3000`.
