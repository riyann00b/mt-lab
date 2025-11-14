# MT Labs Digital Operations Playbook

Repository documenting `moti-taka` self-hosted business operations stack—covering ERPNext deployment, automation workflows, messaging integrations, document infrastructure, and future self-hosted services.

## 🧭 Overview

MT Labs runs a fully self-hosted operations platform. This repository records:

- Infrastructure-as-documentation for ERPNext on Docker, including build pipelines, environment configuration, and production maintenance.
- Backup, restore, and troubleshooting procedures to keep critical data safe.
- Automation playbooks (n8n, messaging bridges, and API glue) that connect ERPNext with external services like WhatsApp (via Evolution API).
- Notes on supporting services such as document rendering (Gotenberg) and paperless document management.
- A roadmap for additional automations and self-hosted tooling as MT Labs expands.

Where possible, guides are tested end-to-end on Linux with a Bash-compatible shell. Adjust commands for your platform or shell of choice.

## 📁 Repository Map

| Area | Description |
|------|-------------|
| [`Installation.md`](Installation.md) | Step-by-step ERPNext Docker build, configuration, backup, and restore guide for custom app stacks. |
| [`Print Formats/`](Print%20Formats) | Collection of ERPNext print formats and related assets. |
| `Workflows/` *(planned)* | n8n workflow exports and documentation, including ERPNext–WhatsApp messaging automation. |
| `Integrations/` *(planned)* | Notes and configuration snippets for Evolution API, Gotenberg, Paperless, and other services. |
| `docs/` *(planned)* | Additional troubleshooting guides, runbooks, and FAQs. |

## 🚀 Quick Start

Clone the repo and explore the documentation locally:

```/dev/null/terminal.sh#L1-4
git clone https://github.com/riyann00b/mt-lab.git
cd mt-lab
ls
```

Recommended workflow:

1. Start with [`Installation.md`](Installation.md) to understand the ERPNext baseline.
2. Review `Print Formats/` for production-ready document templates.
3. Follow upcoming workflow guides to integrate messaging and automation.

## 🧱 ERPNext Platform Highlights

- **Custom Docker Build:** Instructions to assemble a layered image that bundles Frappe/ERPNext with custom apps (HRMS, Payments, India Compliance, and more).
- **Deployment Playbook:** Environment configuration, Docker Compose modifications (`pwd.yml`), and operational reminders such as platform targeting and app auto-installation.
- **Lifecycle Management:** Processes for upgrades, migrations, cache refresh, and safe restarts.
- **Callouts & Tips:** Inline hints for cross-platform command adjustments, PAT usage for private repositories, and validation steps for configuration files.

Dive into the sections inside `Installation.md` that match your current need—build, deploy, backup, or restore.

## ♻️ Backup & Restore Strategy

The backup chapter in `Installation.md` covers:

- Scheduled backups via Docker Compose jobs (with optional Restic integration).
- On-demand bench backups, including database and file archives.
- Secure file transfer patterns between host and containers.
- Comprehensive restore flow: staging archives, executing `bench restore`, migrating, and restarting services.

These workflows are the foundation for disaster recovery and environment cloning.

## 🤖 Automation & Workflow Orchestration

Coming soon:

- **n8n Workflows:** Orchestrations that sync ERPNext events, send WhatsApp updates through Evolution API, and coordinate other SaaS/self-hosted tools.
- **Workflow Documentation:** Trigger conditions, credential management best practices, rollback strategies, and test cases.

Expect exports (`*.json`) plus narrative guides as soon as the workflows are production-ready.

## 🔌 Integrations Portfolio

Planned coverage includes:

- **Evolution API (WhatsApp):** Message routing between ERPNext customer channels and WhatsApp, handling status callbacks, and fallbacks.
- **Gotenberg:** Headless document rendering for invoices and reports generated from ERPNext/Print Formats.
- **Paperless-ngx:** Ingestion pipeline for vendor invoices, HR documents, and automated filing back into ERPNext.
- **Additional Self-hosted Apps:** Tracking evaluation, deployment notes, and operational guidelines for future tools that complement MT Labs’ stack.

## 🧭 Roadmap Snapshot

- [ ] Publish n8n workflow exports with environment variable templates.
- [ ] Document Evolution API deployment and ERPNext webhook bindings.
- [ ] Add Gotenberg and Paperless install/operations guides.
- [ ] Expand troubleshooting catalog (bench commands, Docker maintenance, debugging tips).
- [ ] Introduce CI/CD recipes for image builds and configuration validation.

Progress is tracked in the repository issues and discussions.

## 🤝 Contributing & Maintenance

This repo primarily serves MT Labs’ internal operations, but structured contributions are welcome:

1. Open an issue describing the improvement or problem.
2. Fork the repo, make changes in a feature branch, and cross-reference the relevant guide sections.
3. Submit a pull request with test notes or reproduction steps.

When editing documentation, maintain the callout format (`[!NOTE]`, `[!TIP]`, `[!IMPORTANT]`) and path-aware code blocks so readers can follow along without guesswork.

## 📬 Support & Contact

- **Author:** Riyan (MT Labs)
- **Preferred Contact:** Open a GitHub issue in this repository.
- **Operational Support:** For urgent stack issues, refer to the backup/restore runbooks first; they cover the most common recovery scenarios.

---

_“Automate fearlessly, document ruthlessly.”_ This repository keeps MT Labs’ digital operations transparent, reproducible, and ready for the next automation wave.
