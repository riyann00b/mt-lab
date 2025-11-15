# Documentation Hub

This directory houses MT Labs’ written playbooks: installation guides, operational runbooks, troubleshooting notes, and integration references for our self-hosted stack.

## Navigation

Current service-specific collections:

- `erpnext/` — Build/install guide, print-format catalogue, and troubleshooting notes for the ERPNext platform.
- `evolution-api/` — Documentation for the WhatsApp bridge (pending publication).
- `gothenburg/` — Document rendering service notes (pending).
- `n8n/` — Workflow design guides and orchestration tips (pending).
- `paperless/` — Paperless document management workflows (pending).

A quick snapshot of the structure:

```/dev/null/tree.txt#L1-10
docs/
├── README.md
├── erpnext/
│   ├── installation.md
│   ├── print-formats/
│   └── troubleshooting/
├── evolution-api/
├── gothenburg/
├── n8n/
└── paperless/
```

## Writing Conventions

- **Callouts:** Use `[!NOTE]`, `[!TIP]`, `[!IMPORTANT]`, and `[!WARNING]` for quick-scanning critical information.
- **Code Blocks:** Always prefer path-aware fences (` ```path/to/file#Lx-y `) so readers can trace the originating file or command context.
- **Shell Commands:** Assume a Bash-compatible environment on Linux. Mention platform differences only when necessary.
- **Links:** Use relative paths to keep cross-references portable inside the repository.

## Template Checklist

When drafting a new guide, cover the following:

1. **Summary:** One paragraph describing the scope of the document.
2. **Prerequisites:** Dependencies, credentials, or infrastructure requirements.
3. **Step-by-step Flow:** Numbered sections with command snippets or screenshots as needed.
4. **Validation:** How to confirm the procedure succeeded.
5. **Rollbacks / Recovery:** Fallback steps if something fails.
6. **References:** Link to upstream docs, diagrams, or related guides.

Feel free to add appendices for expanded troubleshooting or FAQ content.

## Maintenance

- Keep headings and filenames lowercase with hyphens for consistency.
- Archive superseded docs under the service directory with a `deprecated/` subfolder if historical context is important.
- Log major updates in the corresponding service README so changes remain discoverable.

## Roadmap

- [ ] Publish workflow documentation for `n8n/`.
- [ ] Add deployment notes for `evolution-api/`, `gothenburg/`, and `paperless/`.
- [ ] Expand ERPNext troubleshooting with performance and scaling guides.
- [ ] Introduce diagram exports where visual flows aid comprehension.

To contribute, open an issue or pull request referencing the document you’re updating. All edits should follow the conventions outlined above.