# syntax=docker/dockerfile:1

# ── Stage 1: compile pg_uuidv7 ────────────────────────────────────────────
FROM postgres:17-bookworm AS builder

ARG PG_UUIDV7_TAG=v1.7.0

RUN apt-get update && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
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

COPY --from=builder /usr/lib/postgresql/17/lib/pg_uuidv7.so \
                    /usr/lib/postgresql/17/lib/pg_uuidv7.so
COPY --from=builder /usr/share/postgresql/17/extension/pg_uuidv7* \
                    /usr/share/postgresql/17/extension/

RUN echo "shared_preload_libraries = 'pg_uuidv7'" \
      >> /usr/share/postgresql/postgresql.conf.sample

LABEL org.opencontainers.image.title="postgres-uuidv7-slim"
LABEL org.opencontainers.image.description="PostgreSQL 17 with pg_uuidv7 extension only"
LABEL org.opencontainers.image.licenses="PostgreSQL"
LABEL org.opencontainers.image.vendor="Tyoho Group"
LABEL org.opencontainers.image.source="https://github.com/Tyoho-Group/docker-postgres"
