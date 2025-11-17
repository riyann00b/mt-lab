# n8n Docker Stack Overview

This folder contains the Docker resources that run an n8n instance with its queue worker, PostgreSQL, Redis, and an embedded Gotenberg service for PDF generation. It’s meant as a quick reference for how the stack is wired together, not a comprehensive installation guide.

## Components

| Service | Purpose |
| --- | --- |
| `postgres` | Primary data store for n8n |
| `redis` | Queue backend used by n8n in executions queue mode |
| `n8n` | Main web UI/API (listens on port `5678` by default) |
| `n8n-worker` | Dedicated worker processing queued executions |
| `gotenberg` | Self-contained API for HTML/Office → PDF conversions (port `3000`) |

All services run on the external Docker network named `frappe_network` so they can integrate with the rest of the ERPNext environment.

## Directory Layout

- `docker-compose.yml`: Orchestration file for the stack
- `env/`: Optional folder for environment files; copy values into `.env` before launching
- `script/init-data.sh`: Optional seed data for PostgreSQL (mounted automatically)

## Required Environment Variables

Populate the following keys in your `.env` (or export them in your shell session) before starting the stack:

- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- `POSTGRES_NON_ROOT_USER`, `POSTGRES_NON_ROOT_PASSWORD`
- `ENCRYPTION_KEY` (a random 32-byte string for n8n credential encryption)

Tip: generate secure values with `openssl rand -hex 32`.

## Working With the Stack

Run all commands from this directory unless noted otherwise.

### Start or update the services

Use `docker compose up -d` to pull images (if needed), create volumes (`db_storage`, `n8n_storage`, `redis_storage`), attach to `frappe_network`, and launch every container.

### Stop the services

Run `docker compose stop` to halt the stack without removing containers.

### Check container status

Execute `docker compose ps` to view health checks and port bindings.

### Tail service logs

Use `docker compose logs -f n8n` for the main application and `docker compose logs -f n8n-worker` for the queue consumer.

### Access points

- n8n UI: `http://localhost:5678` (adjust if you rebind ports)
- Gotenberg API health: `http://localhost:3000/health`

For added safety, bind exposed ports to `127.0.0.1` in `docker-compose.yml` if you do not need LAN access.

## Data Persistence and Backups

- PostgreSQL data lives in the `db_storage` volume.
- n8n workflows and credentials reside in `n8n_storage`.
- Redis queues are stored in `redis_storage`; prune it during maintenance windows if you want a clean slate.

Leverage `pg_dump` or other PostgreSQL tooling to back up critical data, and snapshot the `n8n_storage` volume before major upgrades.

## Updating the Stack

1. Run `docker compose pull` to fetch the latest images.
2. Apply updates with `docker compose up -d`.
3. Review `docker compose logs` for migration output or warnings.

## Troubleshooting Checklist

- **Containers restart continuously:** Verify `.env` values are set and align with `script/init-data.sh`.
- **Worker reports connection errors:** Ensure Redis and Postgres show `healthy` status in `docker compose ps`; the worker exits if dependencies remain unavailable.
- **Gotenberg unreachable from workflows:** Confirm both services share `frappe_network` and reference the internal URL `http://gotenberg:3000`.
- **Host port conflict:** Adjust the `ports` mapping in `docker-compose.yml` or bind to loopback only.

## Security Notes

- Treat Gotenberg like a database: keep it behind firewalls or bind to localhost.
- Rotate the n8n encryption key and database passwords periodically.
- Place an authenticating proxy (Traefik, Caddy, Nginx, etc.) in front of the n8n UI if exposing it beyond trusted networks.

## Additional References

- [n8n Docker documentation](https://docs.n8n.io/hosting/)
- [Gotenberg official docs](https://gotenberg.dev/docs/getting-started/introduction)
- Upstream PostgreSQL and Redis guides for performance tuning, monitoring, and backup strategies
