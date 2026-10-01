# CF Inventory Explorer — User Guide & README

**File:** `CF-Inventory-Explorer_Final_Version.html`
**Author:** Prasanna Krishna Reddy Ch
**Type:** Single-file, client-side HTML tool (no install, no server, no data leaves your machine)

---

## 1. What this tool is

Where *Data Discovery* helps you **design new** calculated fields, CF Inventory Explorer helps you **understand, govern, and replicate the ones you already have**.

Attach one Master CF DDR export and you can:

- Trace any CF's full lineage — what it's built from, and every CF built on top of it, level by level until the chain goes blank.
- See the **blast radius** before you change a hub CF.
- Find dead weight: unused CFs, Do-Not-Use exposure, duplicate definitions.
- Discover which reports, condition rules, integrations, and notifications actually consume a CF.
- Generate a **Replication Kit** — a dependency-safe, numbered build order for recreating a CF (and everything it depends on) in another tenant, rendered in Workday's own CF form layout.

Everything runs in your browser. Nothing is uploaded anywhere.

---

## 2. Quick start (60 seconds)

1. Open `CF-Inventory-Explorer_Final_Version.html` in Chrome or Edge.
2. In **Tenant Files**, attach your Master CF Inventory JSON. The status shows e.g. *"28,658 CFs attached"*.
3. The **Inventory Dashboard** loads immediately.
4. Go to **CF Lineage Explorer**, type a CF name (autocomplete shows BO + function), click **Explore**.
5. Click **Build Replication Kit** → switch to **Workday View** → expand any card with the **+** symbol.

---

## 3. Input file

One JSON: the Master Calculated Field DDR export. The tool reads these fields (all optional except the first two — missing fields degrade gracefully rather than breaking):

| Field | Used for |
|---|---|
| `Calculated_Field_Starting_Point` | The CF's name |
| `Calculated_Field_starting_Field_Reference_ID` | **Unique key.** Makes chains exact even when names are duplicated |
| `Calculated_Field_Business_Object` | BO badge, grouping, cardinality learning |
| `Calculated_Field_Function` | Which of the 34 Workday functions — drives the form template |
| `Calculated_Field_Starting_Point_Field_Type` | Text / Boolean / Numeric / Date / Currency / Single instance / Multi-instance … |
| `Calculated_Field_Starting_Point_Related_Business_Object` | Related BO row on the CF card |
| `Standard_Fields_Used_In_Calculated_Field_Starting_Point` | The standard fields the CF references |
| `Standard_Fields_With_DNU` | Replication blockers |
| `In_Used`, `Is_DNU`, `Standalone_Or_Shared_Calculated_Field` | Governance flags |
| `Calculated_Field_Description` | Description row |
| `Calculated_Field_Where_Used_Level_1_group` → `Starting_Point_Used_Calculated_Field` + `…Used_In_Reference_ID` | **The chain.** Which CFs consume this CF |
| `Where_Used_group` → `Area` + `Usage` | Where the CF is consumed outside CFs (reports, rules, integrations …) |

> **Reference IDs matter.** With them, chains resolve exactly. Without them the tool falls back to name matching, which loses branches when names repeat across BOs.

---

## 4. The six tabs

### 4.1 Inventory Dashboard
Nine KPI tiles (total CFs, unique names, in use, not in use, Do Not Use, shared, standalone, BOs covered, max chain depth) plus six panels: CFs by Function (all 34), Top 15 Business Objects, Standalone-vs-Shared and In-Use-vs-Not donuts, Most Reused CFs, a Governance summary, and Lineage Depth (the longest CF-on-CF chain before it goes blank).

### 4.2 CF Lineage Explorer — the core workflow
Search any CF and you get three blocks:

**Detail card(s)** — Workday-style, one per record (duplicate names each get their own): function, BO, field type, related BO, description, standard fields, and a **Used In** list. Badges flag Shared / Do Not Use / Not In Use.

**Built From (upstream CFs)** — the ingredient CFs this one is made of, plus the **Build Replication Kit** button.

**Where Used (downstream tree)** — your chain, exactly as specified: click any branch to expand its consumers, level by level, **until nodes are marked "Terminal — chain ends here"** (blank Where-Used). Circular references are detected and flagged rather than looping forever.

### 4.3 Impact Analysis — "what breaks if I change this?"
Full recursive downstream closure with KPIs: affected CFs, impact levels, Business Objects hit, shared CFs affected, Do-Not-Use CFs in the blast radius — then the complete affected list grouped by level, closest first. (Example from a real tenant: `Worker Status` → **199 CFs across 3 levels**.)

### 4.4 Governance & Cleanup
Four panels, computed on first open:
- **Safe-delete candidates** — Not In Use **and** no other CF is built on them. Filterable by BO.
- **Do-Not-Use exposure** — flagged DNU but still in use or still consumed. Governance contradictions worth fixing.
- **Replication blockers** — CFs whose standard fields include DNU fields; these will fail in a clean target tenant.
- **Consolidation candidates** — same BO + same function + identical standard-field set. *Candidates for review, not proven duplicates* — the export doesn't include condition logic, so two CFs can share fields yet differ in conditions.

### 4.5 Usage Explorer
A reverse index of `Where_Used_group`: filter by **Area** (Custom Report, Condition Rule, Workflow Notification, Integration System, Discovery Board, Leave Type …) and search a usage name. Each consumer expands to show **exactly which CFs it uses** — effectively a per-report migration checklist. Every CF listed is clickable straight into Lineage.

### 4.6 Browse Inventory
The filterable catalogue: name contains, BO, function, In Use, Shared/Standalone, Do Not Use. Each row shows badges, description, standard-field count, direct-consumer count (with a *terminal* marker), and the Reference ID. Click any name to jump into Lineage.

---

## 5. The Replication Kit

Click **Build Replication Kit** on any CF. The tool walks the complete upstream ingredient closure and prints a **dependency-safe build order** — every CF appears *after* everything it is built from, so you can work top to bottom in the target tenant.

**Two views, switchable:**
- **List View** — compact numbered steps with badges and standard fields.
- **Workday View** — one collapsible card per step (**+** to expand), replicating the real Workday Calculated Field page: blue banner with function name, Field Name / Business Object / Description, the *Calculation* tab, and the function's real form rows.

**Standard-field colour code (both views):**
- 🔵 **Blue = new at this step** — a field this CF references directly.
- 🟠 **Amber = repeated from an earlier step** — inherited from an ingredient CF; already accounted for.

This exists because the DDR accumulates ingredient fields down the chain, which made long kits confusing to read.

---

## 6. How the CF cards are filled in

Each card follows the real Workday form for its function (all 34 functions have their own template — LRV, ESI, EMI, ARI, Sum/Count/Average Related, True/False Condition, Evaluate Expression (Band), Lookup Hierarchy / Hierarchy Rollup, Lookup Organization / Organizational Roles / Range Band / Field with Prompts / Translated Value / Value As Of Date / Date Rollup, Build Date, Date Difference, Increment or Decrement Date, Arithmetic, Concatenate, Substring, Format Text/Date/Number, Convert Currency, Convert Text to Number, Text Length, Prompt for Value, and the constants).

**Slot-filling rules:**
1. **The previous CF in the chain fills the Source / Lookup Field** — that's how Workday chains work.
2. **Boolean-type ingredient CFs and condition-type fields (`Any is True`, `Is True` …) go to Condition** (and Sort Field for ESI), never to Source or Return.
3. **Return Value sits at the bottom**, and is derived from the CF's **own fields** — its standard-field list *minus* everything inherited from its ingredients (i.e. the blue chips).
4. **Type-mandatory slots take strictly-typed ingredients.** A Date slot takes a Date field; a Numeric constant goes to the amount row. (You increment 90 days *from* Hire Date — not the other way round.)
5. **LRV cardinality rule:** the Lookup Field must be a **Single instance** field; a Multi-instance field can only be a Return Value. Return Value itself may be **any type** — Text, Numeric, Date, instance, including Multi-instance.

> ### Caution — the cards are a scaffold, not a specification
>
> **Every function's design can vary with the requirement.** The same function is configured differently from tenant to tenant and from requirement to requirement — different conditions, sort fields, occurrences, operation types, formats and return fields. The card shows the correct **form structure** and everything the export can prove; it is **not guaranteed to be a 100% match** to the CF as it must exist in the target tenant.
>
> Always review each card against the requirement (and against the source tenant's CF, where you can see it) before building. Use it to remove the guesswork, not to skip the thinking.

**What the tool will not guess.** Anything the export cannot prove — operators, formats, delimiters, thresholds, hierarchy options, and ambiguous base-CF lookup/return pairs — renders in **red italics: *"Depends on User requirement"***, with the governing rule stated inline where one exists. This is deliberate: a wrong slot assignment silently breaks a build in the target tenant, whereas a red line costs you ten seconds of judgment you would have applied anyway.

**Cardinality learning.** The tool teaches itself which standard fields are Multi-instance, purely from objective facts in your own file (a base ESI/EMI/Count must have a Multi-instance source), scoped to Business Object + field name, with contradictions discarded. **No naming conventions are used** — CF names are user customisation and cannot be trusted as a signal.

---

## 7. Honest limitations

- **The Replication Kit is a scaffold, not a specification.** Function layouts and configurations vary by requirement; the rendered cards will not always be a 100% match to what you must build. Review every card before building.
- **Only what the export contains can be rendered.** No condition logic, no operators, no saved form settings — these appear as red *"Depends on User requirement"* placeholders rather than guesses.
- **Radio selections and defaults on the cards** (Ascending, First occurrence, "Return Zero on Error: Yes", "(empty)") are the *Workday form defaults*, not each CF's saved settings. The DDR does not export those settings.
- **Base-CF lookup/return ambiguity.** For a base LRV with two candidate fields and no provable cardinality, both candidates are shown with the rule. This is a data limitation, not a logic gap — if the DDR ever emits instance tags on standard-field names (`Enrolled Content (Single instance)`, as the Data Discovery exports do), these cases resolve mechanically.
- **Consolidation candidates are candidates**, not proven duplicates (no condition logic in the export).
- **`In_Used = 1` means "something uses it"** — the Usage Explorer tells you *what*; CFs with no recorded usage are worth reviewing before deletion, not deleting blind.
- **Large lists are capped** for browser performance (300–500 rows per panel). Narrow the filters to see more.

---

## 8. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| "Invalid JSON" | Not the Master CF DDR export, or a partial download |
| A CF isn't found | Pick it from the autocomplete — names must match exactly |
| Chain looks shorter than expected | Older file without Reference IDs; regenerate the DDR with the reference-ID column |
| A branch says "no record in inventory" | The consumer CF isn't in the export (filtered report, or a deleted CF) |
| Governance tab is blank | It computes on first open — switch to the tab once with a file attached |
| Numbers differ from a previous run | The export is a snapshot; regenerate it to refresh |

---

## 9. Technical notes

- **Stack:** single HTML file; Bootstrap 5.2, jQuery 3.5 + jQuery UI 1.12 from CDN.
- **Indexes built on load:** by name, by Reference ID, upstream (name and ID), usage reverse index, learned cardinality sets. Index build on ~28.6K CFs takes ~0.2–0.5s; every tab is instant thereafter.
- **Cycle safety:** all recursive walks (tree, kit, impact, depth) are cycle-guarded.
- **Portability:** no tenant data is hardcoded. The same file works on any tenant's Master CF DDR export with the same schema. Fixed by design: the 34 Workday form templates, the JSON field-name contract, and the English condition-field pattern (`is/are true/false`).

---

## 10. Recommended workflows

**Before changing a hub CF** → Impact Analysis → check shared CFs and DNU CFs in the blast radius.

**Quarterly cleanup** → Governance & Cleanup → export the safe-delete list by BO → confirm each in Usage Explorer → retire.

**Migrating a report to a new tenant** → Usage Explorer → find the report → note its CFs → Lineage Explorer on each → Build Replication Kit → build in Workday View order.

**Onboarding onto an unfamiliar tenant** → Inventory Dashboard → then Browse Inventory filtered to Shared + In Use to learn the CFs that matter.
