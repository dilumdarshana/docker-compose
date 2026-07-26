# AGENTS.md

## What this is

A flat collection of standalone Docker Compose files for local infrastructure services. No build system, no tests, no package manager.

## Starting a service

Every service is launched from the repo root with `-f`:

```bash
docker compose -f <service>/docker-compose.yml up -d
```

## .env setup required before first start

**n8n** — copy `n8n/.env_example` to `n8n/.env`, then edit timezone.
**PostgreSQL** — copy `postgres/.env_example` to `postgres/.env`, then edit credentials.
**Redis standalone** — copy `redis/.env_example` to `redis/.env`, then edit password.

The above will fail to start if `.env` is missing.

## Services at a glance

| Service     | Compose file                         | Port(s)                           | Data persistence        |
|-------------|--------------------------------------|-----------------------------------|-------------------------|
| ChromaDB    | `chromadb/server-docker-compose.yml` | 8000                              | `chromadb/chroma_data/` (bind) |
| Floci       | `floci/docker-compose.yml`           | 4566                              | `floci/data/` (bind)    |
| MongoDB     | `mongo/docker-compose.yml`           | 27017                             | `mongo/mongodb_data/` (bind) |
| PostgreSQL  | `postgres/docker-compose.yml`        | 5432                              | `postgres/postgres_data/` (bind) |
| n8n         | `n8n/docker-compose.yml`             | 5678                              | `n8n/n8n_data/` (bind)  |
| NATS        | `nats/docker-compose.yml`            | 4222, 8222, 6222                  | named volume            |
| Pulsar      | `pulsar/docker-compose.yml`          | 6650, 8080, 9527                  | named volumes           |
| Redis solo  | `redis/docker-compose.yml`           | 6379                              | named volume            |
| Redis pair  | `redis/replica-docker-compose.yml`   | 6379, 6479                        | named volumes           |
| Redis HA    | `redis/sentinel-docker-compose.yml`  | 6379-6381, 26379-26381, 5540      | `redis/data/` (bind)    |

## Notable quirks

- **Redis Sentinel** — hardcoded static IPs on `172.21.0.0/24`. The sentinel config is generated dynamically via shell commands in each container's `command:`.
- **Redis Cluster** — `redis/cluster-docker-compose.yml` is empty (not implemented).
- **Chromadb** — CORS allow origins set to `['*']` (permissive; tighten for production).
- **Floci** — mounts the host Docker socket (`/var/run/docker.sock`).
- **n8n** — contains hardcoded ngrok URLs in env vars (`N8N_EDITOR_BASE_URL`, `WEBHOOK_URL`). Replace these for your own tunnel.
- **Pulsar** — runs in standalone mode with a healthcheck. Includes Pulsar Manager on port 9527.
- **NATS** — JetStream enabled via `-js` CLI flag.

## Filesystem layout

```
service-name/
├── docker-compose.yml          # main compose file
├── .env_example                # copy to .env before starting (n8n, redis only)
├── .gitignore                  # local: redis/data; root: .env, n8n_data, mongodb_data, postgres_data
└── data/                       # persistent data (bind mounts) — gitignored
```

## Convention

When adding or updating a service, update both the compose file and `README.md` (services section, ports, env vars, data paths) in the same commit.

## Useful commands

```bash
# Logs (any service)
docker compose -f <path>/docker-compose.yml logs -f

# Stop + clean volumes (destroys data)
docker compose -f <path>/docker-compose.yml down -v

# Rebuild
docker compose -f <path>/docker-compose.yml up -d --build
```
