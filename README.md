# Docker Compose Collection

A collection of Docker Compose files for running common infrastructure services locally.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) (included with Docker Desktop)

---

## Services

### ChromaDB

Vector database for AI/ML applications. Runs on port `8000`.

```bash
docker compose -f chromadb/server-docker-compose.yml up -d
```

| Port  | Usage          |
|-------|----------------|
| 8000  | HTTP API       |

- Persistent data stored in `chromadb/chroma_data/`

---

### Floci

Local AI inference server. Runs on port `4566`.

```bash
docker compose -f floci/docker-compose.yml up -d
```

| Port  | Usage              |
|-------|--------------------|
| 4566  | API server         |

- Mounts Docker socket for container management
- Data stored in `floci/data/`

---

### MongoDB

Document database. Runs on port `27017`.

```bash
docker compose -f mongo/docker-compose.yml up -d
```

| Port   | Usage      |
|--------|------------|
| 27017  | Database   |

| Variable                    | Default |
|-----------------------------|---------|
| `MONGO_INITDB_ROOT_USERNAME` | admin   |
| `MONGO_INITDB_ROOT_PASSWORD` | admin   |

- Data stored in `mongo/mongodb_data/`

---

### n8n

Workflow automation tool. Runs on port `5678`.

```bash
# 1. Configure timezone
cp n8n/.env_example n8n/.env

# 2. Start
docker compose -f n8n/docker-compose.yml up -d
```

| Port  | Usage      |
|-------|------------|
| 5678  | Web UI     |

| Variable              | Default        |
|-----------------------|----------------|
| `GENERIC_TIMEZONE`    | Asia/Colombo   |

- Workflows stored in `n8n/n8n_data/`
- Exposes a webhook URL for external integrations

---

### NATS

Cloud-native messaging system with JetStream. Ports `4222`, `8222`, `6222`.

```bash
docker compose -f nats/docker-compose.yml up -d
```

| Port  | Usage              |
|-------|--------------------|
| 4222  | Client connections |
| 8222  | HTTP monitoring    |
| 6222  | Clustering         |

- JetStream enabled (`-js` flag)

---

### Apache Pulsar

Pub/sub messaging system with Pulsar Manager UI.

```bash
docker compose -f pulsar/docker-compose.yml up -d
```

| Port  | Usage                    |
|-------|--------------------------|
| 6650  | Broker                   |
| 8080  | Admin REST API           |
| 9527  | Pulsar Manager Web UI    |

- Runs in standalone mode
- Includes Pulsar Manager for web-based management

---

### Redis

Several deployment modes available.

#### Standalone

```bash
# 1. Set password
cp redis/.env_example redis/.env
# Edit redis/.env with your password

# 2. Start
docker compose -f redis/docker-compose.yml up -d
```

| Port  | Usage       |
|-------|-------------|
| 6379  | Redis       |

- AOF persistence enabled
- Password-protected

#### Replica (Master-Replica)

```bash
docker compose -f redis/replica-docker-compose.yml up -d
```

| Port  | Usage          |
|-------|----------------|
| 6379  | Master         |
| 6479  | Replica 1      |

#### Sentinel (High Availability)

1 master, 2 replicas, 3 sentinels, and RedisInsight UI.

```bash
docker compose -f redis/sentinel-docker-compose.yml up -d
```

| Port   | Usage            |
|--------|------------------|
| 6379   | Master           |
| 6380   | Replica 1        |
| 6381   | Replica 2        |
| 26379  | Sentinel 1       |
| 26380  | Sentinel 2       |
| 26381  | Sentinel 3       |
| 5540   | RedisInsight UI  |

**Test failover:**

```bash
# Check who is master
docker exec sentinel-1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

# Simulate master failure
docker stop redis-master

# After ~10s, check the new master
docker exec sentinel-1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

# Bring old master back — it rejoins as a replica
docker start redis-master
docker exec redis-master redis-cli INFO replication
```

---

## Useful Commands

```bash
# Start all services
docker compose -f <path>/docker-compose.yml up -d

# View logs
docker compose -f <path>/docker-compose.yml logs -f

# Stop services
docker compose -f <path>/docker-compose.yml down

# Stop and remove volumes (data will be lost!)
docker compose -f <path>/docker-compose.yml down -v

# Rebuild containers
docker compose -f <path>/docker-compose.yml up -d --build
```
