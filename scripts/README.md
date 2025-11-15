# Scripts

Utility helpers that streamline recurring operational tasks across MT Labs’ self-hosted stack. Use these scripts as building blocks for local development, maintenance windows, and automation hooks.

## Layout

- `bench/` — Commands tailored for ERPNext bench operations (e.g., migrations, asset rebuilds).
- `maintenance/` — General-purpose cleanup, log rotation, and health-check helpers.
- Additional subdirectories can be introduced as new script families emerge; keep names lowercase with hyphens for clarity.

## Guidelines

- Prefer POSIX-compliant shells so the scripts run consistently in containerized and bare-metal environments.
- Document assumptions (required binaries, environment variables, expected working directory) in header comments.
- Avoid hardcoding secrets. Read them from environment variables or `.env` files that live outside version control.
- Whenever a script changes system state, echo concise status messages so logs remain informative.

Coordinate script usage with the service documentation in `docs/` to ensure operational steps stay in sync.