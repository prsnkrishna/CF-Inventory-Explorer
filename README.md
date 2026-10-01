# CF Inventory Explorer

**Author:** Prasanna Krishna Reddy Ch

Turns a Workday calculated field inventory export into a navigable dependency graph:
lineage, impact, governance, usage, and a dependency-safe replication kit for another tenant.

---

## What's in this repository

| Path | What it is |
|---|---|
| `index.html` | **The tool.** Open in Chrome or Edge. Nothing to install. |
| `docs/How-To-Use.pptx` | Walkthrough with screenshots. Start here if you're new. |
| `docs/User_Guide.md` | Full reference: every tab, rule, limitation and troubleshooting step. |
| `docs/High-Level_Overview.docx` | Two-page summary for leads and stakeholders. |
| `report-design/` | The Master CF DDR report definition (xlsx + pdf). Use it to build the export in any tenant. |
| `sample-data/sample_export.json` | A sample export so you can try the tool immediately. |

---

## First run (2 minutes)

1. Download or clone this repository and open `index.html`.
2. Attach the Master CF Inventory JSON (for example `sample-data/sample_export.json`). The **Inventory Dashboard** loads straight away.
3. Go to **CF Lineage Explorer**, search a calculated field, click **Explore**.
4. Click **Build Replication Kit**, switch to **Workday View**, expand a card with **+**.

**The six tabs:** Inventory Dashboard · CF Lineage Explorer · Impact Analysis ·
Governance & Cleanup · Usage Explorer · Browse Inventory.

All processing happens locally in your browser. No data is uploaded anywhere.

---

## Two things to know before you rely on it

- **Include the Reference ID column in the DDR export.** Reference IDs are unique per
  calculated field, so chains resolve exactly even when names repeat across Business Objects.
  Without them the tool falls back to name matching and silently loses branches.
- **The Replication Kit is a scaffold, not a specification.** Every function's design varies
  with the requirement: conditions, sorts, occurrences, operation types, formats, return
  values. The cards show the correct form structure and everything the export can prove, but
  they will not always be a 100% match. Review each card before building. Anything the export
  cannot prove is shown in red as *"Depends on User requirement"*. The tool never guesses.

---

## Note on your own exports

Real tenant exports contain custom calculated field names, report names, org names and
user IDs. Do not commit them to this repository. The `.gitignore` is set up to help.
