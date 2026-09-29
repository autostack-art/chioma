# Quickstart

The single path from a fresh clone to a running Chioma stack (Postgres + Redis, NestJS backend, Next.js frontend, Soroban contracts). Sub-project READMEs link here instead of repeating these steps.

## 1. Prerequisites

| Tool                          | Version                  | Needed for           |
| ----------------------------- | ------------------------ | -------------------- |
| Git                           | any recent               | cloning              |
| Node.js                       | 20+                      | backend, frontend    |
| pnpm                          | 10.x (`corepack enable`) | backend, frontend    |
| Docker + Docker Compose       | recent                   | Postgres & Redis     |
| Rust + `wasm32v1-none` target | stable                   | contracts (optional) |
| Stellar CLI                   | latest                   | contracts (optional) |

## 2. Clone

```bash
git clone https://github.com/chioma-housing-protocol-I/chioma.git
cd chioma
```

## 3. Start Postgres and Redis

```bash
cd backend
docker compose up -d db redis
```

This exposes Postgres on **5433** (user `postgres`, password `postgres`, db `chioma_db`) and Redis on **6379**.

## 4. Backend (http://localhost:3000)

```bash
# still in backend/
cp .env.example .env
```

Edit `.env` so the database settings match the Docker container:

```env
DB_PORT=5433
DB_PASSWORD=postgres
```

Then install, migrate and run:

```bash
pnpm install
pnpm run migration:run
pnpm run start:dev
```

The API listens on `PORT` from `.env` (default `3000`).

## 5. Frontend (http://localhost:3001)

In a new terminal from the repo root:

```bash
cd frontend
cp .env.example .env.local   # NEXT_PUBLIC_API_URL already points at http://localhost:3000
pnpm install
pnpm dev -p 3001
```

Open http://localhost:3001. Port 3001 is used so the frontend does not clash with the backend on 3000.

## 6. Contracts (optional)

```bash
cd contract
rustup target add wasm32v1-none
cargo test
stellar contract build
```

To point the frontend at a deployed contract, set `NEXT_PUBLIC_CHIOMA_CONTRACT_ID` in `frontend/.env.local`.

## 7. Verify

- `curl http://localhost:3000` returns a response from the API.
- http://localhost:3001 loads the app.
- `docker compose ps` (in `backend/`) shows `chioma_db` and `chioma_redis` running.

## Before opening a PR

Each package has a `check-all.sh` / Makefile that mirrors CI:

```bash
cd backend && make ci      # or ./check-all.sh
cd frontend && make ci
cd contract && ./check-all.sh
```

See the per-package `CONTRIBUTING.md` for coding conventions.

## Troubleshooting

- **`ECONNREFUSED` on 5432** – `.env` still has `DB_PORT=5432`; the Docker DB is on `5433`.
- **Port 3000 in use** – run the frontend with `-p 3001` (see step 5).
- **Stop everything** – `cd backend && docker compose down` (add `-v` to wipe the database).
