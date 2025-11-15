# Docker Files

This directory contains the Docker build and runtime assets that power MT Labs’ self-hosted services. Each service keeps its Containerfiles, Compose bundles, environment samples, and helper scripts in a dedicated subfolder so builds and deployments stay isolated and reproducible.

## Current Layout

```docker-files/tree.txt#L1-12
docker-files/
├── README.md
├── erpnext/
├── evolution-api/
│   └── env/
├── gothenburg/
├── n8n/
│   └── env/
└── paperless/
    └── env/
```

- `erpnext/` — Custom ERPNext image definitions, layered build arguments, and stack overrides.
- `evolution-api/` — WhatsApp bridge Compose bundles plus environment templates.
- `gothenburg/` — Document-rendering service Docker assets.
- `n8n/` — Automation orchestrator runtime definitions and environment samples.
- `paperless/` — Paperless-ngx deployment configuration and secrets templates.
- `*/env/` — Example environment files (`.env.example`) and credential templates kept out of version control.

## Conventions

- Folder names are lowercase with hyphens for consistency across documentation and scripts.
- Compose files use the same service naming found in the production stack (`erpnext`, `evolution-api`, etc.) to simplify overrides.
- Environment templates should never include real secrets—only keys and descriptive comments.
- When a service needs custom entrypoint or helper scripts, keep them beside the Compose bundle in the same subfolder.

## Typical Workflow

1. Copy the relevant `env/` template and fill in real values outside the repository.
2. Build or pull the required images using the command documented in the service-specific README.
3. Run the Compose bundle (or merge it into the primary stack) once credentials and volumes are in place.
4. Push any updates to Containerfiles or Compose definitions back here so they stay versioned with the rest of the infrastructure.

For detailed instructions, consult the README inside each service directory and cross-reference the matching guide under `docs/`.