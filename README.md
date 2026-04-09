# docker-postgres

Public PostgreSQL images with [pg_uuidv7](https://github.com/fboulnois/pg_uuidv7) extension, maintained by Tyoho Group.

## Images

All images are published exclusively to GitHub Container Registry (GHCR).

| Variant | Description | Image |
|---------|-------------|-------|
| **secure** | pg_uuidv7 + startup hardening (disables default superuser) | `ghcr.io/tyoho-group/postgres-uuidv7-secure` |
| **slim** | pg_uuidv7 only | `ghcr.io/tyoho-group/postgres-uuidv7-slim` |

## Quick start

```bash
# Secure variant (recommended for production)
docker run -d \
  -e POSTGRES_USER=myapp \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  ghcr.io/tyoho-group/postgres-uuidv7-secure:latest

# Slim variant (just the extension)
docker run -d \
  -e POSTGRES_USER=myapp \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  ghcr.io/tyoho-group/postgres-uuidv7-slim:latest
```

Then inside the database:

```sql
CREATE EXTENSION IF NOT EXISTS pg_uuidv7;
SELECT uuid_generate_v7();
```

## Secure variant details

On first container start, the `secure` image runs an init script that:

1. Disables the default `postgres` superuser (sets `NOLOGIN` + `CONNECTION LIMIT 0`)
2. Grants the application role (`POSTGRES_USER`) `CREATEROLE`, `CREATEDB`, and `REPLICATION` privileges

To disable this behavior (e.g. for local development):

```bash
docker run -e DISABLE_POSTGRES_SUPERUSER=false ...
```

## Build schedule

Images are rebuilt automatically every Monday at 02:15 UTC via GitHub Actions, and on every push to `main` that changes a Dockerfile.

## Tags

- `latest` — most recent build
- `YYYYMMDD-HHMM` — timestamp of the build
- `sha-<7chars>` — short commit SHA

## Base image

- `postgres:17-bookworm`
- `pg_uuidv7` version: `v1.7.0`

## License

The Dockerfiles in this repository are provided under the [PostgreSQL License](https://opensource.org/licenses/PostgreSQL). PostgreSQL and pg_uuidv7 retain their respective licenses.
