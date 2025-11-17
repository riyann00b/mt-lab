# Gotenberg Installation Guide

> The `docker-files/n8n/docker-compose.yml` stack already provisions a shared `gotenberg` service for n8n workflows. Because Gotenberg is a self-contained HTTP API, no database or additional dependency layers are required. You can keep using the bundled service or deploy a standalone instance if you have other consumers that need dedicated capacity.

## What is Gotenberg?

Gotenberg is an open-source, containerized API for converting a wide range of document formats into PDFs—and, in some cases, images—via multipurpose HTTP endpoints. It ships with everything it needs (Chromium, LibreOffice, PDF tooling), supports AMD64, ARM64, ARMHF, i386, and PPC64LE architectures, and scales horizontally by simply running more containers.

Key characteristics pulled from the official documentation:

- **Versatile conversion endpoints** backed by Chromium, LibreOffice, and PDF utilities.
- **Containerized & self-contained** delivery with zero external dependencies.
- **HTTP/2 ready** with support for both classic TLS and H2C.
- **Modular design** so each feature (Chromium, LibreOffice, Webhooks, Prometheus metrics, etc.) is independently configurable.
- **Open source** and free to integrate into your platform.

> ⚠️ Treat a Gotenberg instance like you would a database: do not expose it directly to the public internet without proper security controls (firewalls, proxies, auth).

## Using the bundled n8n stack (recommended)

1. **Prepare environment variables**  
   Populate the `.env` file that the n8n compose stack expects (`POSTGRES_*`, `ENCRYPTION_KEY`, etc.). Gotenberg itself does not need any of these variables, but the n8n services do.

2. **Create the shared Docker network**  
   ```docs/gothenburg/Installation.md#L26-26
   docker network create frappe_network
   ```
   (Skip this if the network already exists.)

3. **Start or update the stack**  
   ```docs/gothenburg/Installation.md#L32-32
   docker compose -f docker-files/n8n/docker-compose.yml up -d
   ```
   This brings up Postgres, Redis, the main n8n service, its worker, and the `gotenberg` container listening on port 3000 inside the compose network. No extra configuration is required.

4. **Verify Gotenberg health**  
   - Inside the Docker network, services reach it at `http://gotenberg:3000`.
   - From the host, if the compose file publishes the port, curl `http://localhost:3000/health`. A JSON payload with `"status": "up"` confirms it’s ready.

5. **Consume from n8n**  
   Your workflows can call `http://gotenberg:3000/forms/...` endpoints via the HTTP Request node. Because Gotenberg is already in the stack, there is nothing else to install.

## Optional standalone deployment

If you need an independent Gotenberg instance (for example, for another application or to isolate workloads), launch one manually:

```docs/gothenburg/Installation.md#L48-48
docker run --rm -p "127.0.0.1:3000:3000" gotenberg/gotenberg:8
```

This mirrors the official quick-start command and keeps the service bound to localhost for safety. Within Docker Compose, you can add:

```docs/gothenburg/Installation.md#L54-58
services:
  gotenberg:
    image: gotenberg/gotenberg:8
    ports:
      - "127.0.0.1:3000:3000"
```

Once running, the API surface matches the documentation (e.g., `/forms/chromium/convert/url`, `/forms/libreoffice/convert`, `/forms/pdfengines/*`, `/health`, `/prometheus/metrics`, `/version`, `/debug`).

## Common commands (from official docs)

- **Live demo for testing**: `https://demo.gotenberg.dev` (rate-limited, 5 MB body limit, 2 rps per IP).
- **Quick conversion example**:
  ```docs/gothenburg/Installation.md#L68-71
  curl \
    --request POST http://localhost:3000/forms/chromium/convert/url \
    --form url=https://sparksuite.github.io/simple-html-invoice-template/ \
    -o invoice.pdf
  ```
- **Health check**: `curl http://localhost:3000/health`
- **Version**: `curl http://localhost:3000/version`

## Configuration highlights

You generally configure Gotenberg through command-line flags or matching environment variables. Examples:

- API: `--api-port`, `--api-timeout`, `--api-body-limit`
- Chromium: `--chromium-ignore-certificate-errors`, `--chromium-disable-javascript`, `--chromium-max-queue-size`
- LibreOffice: `--libreoffice-disable-routes`, `--libreoffice-restart-after`
- Webhook: `--webhook-allow-list`, `--webhook-max-retry`
- Prometheus: `--prometheus-collect-interval`, `--prometheus-namespace`
- Logging: `--log-format`, `--log-level`

Example override in Compose:

```docs/gothenburg/Installation.md#L90-95
services:
  gotenberg:
    image: gotenberg/gotenberg:8
    environment:
      API_TIMEOUT: "60s"
      CHROMIUM_MAX_QUEUE_SIZE: "10"
```

Remember to override the container command (not the entrypoint) if you pass flags directly.

## Best practices & security

- Bind published ports to `127.0.0.1` (or use a reverse proxy) to avoid exposing the service broadly.
- Add authentication or network-layer protections if other teams or services access the API.
- Scale horizontally—run multiple instances—when you face heavy parallel workloads.
- Monitor `/prometheus/metrics` if you integrate with Prometheus.
- Keep fonts and locales updated by extending the official image when needed.

## Summary

- **No database** or auxiliary services are required for Gotenberg.
- The repository’s n8n compose stack **already includes** a ready-to-use Gotenberg container.
- Standalone deployment is a single Docker command away if you want it.
- Use the official endpoints and configuration flags for advanced behavior.

With this alignment to the official documentation, you can confidently run Gotenberg alongside n8n or in standalone mode and take advantage of its powerful PDF conversion capabilities.
