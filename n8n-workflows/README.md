# n8n Workflows

Automation exports and supporting notes for MT Labs’ n8n orchestration stack. Each workflow directory captures the JSON export, configuration hints, and any complementary assets required to wire ERPNext with external services (e.g., WhatsApp via Evolution API).

## Structure

```/dev/null/tree.txt#L1-6
n8n-workflows/
├── README.md
├── invoicing/
└── ...
```

- `invoicing/` — Workflows that automate billing, status notifications, and ERPNext ledger updates.
- Additional subfolders can be added for each logical workflow family (e.g., `support-desk/`, `inventory-sync/`).

## Conventions

- **Exports:** Keep the latest `*.json` export from n8n alongside a short Markdown note describing triggers, credentials, and downstream effects.
- **Versioning:** Append semantic suffixes to filenames when iterating (e.g., `whatsapp-sync-v1.json`, `whatsapp-sync-v1.1.json`).
- **Environment Variables:** Reference `.env` keys defined in `docker-files/n8n/env/` and avoid storing secrets in the exports.
- **Dependencies:** Note any required n8n community nodes or external services in the accompanying Markdown.

## Workflow Lifecycle

1. Prototype inside the `experimental/` area until the flow is reliable.
2. Export the stable workflow from n8n and drop it here with a concise README section.
3. Update the corresponding service documentation under `docs/n8n/` so operational procedures stay aligned.
4. When changes ship to production, archive prior exports in an `archive/` subfolder if historical rollback is important.

## Getting Started

- Import a JSON export directly into an n8n instance via **Import from File**.
- Configure credentials and environment variables referenced in the workflow before activating it.
- Test end-to-end with staging ERPNext/Evolution API endpoints prior to enabling in production.

For deeper operational guidance, refer to `docs/n8n/` and the deployment notes in `docker-files/n8n/`.