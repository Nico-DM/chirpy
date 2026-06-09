# Chirpy

An HTTP server built on Go.

## Build and Run

```bash
go build -o out && ./out
```

## URL

http://localhost:8080/

## Database

### Enter Toolbox

```bash
toolbox enter
```

### Start Database

```bash
pg_ctl -D ~/postgres-data -l ~/postgres.log start
```

### First Time Setup

```bash
psql -h 127.0.0.1 -d postgres
```

```sql
CREATE DATABASE chirpy;
```

### Connect to Database

```bash
psql "postgres://nico:@localhost:5432/chirpy"
```

### Run Migrations

```bash
cd sql/schema && goose postgres "postgres://nico:@localhost:5432/chirpy" up && cd ../..
```

```bash
cd sql/schema && goose postgres "postgres://nico:@localhost:5432/chirpy" down && cd ../..
```