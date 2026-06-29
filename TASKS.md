# ewing-outcomes-harmonization — Task Backlog

> **Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 · **Owner:** TBD (maintainer) · **Lane:** donated
>
> Itemized, schema-mapped backlog for the project. Read alongside `PLAN.md`. **No patient-level data.
> No medical advice.** `verifiedNeed=false` across the board until a named partner is secured.

---

## How these tasks map to Elyos

Each task below becomes an **Elyos Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- **`id`** — stable slug ID, `eoh-<area>-NNN` (e.g. `eoh-schema-001`).
- **`title`** — the task title in the milestone tables.
- **`project`** — always `"ewing-outcomes-harmonization"`.
- **`type`** — one of `code | research | writing | data | design-spec | maintenance` (column "Type").
- **`lane`** — `donated` (default). A task is `funded` only if it has a `fundedBudgetUsd` cap.
- **`priority`** — `high | medium | low`.
- **`domain`** — string array, e.g. `["cancer-research","ewing-sarcoma","oncology-data"]`.
- **`riskTier`** — `low | medium | high`. Dataset/extraction = `medium`; patient-facing = `high`.
- **`urgent`** — boolean; all `false` (no secured time-critical beneficiary yet).
- **`deliverable`** — `pr | dataset | document | translation` (column "Deliverable").
- **`tokenEstimate`** — `small | medium | large` (column "Size").
- **`status`** — `open | in-progress | review | delivered | done`; all start `open`.
- **`context` / `objective` / `acceptanceCriteria[]` / `output`** — written per task.
- **`resources[]`** — source/reference links (DOI/PMID/NCT, dataset URLs).
- **`requestor`** — `"TBD (no partner secured)"` until a partner signs on.
- **`verifiedNeed`** — `false` until a named partner confirms consumption.
- **`outputLicense`** — `CC-BY-4.0` (data/docs) or `MIT` (code).
- **`fundedBudgetUsd`** — only on `funded` tasks (schema requires it then); none are funded here.

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| eoh-schema-001 | Design outcome record schema + JSON Schema | design-spec | medium | medium | document | — | Maintainer + domain reviewer |
| eoh-ontology-002 | Define ES outcome ontology + data dictionary v0.1 | writing | medium | medium | document | eoh-schema-001 | Domain reviewer |
| eoh-compliance-003 | Build per-source license register + aggregate-only intake checklist | data | small | medium | document | — | Maintainer |
| eoh-ci-004 | Implement CI validators (schema, provenance, min-cell, no-unsourced-number) | code | medium | medium | pr | eoh-schema-001 | Maintainer |
| eoh-pilot-005 | Pilot-extract ≥3 landmark open-access ES outcome studies (double-extracted) | data | medium | medium | dataset | eoh-schema-001, eoh-ontology-002, eoh-compliance-003, eoh-ci-004 | Domain reviewer |

**Acceptance criteria — key tasks**

- **eoh-schema-001 (schema):**
  - One-row-per-reported-outcome model implemented as a draft-07 JSON Schema in the repo.
  - Includes all core fields from PLAN data model (citation, source_license, source_locator,
    disease_extent, primary_site, age_band, risk_group, regimen_backbone, local_control, n,
    outcome_measure, event_definition, timepoint_years, estimate_pct, ci_low/high, estimate_method,
    median_followup_months, accrual_period, comparability_flags, provenance, notes).
  - Controlled vocabularies enumerated for all enum fields.
  - Schema validates the pilot examples and rejects a record missing provenance or with `n<5`.

- **eoh-compliance-003 (license register + intake):**
  - Register template captures access status, license, and permitted use per source.
  - Aggregate-only intake checklist requires affirmative "aggregate, de-identified" classification
    before a source enters extraction.
  - COSMIC/OncoKB and any controlled-access source explicitly marked **out of scope**.
  - No source can be marked "ready for extraction" with an incomplete license row.

- **eoh-ci-004 (CI validators):**
  - CI fails on: schema-invalid record, missing/incomplete provenance, any subgroup `n` below the
    documented floor, or any numeric value lacking a `source_locator`.
  - Validators run on every PR; documented and reproducible locally via pnpm.

- **eoh-pilot-005 (pilot extraction):**
  - ≥3 landmark OA studies harmonized; every record cites DOI/PMID + source table/figure + license.
  - Each record double-extracted; discrepancies adjudicated and logged.
  - 100% of records pass all CI gates; domain reviewer signs the pilot.

**M0 Definition of Done:** schema + ontology + data dictionary v0.1 published; CI gates live and
green; license register and intake checklist in place; ≥3 OA studies harmonized and reviewer-signed;
all records provenance-complete and within compliance rules.

## Milestone M1 — Corpus & extraction at scale

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| eoh-search-101 | Pre-registered search strategy + screening protocol | research | medium | medium | document | eoh-ontology-002 | Domain reviewer |
| eoh-register-102 | Frozen included-studies register (inclusion/exclusion applied) | data | medium | medium | dataset | eoh-search-101, eoh-compliance-003 | Domain reviewer |
| eoh-extract-103 | Extract localized-disease outcomes across included studies | data | large | medium | dataset | eoh-register-102, eoh-ci-004 | Domain reviewer |
| eoh-extract-104 | Extract metastatic & relapsed/refractory outcomes | data | large | medium | dataset | eoh-register-102, eoh-ci-004 | Domain reviewer |
| eoh-qa-105 | Double-extraction QA + ≥10% audit + adjudication log | data | medium | medium | document | eoh-extract-103, eoh-extract-104 | Domain reviewer |

**Acceptance criteria — key tasks**

- **eoh-search-101 (search strategy):**
  - Documented, reproducible queries for PMC-OA, ClinicalTrials.gov, SEER, Cochrane.
  - Explicit inclusion/exclusion (excludes case reports/series below sample floor; aggregate-only).
  - Pre-registered before screening to prevent cherry-picking.

- **eoh-register-102 (included-studies register):**
  - Frozen, versioned list with per-study license row complete before extraction.
  - Each study tagged by disease extent(s), era, regimen, and OA/closed-access status.

- **eoh-qa-105 (QA):**
  - ≥10% random sample independently re-extracted; field-level agreement ≥95%.
  - 100% of discrepancies adjudicated with a logged resolution and rationale.

**M1 Definition of Done:** ≥30 studies / ≥150 outcome records spanning localized, metastatic, and
relapsed disease; every source license-registered pre-extraction; audit agreement ≥95% with all
discrepancies adjudicated; all records pass CI gates.

## Milestone M2 — Harmonization & normalization

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| eoh-mapping-201 | Versioned mapping rules (era, regimen, extent, age, event-definition) | design-spec | medium | medium | document | eoh-ontology-002, eoh-extract-103, eoh-extract-104 | Domain reviewer |
| eoh-flags-202 | Apply comparability flags + non-comparability audit | data | medium | medium | dataset | eoh-mapping-201 | Domain reviewer |
| eoh-pooled-203 | Optional labeled pooled views (method-documented) | research | medium | medium | document | eoh-flags-202 | Biostatistician + domain reviewer |

**Acceptance criteria — key tasks**

- **eoh-mapping-201 (mapping rules):**
  - Each mapping rule versioned and reviewer-approved; original source framing preserved verbatim.
  - No raw label is silently coerced; ambiguous mappings flagged, not guessed.

- **eoh-pooled-203 (pooled views):**
  - Any pooled estimate is clearly labeled "derived, not as-published," with full method
    documentation and heterogeneity caveats; biostatistician sign-off recorded.
  - Pooling is opt-in; the default dataset still reports per-study outcomes.

**M2 Definition of Done:** ontology v1.0 + mapping rules reviewed and versioned; 100% of records
normalized and comparability-flagged; non-comparability audit passes; any pooled view is labeled,
documented, and biostatistician-reviewed.

## Milestone M3 — Publication & access

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| eoh-release-301 | Versioned CC-BY dataset release (CSV/Parquet/JSON) + Zenodo DOI | data | medium | medium | dataset | eoh-flags-202 | Maintainer + domain reviewer |
| eoh-dict-302 | Data dictionary v1.0 + provenance graph publication | writing | medium | medium | document | eoh-release-301 | Domain reviewer |
| eoh-notebook-303 | Reproducible researcher analysis notebook (caveat-laden) | code | medium | medium | pr | eoh-release-301 | Domain reviewer |
| eoh-readme-304 | README + "not medical advice" notices + contribution guide | writing | small | medium | document | eoh-release-301 | Maintainer |

**Acceptance criteria — key tasks**

- **eoh-release-301 (release):**
  - 100% records pass schema + provenance + min-cell + no-unsourced-number gates.
  - Semver-tagged, Zenodo DOI minted, changelog + "what's covered / what's missing" ledger published.

- **eoh-notebook-303 (notebook):**
  - Runs clean from the released artifacts; produces a caveat-laden comparison (e.g. 5-year EFS by
    disease extent and era) with comparability flags surfaced.
  - Explicitly labeled researcher-facing / not a patient tool.

**M3 Definition of Done:** public CC-BY dataset release with DOI; data dictionary + provenance graph
published; notebook runs clean from released data; README carries "not medical advice"; domain
reviewer signs the release (project Definition of Shipped met for the dataset).

## Milestone M4 — Sustainability, partner & (gated) education

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| eoh-maint-401 | Maintenance plan, update cadence + outcome-tracking ledger | maintenance | small | low | document | eoh-release-301 | Maintainer |
| eoh-partner-402 | Partner handoff + flip `verifiedNeed` on signed confirmation | maintenance | small | medium | document | eoh-release-301 | Maintainer + steward |
| eoh-edu-403 | (GATED) Plain-language "how to read ES survival numbers" explainer | writing | medium | high | document | eoh-release-301, eoh-partner-402 | **Oncologist + patient advocate** |

**Acceptance criteria — key tasks**

- **eoh-partner-402 (partner handoff):**
  - Named partner confirms consumption in writing; `verifiedNeed` flips to `true` in affected tasks.
  - Steward role assigned for last-mile delivery.

- **eoh-edu-403 (GATED education) — risk tier `high`:**
  - **Not started** unless an oncologist + patient-advocate reviewer pair is secured.
  - Every claim sourced to the dataset/literature; prominent "not medical advice"; no individualized
    prognosis; written for families with care and plain language.
  - Oncologist sign-off **and** patient-advocate review recorded before any publication.

**M4 Definition of Done:** documented update cadence + named maintainer; ≥1 partner consuming the
dataset (`verifiedNeed=true`); outcome-tracking ledger live; if (and only if) the high-tier reviewer
pair exists, ≥1 expert-signed explainer published through a vetted channel.

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Notes |
|---|---|---|---|---|---|
| eoh-bk-501 | Cross-link records to `ewing-open-data-catalog` study entries | data | small | low | Sibling-project integration |
| eoh-bk-502 | Translate gated explainer (hand off to `ewing-info-translations`) | writing | medium | high | Only after eoh-edu-403 signed |
| eoh-bk-503 | Automated source link-rot checker + archival snapshot capture | code | small | low | Sustainability hardening |
| eoh-bk-504 | Interactive forest-plot viewer (researcher-facing, static site) | code | medium | medium | Not a patient tool; caveats enforced |
| eoh-bk-505 | Quarterly new-trial review + dataset increment | maintenance | small | medium | Recurring update task |
| eoh-bk-506 | (Optional, funded) Bulk OA extraction with budget cap | data | large | medium | `funded` lane; requires `fundedBudgetUsd` |

## Example task JSON

Complete, schema-valid Task JSON for the first M0 task (`eoh-schema-001`). Validated against
`packages/schema/src/schemas.ts`: all required fields present, enums valid, `verifiedNeed=false`,
donated lane (no `fundedBudgetUsd` needed).

```json
{
  "id": "eoh-schema-001",
  "title": "Design outcome record schema + JSON Schema",
  "project": "ewing-outcomes-harmonization",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "ewing-sarcoma", "oncology-data", "data-harmonization"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "Published Ewing Sarcoma outcome data is not directly comparable across studies: event definitions (EFS/RFS/DFS/OS), disease extent, risk groups, chemotherapy backbones, local-control modality, treatment era, and follow-up all vary. There is no open, harmonized, provenance-rich substrate. This task defines the foundational data model (one row = one reported aggregate outcome) and its draft-07 JSON Schema so every downstream extraction is structured, comparable, and source-cited. Strictly aggregate, de-identified, open-access data only; no patient-level data; not medical advice.",
  "objective": "Produce a versioned JSON Schema and written data model for a single harmonized Ewing outcome record, capturing citation/provenance, cohort metadata (disease extent, primary site, age band, risk group), treatment metadata (regimen backbone, local control), the outcome (measure, event definition, timepoint, estimate, CI, method, median follow-up), accrual era, and comparability flags — with controlled vocabularies and enforceable compliance rules (provenance required; minimum-cell-size floor).",
  "acceptanceCriteria": [
    "Draft-07 JSON Schema committed implementing the one-row-per-reported-outcome model with all core fields from the PLAN data model.",
    "Controlled vocabularies enumerated for every enum field (disease_extent, primary_site, age_band, outcome_measure, estimate_method, local_control, source_license, comparability_flags).",
    "Schema requires complete provenance (DOI/PMID/NCT + source_locator + source_license + extractor + verifier + extraction_date) on every record.",
    "Schema rejects any record with a subgroup n below the documented minimum-cell-size floor (default n<5) and any numeric value lacking a source_locator.",
    "Schema validates the M0 pilot example records and rejects a deliberately malformed record in a test fixture.",
    "Data model documented with field definitions and rationale; reviewed and signed by a domain (sarcoma-oncology / clinical-epidemiology) reviewer."
  ],
  "resources": [
    "C:/code/elyos/planning/projects/ewing-outcomes-harmonization/PLAN.md",
    "C:/code/elyos/packages/schema/src/schemas.ts",
    "https://www.ncbi.nlm.nih.gov/pmc/tools/openftlist/",
    "https://clinicaltrials.gov/",
    "https://seer.cancer.gov/"
  ],
  "output": "A committed JSON Schema file plus a written data-dictionary draft (v0.1) defining the harmonized Ewing outcome record, its controlled vocabularies, and its compliance constraints, with passing validation fixtures.",
  "requestor": "TBD (no partner secured)",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
