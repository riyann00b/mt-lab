# MT Labs Digital Operations Playbook

Repository documenting Moti Taka’s self-hosted business operations stack—covering ERPNext deployment, automation workflows, messaging integrations, document services, and future self-hosted tooling.

## 🧭 Overview

MT Labs runs a fully self-hosted operations platform. This repository captures:

- Infrastructure-as-documentation for ERPNext on Docker, including build pipelines, compose bundles, and production maintenance.
- Backup, restore, and troubleshooting procedures that keep mission-critical data safe.
- Automation playbooks (n8n, messaging bridges, and API glue) connecting ERPNext with channels such as WhatsApp via the Evolution API.
- Notes on supporting services like Gothenburg (document rendering) and Paperless document management.
- A roadmap for expanding automations and self-hosted applications across the organization.

Guides are authored and tested on Linux with a Bash-compatible shell. Adjust commands as needed for your operating system and shell environment.

## 📁 Repository Map

| Path | Purpose |
|------|---------|
| [`assets/`](assets/README.md) | Branding assets, print layouts, and other reusable UI collateral. |
| [`docker-files/`](docker-files/README.md) | Compose bundles, Containerfiles, environment samples, and scripts for each self-hosted service. |
| [`docs/`](docs/README.md) | Human-readable guides, runbooks, and troubleshooting notes grouped by platform. |
| [`n8n-workflows/`](n8n-workflows/README.md) | Automation exports (JSON), playbooks, and diagrams for business processes. |
| [`scripts/`](scripts/README.md) | Operational helper scripts for bench tasks, maintenance routines, and utilities. |
| [`experimental/`](experimental/README.md) | Work-in-progress experiments and prototypes slated for future productionization. |

Each subdirectory contains its own `README.md` to explain scope, conventions, and cross-links into the documentation set.

## 🗂️ Current Layout

```/dev/null/tree.txt#L1-20
mt-lab/
├── README.md
├── LICENSE
├── assets/
├── docker-files/
├── docs/
├── experimental/
├── n8n-workflows/
└── scripts/
```

Inside `docs/` and `docker-files/`, services are organized consistently (`erpnext/`, `evolution-api/`, `gothenburg/`, `n8n/`, `paperless/`) so prose and configuration stay in sync.

## 🚀 Quick Start

```/dev/null/terminal.sh#L1-4
git clone https://github.com/riyann00b/mt-lab.git
cd mt-lab
ls
```

Suggested reading order:

1. Browse `docs/erpnext/installation.md` for the build-and-deploy walkthrough.
2. Check `docs/erpnext/print-formats/` for production-ready document templates.
3. Explore `docker-files/erpnext/` to review Containerfiles, Compose stacks, and environment samples.
4. Inspect `n8n-workflows/` for automation assets that bridge ERPNext with WhatsApp, email, and other channels.

## 🧱 ERPNext Platform Highlights

- **Custom Docker Build:** Layered image instructions bundling ERPNext with HRMS, Payments, India Compliance, and other custom apps.
- **Deployment Playbook:** Guidance on tailoring `pwd.yml`, setting platform targets, and automating app installs.
- **Lifecycle Management:** Procedures for upgrades, migrations, cache refresh, and safe container restarts.
- **Callouts & Tips:** Throughout the docs you’ll find `[!NOTE]`, `[!TIP]`, and `[!IMPORTANT]` callouts covering cross-platform command adjustments, private repo PAT usage, and sanity checks for configs.

Consult the relevant section in `docs/erpnext/` based on whether you’re building, operating, or recovering an instance.

## ♻️ Backup & Restore Strategy

`docs/erpnext/troubleshooting/` and `docs/erpnext/installation.md` document:

- Scheduled backups via dedicated Compose services (including optional Restic integration).
- On-demand bench backups (database + files) and file transfer patterns between host and container.
- Restore workflows: staging archives, running `bench restore`, migrating, and restarting services with confidence.

These runbooks are the backbone for disaster recovery and environment cloning.

## 🤖 Automation & Workflow Orchestration

The `n8n-workflows/` tree (with matching narratives under `docs/n8n/`) will contain:

- **n8n Workflow Exports:** JSON definitions for ERPNext event handling, invoicing automation, and WhatsApp messaging via Evolution API.
- **Operational Playbooks:** Credential management, environment variable templates, rollback strategies, and test cases for each flow.

Updates will land as workflows graduate from experimental to stable status.

## 🔌 Integrations Portfolio

Documentation and configs live in `docs/<service>/` and `docker-files/<service>/` for:

- **Evolution API (WhatsApp):** Message routing, delivery status callbacks, and ERPNext webhook bindings.
- **Gothenburg:** Headless document rendering for ERPNext print formats.
- **Paperless:** Document ingestion pipelines, automated classification, and synchronization back into ERPNext.
- **Future Services:** Any additional self-hosted tools required for MT Labs’ operations.

## 🧭 Roadmap Snapshot

- [ ] Publish n8n workflow exports with environment variable templates.
- [ ] Document Evolution API deployment alongside ERPNext webhook configuration.
- [ ] Add Gothenburg and Paperless install/operations guides.
- [ ] Expand troubleshooting catalog (bench commands, Docker maintenance, debugging tips).
- [ ] Introduce CI/CD automation for image builds and documentation linting.

Roadmap progress is tracked through repository issues and discussions.

## 🤝 Contributing & Maintenance

While the repo primarily supports MT Labs’ internal operations, structured contributions are welcome:

1. Open an issue describing the improvement or problem.
2. Fork the repo, implement changes in a dedicated branch, and reference the pertinent docs.
3. Submit a pull request with testing notes or reproduction steps.

Please preserve the callout pattern (`[!NOTE]`, `[!TIP]`, `[!IMPORTANT]`) and use path-based code fences to keep the documentation style consistent.

## 📬 Support & Contact

- **Author:** Riyan (MT Labs)
- **Preferred Contact:** Open a GitHub issue in this repository.
- **Operational Support:** For urgent incidents, follow the backup/restore runbooks—they cover the most common recovery scenarios before escalation.

---

**“Automate fearlessly, document ruthlessly.”** This playbook keeps Moti Taka’s digital operations transparent, reproducible, and ready for the next wave of automation.