# Local Docker Compose

This repository contains a Docker Compose setup for local development. The canonical compose file is `.docker-compose.yml` — it brings up SonarQube (with its dedicated PostgreSQL), development databases, object storage emulators and caches used by projects in this workspace.

---

## Scope

This README documents the services defined in `.docker-compose.yml` and how to run them on Windows (PowerShell) or WSL.

Services included (high-level):
- `sonarqube` + `sonarqube_db` — SonarQube (Community) with Postgres
- `sqlserver` — Microsoft SQL Server (Developer)
- `postgres` — app Postgres (pgvector image)
- `redis` — Redis cache
- `minio` — S3-compatible object store (console on :9001)
- `azurite` — Azure Storage emulator

---

## Quick start (PowerShell)

Open a PowerShell prompt and run:

```powershell
cd path:\tools
docker compose up -d
```

WSL (Ubuntu / other):

```bash
cd /path/tools
docker compose up -d
```

Verify services:

```powershell
docker compose ps
# or
docker compose logs -f sonarqube
```

---

## Ports & quick map (host → container)

| Service        | Image                    | Host ports (host:container) | Notes |
|----------------|--------------------------|-----------------------------:|-------|
| SonarQube      | `sonarqube:community`    | `9000:9000`                  | UI: http://localhost:9000 |
| Sonar DB       | `postgres:15`            | `5433:5432`                  | Dedicated DB for SonarQube (host 5433) |
| App Postgres   | `pgvector/pgvector:pg16` | `5432:5432`                  | app DB with pgvector extensions |
| SQL Server     | `mcr.microsoft.com/mssql/server:latest` | `1433:1433` | SA account enabled (dev only) |
| Redis          | `redis:7-alpine`         | `6379:6379`                  | cache |
| MinIO (S3)     | `minio/minio:latest`     | `9002:9000`, `9001:9001`     | S3 API on 9002, console on 9001 |
| Azurite        | `mcr.microsoft.com/azure-storage/azurite:latest` | `10000-10002:10000-10002` | Blob/Queue/Table emulation |

> Note: `sonarqube_db` intentionally maps to host **5433** so it doesn't conflict with the app Postgres on **5432**.

---

## Persistent data (named volumes)

The compose file creates these volumes (persisted by Docker):
- `sonarqube_data`, `sonarqube_extensions`, `sonarqube_logs`, `sonarqube_postgres_data`
- `postgres_data`, `sqlserver_data`, `redis_data`, `minio_data`, `azurite_data`

Backup example (PowerShell) — export Sonar Postgres volume:

```powershell
# create a tarball of the sonarqube DB volume
docker run --rm -v sonarqube_postgres_data:/data -v ${PWD}:/backup alpine \
  tar czf /backup/sonar_pg_backup.tar.gz -C /data .
```

Restore example:

```powershell
# restore into the volume (overwrites contents)
docker run --rm -v sonarqube_postgres_data:/data -v ${PWD}:/backup alpine \
  sh -c "cd /data && tar xzf /backup/sonar_pg_backup.tar.gz"
```

---

## Default credentials (development only)

- SonarQube UI: `admin` / `admin` (unchanged by compose)
- SonarQube DB: `POSTGRES_USER=sonar` / `POSTGRES_PASSWORD=sonar`
- MinIO: `MINIO_ROOT_USER=minioadmin` / `MINIO_ROOT_PASSWORD=minioadmin`
- SQL Server SA password is set in the compose (change before sharing)

**DO NOT** use these credentials in production. Change secrets via environment variables or an env-file.

---

## Troubleshooting & Windows tips

- If SonarQube (Elasticsearch) fails to start, increase vm.max_map_count on WSL/host:

```bash
# WSL / Linux
sudo sysctl -w vm.max_map_count=262144
```

- Port conflicts: compose exposes multiple DBs — confirm nothing else is listening on `5432`, `1433`, `9000`, etc.
- Windows file sharing / permissions can slow bind-mounted volumes; use named volumes (this compose uses them).
- If you change DB images or wipe volumes, you may need to reinitialize SonarQube data (backup first).

---

## Useful commands

```powershell
# start
cd path:\tools
docker compose up -d

# stop and remove containers (keep volumes)
docker compose down

# stop and remove containers + volumes (DATA LOSS)
docker compose down -v

# pull updated images
docker compose pull

# recreate a single service (e.g., sonarqube)
docker compose up -d --force-recreate sonarqube

# view logs
docker compose logs -f sonarqube
```

---

## Upgrading SonarQube

1. Read SonarQube release notes for breaking changes.
2. Update the `image` tag for `sonarqube` in `.docker-compose.yml`.
3. Backup volumes (see Backup example).
4. Run:

```powershell
docker compose down -v
docker compose pull
docker compose up -d
```

> `-v` removes named volumes (data). Only use when you intend to recreate data.

---

## Security & production notes

This compose is intended for local development and CI. Do not deploy this stack to production as-is — secrets are in cleartext and images/config are not hardened.


---

© Local development setup — keep credentials and secrets out of commits.

