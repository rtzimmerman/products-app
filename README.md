# Products App

A minimal end-to-end app: Next.js (React) frontend → Spring Boot backend → Postgres. Lists products and creates a product.

## Prerequisites

- Node.js 24 LTS or newer (`nvm use` in `frontend/` reads `.nvmrc`)
- Gradle is running on JDK 25, specifically Amazon Corretto 25.0.4.1
- Docker and Docker Compose (for Postgres)

## Database setup

From the repo root, start Postgres:

```bash
docker compose up -d db
```

This starts Postgres 16 on port 6000 with database `productsdb`, username and password have been removed (see `docker-compose.yml`).

The `products` table is created automatically when the backend starts, from `backend/src/main/resources/schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS products (
  id   BIGSERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);
```

To create it manually instead:

```bash
docker compose exec db psql -U products -d productsdb \
  -c "CREATE TABLE IF NOT EXISTS products (id BIGSERIAL PRIMARY KEY, name VARCHAR(255) NOT NULL);"
```

To reset all data: `docker compose down -v`.

## Start backend

```bash
cd backend
./gradlew bootRun
```

Runs on http://localhost:8080.

- `GET  /api/products` returns all products
- `POST /api/products` with body `{"name": "P1"}` creates a product (400 if `name` is blank)

## Start frontend

```bash
cd frontend
npm install
npm run dev
```

Runs on http://localhost:3000 and calls the backend at http://localhost:8080 (override with `NEXT_PUBLIC_API_URL`).

## Verify

1. Open http://localhost:3000
2. Create a product named `P1`
3. Confirm `P1` appears in the list (refresh to confirm it persisted)

Optional API check:

```bash
curl -X POST localhost:8080/api/products -H 'Content-Type: application/json' -d '{"name":"P1"}'
curl localhost:8080/api/products
```

## Notes

- Frontend uses Next.js (App Router, TypeScript) instead of plain React. It is React-based and runs as a client component calling the Spring Boot API directly.
- Backend allows the frontend origin (`http://localhost:3000`) via `CorsConfig`.

## How you built it

> Edit this section so it reflects what you actually did.

- **Tools:** Built with IntelliJ for Backend and VSCode for Frontend. Docker Desktop was used to run container for DB locally. Claude was also used to create some initial scaffolding and Docker Compose file, as well as template for initial documentation.
- **Approach:** Database table first, then backend endpoints (verified with curl), then the UI, checking each layer before moving on.
- **Decision:** Kept the schema to `id` and `name`. Layered packages pattern (`controller`, `model`, `repository`, `config`) to allow for future codebase growth.
- **Decision:** Docker Compose for Postgres and a pinned Node version (`.nvmrc`) so setup is repeatable for a teammate.
- **Would do differently:** Add automated tests (controller and repository tests, a frontend component test), and a CI workflow.