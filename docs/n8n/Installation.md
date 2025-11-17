# n8n Installation Guide (Docker Compose Stack)

## Overview
This document walks you through deploying the bundled n8n stack (web UI, worker, PostgreSQL, Redis, and Gotenberg) using the provided `docker-compose.yml` located in `docker-files/n8n/`. Follow these steps if you are comfortable managing Docker-based services.

## Architecture Summary
The stack follows the official [n8n Docker installation guidance](https://docs.n8n.io/hosting/installation/docker/) and extends it with production-ready components:
- PostgreSQL 16 as the primary database
- Redis 6 for queue-backed execution mode
- Dedicated n8n worker container
- Gotenberg 8 for document-to-PDF conversions
- Persistent Docker volumes for stateful data
- Shared external Docker network `frappe_network` to integrate with other ERPNext services

## Prerequisites
Make sure you have the following before you begin:
- Docker Engine 20.10+ and Docker Compose plugin v2.0+ installed on the host
- Access to the project repository on the host machine
- Ability to create Docker networks on the host (for `frappe_network`)
- Shell access with permissions to read and edit files under `mt-lab/docker-files/n8n/`
- (Optional) An outbound internet connection for pulling container images

## Directory Layout
All referenced paths are relative to the repository root.
- `docker-files/n8n/docker-compose.yml` – Docker Compose definition of the stack
- `docker-files/n8n/env/` – Staging area for environment variable files
- `docker-files/n8n/script/init-data.sh` – PostgreSQL bootstrap script that seeds a non-root user
- `docs/n8n/Installation.md` – This installation guide

## Step 1: Ensure the External Network Exists
The Compose file expects an external Docker network called `frappe_network`. Create it once per Docker host:
```/dev/null/n8n-network.sh#L1-1
docker network create frappe_network
```
Skip this command if the network already exists (`docker network ls` to verify).

## Step 2: Prepare Environment Variables
Compose relies on a `.env` file to inject secrets and configuration. Copy the template below into `docker-files/n8n/.env` and adjust every placeholder to strong, unique values:
```/dev/null/n8n.env#L1-11
POSTGRES_USER=postgres
POSTGRES_PASSWORD=change-me
POSTGRES_DB=n8n
POSTGRES_NON_ROOT_USER=n8n_user
POSTGRES_NON_ROOT_PASSWORD=change-me-too
ENCRYPTION_KEY=run-openssl-rand-hex-32
GENERIC_TIMEZONE=Etc/UTC
TZ=Etc/UTC
QUEUE_BULL_REDIS_HOST=redis
QUEUE_HEALTH_CHECK_ACTIVE=true
```
Guidance:
- Generate strong passwords: `openssl rand -hex 32`
- `ENCRYPTION_KEY` must remain stable; rotating it will invalidate stored credentials
- Timezone variables keep scheduled workflows aligned with your region

## Step 3: Review `init-data.sh`
The PostgreSQL initialization script creates the non-root database user specified above during the first startup. Ensure the usernames in `.env` match the values referenced in the script (`POSTGRES_NON_ROOT_USER` and `POSTGRES_NON_ROOT_PASSWORD`).

## Step 4: Start the Stack
Change into the Compose directory and launch all services in detached mode:
```/dev/null/n8n-start.sh#L1-3
cd docker-files/n8n
docker compose pull
docker compose up -d
```
- `docker compose pull` ensures you are using the latest image tags pinned in the compose file
- `docker compose up -d` creates the volumes `db_storage`, `n8n_storage`, and `redis_storage`, attaches every container to `frappe_network`, and starts services with health checks enabled

## Step 5: Verify Service Health
Use the following commands to confirm everything is running:
```/dev/null/n8n-verify.sh#L1-3
docker compose ps
docker compose logs -f postgres
docker compose logs -f n8n
```
- Wait for the PostgreSQL and Redis health checks to report `healthy`
- Ensure the main `n8n` service displays “Editor is now accessible” messages
- Stop tailing logs with `Ctrl+C` after inspection

## Step 6: Access the Application
Once containers are healthy:
- Open `http://localhost:5678` in your browser (adjust the host/port as needed)
- Create your initial n8n user account when prompted
- Configure additional settings (e.g., SMTP, OAuth) through the Settings panel

## Queue Worker Behavior
This stack runs n8n in queue mode (`EXECUTIONS_MODE=queue`). The `n8n-worker` container:
- Depends on Redis and PostgreSQL health before starting
- Processes jobs dispatched by the main `n8n` container
- Shares the same image and environment settings through the Compose `x-shared` anchor

## Gotenberg Integration
For PDF conversions in workflows:
- Target the internal URL `http://gotenberg:3000` from n8n HTTP nodes
- The service publishes port `3000` to the host; restrict exposure by rebinding to `127.0.0.1:3000` in `docker-compose.yml` if external access is unnecessary

## Data Persistence and Backups
Persistent data lives in Docker volumes:
- `db_storage` – PostgreSQL cluster files
- `n8n_storage` – Workflow definitions, credential store, logs, and source control data
- `redis_storage` – Redis queue snapshots (safe to prune during maintenance)
Recommended backup workflow:
1. Pause external traffic to n8n (maintenance window)
2. Run `pg_dump` or volume snapshots for `db_storage`
3. Archive `n8n_storage` (e.g., `docker run --rm -v n8n_storage:/data busybox tar czf /backup/n8n_storage.tar.gz /data`)
4. Optionally flush `redis_storage` if you want to clear queued jobs

## Maintenance and Upgrades
1. Pull new images: `docker compose pull`
2. Apply updates: `docker compose up -d`
3. Review logs for database migrations or warnings
4. Remove unused images: `docker image prune`
Pin specific n8n image tags (`docker.n8n.io/n8nio/n8n:<version>`) if you require deterministic updates.

## Stopping and Restarting
- Stop services without removing containers: `docker compose stop`
- Restart everything with the existing state: `docker compose start`
- Tear down containers but keep volumes: `docker compose down`
- Remove all data (irreversible): `docker compose down -v`

## Security Considerations
- Bind exposed ports to `127.0.0.1` or secure them behind reverse proxies
- Rotate database passwords and the n8n encryption key periodically (update `.env` and restart)
- Enforce TLS and authentication if publishing the n8n UI beyond trusted networks
- Limit shell access to the host and treat backups as sensitive data

## Troubleshooting
| Symptom | Likely Cause | Resolution |
| --- | --- | --- |
| Containers restart repeatedly | Missing or mismatched environment variables | Revisit `.env` and ensure `POSTGRES_*` values align with `init-data.sh` |
| `n8n-worker` exits | Redis or PostgreSQL health checks failing | Inspect with `docker compose ps` and logs; restart once dependencies are healthy |
| Cannot reach n8n UI | Ports blocked or bound to loopback | Confirm `docker-compose.yml` port mappings or adjust firewall rules |
| Gotenberg node errors | Using host URL inside workflows | Use `http://gotenberg:3000` from within n8n |
| Database authentication errors | Non-root user not created | Remove `db_storage` volume (if safe) so Postgres reruns `init-data.sh`, then relaunch |

## References
- Official docs: [n8n Docker installation](https://docs.n8n.io/hosting/installation/docker/)
- Stack README: `docker-files/n8n/README.md`
- Gotenberg docs: [https://gotenberg.dev/docs/getting-started/introduction](https://gotenberg.dev/docs/getting-started/introduction)

## Change Log
- 2025-XX-XX: Initial publication of comprehensive installation procedure