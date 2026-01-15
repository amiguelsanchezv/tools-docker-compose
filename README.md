# SonarQube Docker Setup

A Docker Compose configuration for running SonarQube with PostgreSQL.

## Prerequisites

- Docker and Docker Compose installed
- WSL2 (if running on Windows)

## Quick Start

### From WSL

```bash
cd /mnt/c/sonarqube
docker compose up -d
```

### From PowerShell

```powershell
cd C:\sonarqube
docker compose up -d
```

## Access

- **URL**: http://localhost:9000
- **Default credentials**: `admin` / `admin`

## Services

| Service   | Image                  | Port |
|-----------|------------------------|------|
| SonarQube | sonarqube:community    | 9000 |
| PostgreSQL| postgres:15            | 5432 (internal) |

## Upgrading SonarQube

To upgrade to a newer version:

1. Update the image tag in `docker-compose.yml` (or use `community` tag for latest)
2. Run the following commands:

```bash
docker compose down -v
docker compose pull
docker compose up -d
```

> **Warning**: Using `-v` removes all volumes including data. If upgrading between major versions, Elasticsearch indexes may be incompatible and require a clean install.

> **Note**: Always check the [SonarQube release notes](https://www.sonarsource.com/products/sonarqube/downloads/) for breaking changes before upgrading.

## Useful Commands

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f sonarqube

# Restart services
docker compose restart
```

## Troubleshooting

If SonarQube fails to start, ensure your system has enough virtual memory:

```bash
# On WSL/Linux
sudo sysctl -w vm.max_map_count=262144
```

To make it permanent, add to `/etc/sysctl.conf`:

```
vm.max_map_count=262144
```
