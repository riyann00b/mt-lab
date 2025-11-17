# Evolution API Installation (Docker Compose)

Deploy Evolution API alongside its Redis and PostgreSQL dependencies using Docker Compose. This guide reflects the curated configuration under `docker-files/evolution-api/` and explains the rationale behind the non-default host port: ERPNext already occupies port **8080**, so the Evolution API is exposed on **8081** instead.

---

## Prerequisites

- Docker Engine 24.x or newer
- Docker Compose plugin 2.x (invoked as `docker compose`)
- Access to the host user-defined Docker network that ERPNext runs on (`frappe_network`). If it does not already exist, create it with `docker network create frappe_network`.
- Host firewall opened for TCP 8081 if you plan to access the API remotely

---

## Directory Layout

Organize the deployment assets as follows:

```/dev/null/layout.txt#L1-7
docker-files/
└── evolution-api/
    ├── docker-compose.yaml
    └── env/
        └── .env
```

All commands in this document assume you are inside `docker-files/evolution-api/`.

---

## Configure Environment Variables

Edit `env/.env` before starting the stack. At minimum:

- **AUTHENTICATION_API_KEY**: replace the placeholder with a strong secret.
- **POSTGRES_DATABASE / POSTGRES_USERNAME / POSTGRES_PASSWORD**: use matching values that suit your security policy.
- **SERVER_URL**: ensure it reflects the externally reachable URL. The compose file maps `127.0.0.1:8081` to the container’s internal port 8080, so `http://localhost:8081` is appropriate for local testing.
- **DATABASE_CONNECTION_URI**: confirm the credentials mirror the values above.
- Review optional integrations (RabbitMQ, Kafka, Webhooks, etc.) and toggle them as needed.

---

## Docker Compose Definition

The stack runs Evolution API, Redis, and PostgreSQL services, attaches them to the shared ERPNext network, and binds the API to port 8081 on the host:

```mt-lab/docker-files/evolution-api/docker-compose.yaml#L1-72
version: "3.8"

services:
  api:
    container_name: evolution_api
    image: evoapicloud/evolution-api:latest
    platform: linux/amd64
    restart: always
    depends_on:
      - redis
      - evolution-postgres
    ports:
      - "127.0.0.1:8081:8080"
    volumes:
      - evolution_instances:/evolution/instances
    networks:
      - evolution-net
      - frappe_network
    env_file:
      - env/.env
    expose:
      - "8080"

  redis:
    container_name: evolution_redis
    image: redis:latest
    platform: linux/amd64
    restart: always
    command: >
      redis-server --port 6379 --appendonly yes
    volumes:
      - evolution_redis:/data
    networks:
      evolution-net:
        aliases:
          - evolution-redis
      frappe_network:
        aliases:
          - evolution-redis
    expose:
      - "6379"

  evolution-postgres:
    container_name: evolution_postgres
    image: postgres:15
    platform: linux/amd64
    restart: always
    env_file:
      - env/.env
    command:
      - postgres
      - -c
      - max_connections=1000
      - -c
      - listen_addresses=*
    environment:
      - POSTGRES_DB=${POSTGRES_DATABASE}
      - POSTGRES_USER=${POSTGRES_USERNAME}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - evolution-net
      - frappe_network
    expose:
      - "5432"
    ports:
      - "5433:5432"

volumes:
  evolution_instances:
  evolution_redis:
  postgres_data:

networks:
  evolution-net:
    name: evolution-net
    driver: bridge
  frappe_network:
    external: true
```

**Key points**

- `ports: "127.0.0.1:8081:8080"` guarantees ERPNext can continue using port 8080. Adjust the host binding if you want remote access (for example `0.0.0.0:8081:8080`).
- The API container shares `env/.env` with PostgreSQL to keep credentials in sync.
- `frappe_network` is marked `external: true`; Docker Compose expects the network to exist beforehand (usually provisioned by your ERPNext deployment).
- Data persists via named volumes (`evolution_instances`, `evolution_redis`, `postgres_data`).

---

## Start the Stack

From `docker-files/evolution-api/` run:

```/dev/null/commands.sh#L1-1
docker compose up -d
```

Docker will pull images, create volumes/networks (excluding external ones), and launch the containers in detached mode.

---

## Verify the Deployment

1. Check service status:

   ```/dev/null/commands.sh#L3-3
   docker compose ps
   ```

2. Inspect logs if needed:

   ```/dev/null/commands.sh#L5-5
   docker logs evolution_api --follow
   ```

3. Confirm the health endpoint (replace `localhost` if the service is remote):

   ```/dev/null/commands.sh#L7-7
   curl http://localhost:8081/health
   ```

---

## Accessing the API

Open a browser (or REST client) and navigate to:

```/dev/null/url.txt#L1-1
http://localhost:8081
```

Remember that the port is 8081 because ERPNext already occupies 8080 on the host.

---

## Managing the Stack

- Stop services: `docker compose down`
- Restart after configuration changes: `docker compose down && docker compose up -d`
- Update images: `docker compose pull` followed by `docker compose up -d`

Persistent volumes keep data across restarts. Remove them only if you want a clean slate (`docker compose down -v`).

---

## Next Steps

- Harden the `env/.env` file (restrict filesystem permissions and rotate secrets regularly).
- Configure HTTPS fronting (reverse proxy) if exposing the API publicly.
- Enable observability integrations (Sentry, Prometheus, etc.) as required by your operations team.