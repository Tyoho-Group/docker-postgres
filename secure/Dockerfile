# syntax=docker/dockerfile:1

# ── Stage 1: compile pg_uuidv7 ────────────────────────────────────────────
FROM postgres:17-bookworm AS builder

# Pin to a specific release tag for reproducibility.
# Check latest: https://github.com/fboulnois/pg_uuidv7/releases
ARG PG_UUIDV7_TAG=v1.7.0

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    git \
    gcc \
    make \
    postgresql-server-dev-17 \
    clang \
    llvm \
    && rm -rf /var/lib/apt/lists/*

RUN git clone --depth 1 --branch "${PG_UUIDV7_TAG}" \
        https://github.com/fboulnois/pg_uuidv7.git /tmp/pg_uuidv7 \
    && cd /tmp/pg_uuidv7 \
    && make \
    && make install \
    && rm -rf /tmp/pg_uuidv7

# ── Stage 2: final image (no build tools) ─────────────────────────────────
FROM postgres:17-bookworm

ARG PG_UUIDV7_TAG=v1.7.0

# Copy only the compiled extension files from the builder stage
COPY --from=builder /usr/lib/postgresql/17/lib/pg_uuidv7.so \
                    /usr/lib/postgresql/17/lib/pg_uuidv7.so
COPY --from=builder /usr/share/postgresql/17/extension/pg_uuidv7* \
                    /usr/share/postgresql/17/extension/

# Append to the sample template that initdb copies to postgresql.conf on
# first container start.  The base image only has the setting commented out,
# so a plain append is safe.  PostgreSQL also honours the last occurrence of
# a duplicate setting, so this stays correct even if a future base image
# uncomments the line.
RUN echo "shared_preload_libraries = 'pg_uuidv7'" \
      >> /usr/share/postgresql/postgresql.conf.sample

# Security init script: runs once on first container start to disable the
# default postgres superuser and grant the app role appropriate privileges.
# Set DISABLE_POSTGRES_SUPERUSER=false to skip (e.g. for local dev).
RUN mkdir -p /docker-entrypoint-initdb.d
COPY --chmod=755 <<'EOF' /docker-entrypoint-initdb.d/01-security.sh
#!/bin/bash
set -euo pipefail

DISABLE_POSTGRES_USER="${DISABLE_POSTGRES_SUPERUSER:-true}"

if [ "$DISABLE_POSTGRES_USER" = "true" ]; then
    echo "Applying security settings: disabling postgres superuser login..."

    if psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" \
            -tAc "SELECT 1 FROM pg_roles WHERE rolname='postgres'" | grep -q 1; then
        echo "Role 'postgres' found — disabling login and setting connection limit to 0"
        psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" \
            -c "ALTER ROLE postgres WITH CONNECTION LIMIT 0 NOLOGIN;"
    else
        echo "Role 'postgres' not found — skipping"
    fi

    echo "Ensuring application role '$POSTGRES_USER' has appropriate privileges"
    psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" \
        -c "ALTER ROLE \"$POSTGRES_USER\" WITH CREATEROLE CREATEDB REPLICATION;"

    echo "PostgreSQL security initialization completed"
else
    echo "Security mode disabled: postgres superuser remains active"
fi
EOF

LABEL org.opencontainers.image.title="postgres-uuidv7-secure"
LABEL org.opencontainers.image.description="PostgreSQL 17 with pg_uuidv7 ${PG_UUIDV7_TAG} and startup hardening"
LABEL org.opencontainers.image.licenses="PostgreSQL"
LABEL org.opencontainers.image.vendor="Tyoho Group"
LABEL org.opencontainers.image.source="https://github.com/Tyoho-Group/docker-postgres"
