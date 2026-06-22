# Chirpy

A Twitter-style JSON API built in Go with the standard library. Chirpy is the capstone project from [Boot.dev's Learn HTTP Servers in Go](https://www.boot.dev/courses/learn-http-servers-golang) course — routing, middleware, PostgreSQL, JWT authentication, authorization, and webhooks, all without a web framework.

Supported by [Boot.dev](https://boot.dev). If you'd like to learn about HTTP servers and Go, you can check out [the course here](https://www.boot.dev/courses/learn-http-servers-golang).

## Motivation

Go's `net/http` package is powerful enough to build production APIs on its own, but most tutorials stop at "Hello, World." Chirpy walks through the full stack of backend concerns: structured JSON handlers, database access, password hashing, token-based auth, and per-resource authorization rules.

### Goal

The goal with Chirpy is to practice building a real JSON API from scratch using minimal dependencies. In particular, the project covers:

* HTTP routing with Go 1.22+ method and path patterns
* Middleware for metrics and static file serving
* PostgreSQL with [goose](https://github.com/pressly/goose) migrations and [sqlc](https://sqlc.dev/) type-safe queries
* JWT access tokens and refresh-token sessions
* Authorization rules (users can only modify their own chirps and profile)
* Webhook handlers secured with API keys

## ⚙️ Installation

Clone the repo and install dependencies:

```bash
git clone https://github.com/Nico-DM/chirpy.git
cd chirpy
go mod download
```

You'll also need PostgreSQL, [goose](https://github.com/pressly/goose), and [sqlc](https://docs.sqlc.dev/en/latest/overview/install.html) for database work.

## ⚙️ Configuration

Create a `.env` file in the project root (loaded automatically at startup):

```bash
DB_URL=postgres://user:password@localhost:5432/chirpy
JWT_SECRET=your-jwt-secret
POLKA_KEY=your-polka-webhook-api-key
PLATFORM=dev
```

| Variable     | Description |
| ------------ | ----------- |
| `DB_URL`     | PostgreSQL connection string |
| `JWT_SECRET` | Secret used to sign and validate access JWTs |
| `POLKA_KEY`  | API key expected by the Polka webhook handler |
| `PLATFORM`   | Set to `dev` to enable the `/admin/reset` endpoint |

## 🗄️ Database Setup

### Start PostgreSQL

```bash
pg_ctl -D ~/postgres-data -l ~/postgres.log start
```

### First-Time Setup

```bash
psql -h 127.0.0.1 -d postgres
```

```sql
CREATE DATABASE chirpy;
```

### Run Migrations

```bash
cd sql/schema && goose postgres "$DB_URL" up && cd ../..
```

To roll back:

```bash
cd sql/schema && goose postgres "$DB_URL" down && cd ../..
```

### Generate sqlc Code

After changing SQL queries or schema:

```bash
sqlc generate
```

### Stop PostgreSQL

```bash
pg_ctl -D ~/postgres-data stop
```

## 🚀 Quick Start

Build and run the server:

```bash
go build -o out && ./out
```

The server listens on [http://localhost:8080/](http://localhost:8080/).

Verify it's up:

```bash
curl http://localhost:8080/api/healthz
# OK
```

Create a user and log in:

```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"secret123"}'

curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"secret123"}'
```

Post a chirp (replace `$TOKEN` with the access token from login):

```bash
curl -X POST http://localhost:8080/api/chirps \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"body":"hello chirpy"}'
```

## API Reference

### Health & Static Files

| Method | Path           | Auth | Description |
| ------ | -------------- | ---- | ----------- |
| GET    | `/api/healthz` | —    | Readiness check |
| GET    | `/app/*`       | —    | Static file server (tracks visit count) |

### Users & Auth

| Method | Path            | Auth            | Description |
| ------ | --------------- | --------------- | ----------- |
| POST   | `/api/users`    | —               | Create a user |
| PUT    | `/api/users`    | Bearer JWT      | Update own email and password |
| POST   | `/api/login`    | —               | Log in; returns access and refresh tokens |
| POST   | `/api/refresh`  | Bearer refresh  | Issue a new access token |
| POST   | `/api/revoke`   | Bearer refresh  | Revoke a refresh token |

### Chirps

| Method | Path                      | Auth       | Description |
| ------ | ------------------------- | ---------- | ----------- |
| POST   | `/api/chirps`             | Bearer JWT | Create a chirp (max 140 chars; profanity filtered) |
| GET    | `/api/chirps`             | —          | List chirps; optional `?author_id=` and `?sort=asc\|desc` |
| GET    | `/api/chirps/{chirpID}`   | —          | Get a single chirp |
| DELETE | `/api/chirps/{chirpID}`   | Bearer JWT | Delete own chirp |

### Webhooks & Admin

| Method | Path                    | Auth          | Description |
| ------ | ----------------------- | ------------- | ----------- |
| POST   | `/api/polka/webhooks`   | ApiKey header | Handle Polka upgrade events (`user.upgraded`) |
| GET    | `/admin/metrics`        | —             | HTML page showing `/app/` visit count |
| POST   | `/admin/reset`          | —             | Reset DB and metrics (`PLATFORM=dev` only) |

## Project Layout

```
.
├── main.go                 # Server setup and route registration
├── handler_*.go            # HTTP handlers
├── internal/
│   ├── auth/               # JWT, password hashing, token helpers
│   └── database/           # sqlc-generated query layer
└── sql/
    ├── schema/             # goose migrations
    └── queries/            # sqlc query definitions
```

## Testing

Run unit tests:

```bash
go test ./...
```

Auth helpers have dedicated tests in `internal/auth/auth_test.go`.

## Dependencies

| Package | Purpose |
| ------- | ------- |
| [lib/pq](https://github.com/lib/pq) | PostgreSQL driver |
| [golang-jwt/jwt](https://github.com/golang-jwt/jwt) | JWT signing and validation |
| [alexedwards/argon2id](https://github.com/alexedwards/argon2id) | Password hashing |
| [joho/godotenv](https://github.com/joho/godotenv) | `.env` file loading |
| [google/uuid](https://github.com/google/uuid) | UUID generation |

## 👏 Contributing

This is a personal learning project, but suggestions and pull requests are welcome. Please ensure tests pass before submitting:

```bash
go test ./...
```
