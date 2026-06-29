# ewing-outcomes-harmonization — Project Plan

> **Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 · **Owner:** TBD (maintainer) · **Lane:** donated
>
> Harmonize **published, aggregate** Ewing Sarcoma outcome data across studies into one open,
> provenance-rich, machine-readable dataset — so researchers can compare like with like, advocates
> can argue from evidence, and family-facing education can be grounded in sourced numbers instead of
> hearsay. **No patient-level data. No medical advice.**

---

> **For the families reading this.** Ewing Sarcoma is rare, it mostly strikes children, teenagers,
> and young adults, and when a family goes looking for "what are the chances," they hit a wall of
> papers that each count survival a slightly different way, over different eras, with different
> treatments — numbers that look comparable but are not. That confusion costs trust and, sometimes,
> good decisions. This project does not give anyone an answer about their own child. It does the
> unglamorous, careful work of lining up what the published studies *actually* reported, exactly as
> they reported it, with the source attached to every number — so that the people who counsel
> families, and the resources families read, are standing on solid, cited ground. We treat every
> figure as if a frightened parent will read it, because one might.

---

## Executive summary

Ewing Sarcoma (ES) outcomes are reported across decades of clinical trials and cohort studies, but
the published numbers are **not directly comparable**. Studies differ in how they define an "event"
(EFS vs. RFS vs. DFS), which disease extent and risk groups they include (localized vs. metastatic
vs. relapsed; axial vs. appendicular primary), which chemotherapy backbone and local-control
modality they used, the treatment era and follow-up duration, and how they computed and reported
confidence intervals. A 5-year EFS of "73%" in one paper and "68%" in another may be measuring
different things on different populations. This heterogeneity slows meta-analysis, muddies advocacy,
and produces family-facing materials that quote a single scary or rosy number out of context.

This project builds an **open, harmonized, fully-provenanced dataset of aggregate ES outcomes**
extracted from the published literature and public registries. Every record is a single reported
outcome (e.g., "5-year EFS, localized, VDC/IE backbone, n=…, 73% [95% CI …]") tied to: a normalized
outcome ontology, structured cohort/treatment/era metadata, and a citation down to the source table
or figure. The deliverable is a versioned dataset (CSV/Parquet + JSON), a published data dictionary,
a provenance graph, and a small reproducible analysis notebook — released under CC-BY-4.0 (code
MIT). A **separate, high-risk, expert-reviewed** plain-language explainer ("how to read Ewing
survival numbers") sits downstream and is explicitly gated.

**Hard guardrails (binding, lead the compliance section):** only **open-access / aggregate /
de-identified** sources; **controlled-access (dbGaP, EGA, individual-level biobanks) and any
identifiable patient data are strictly out of scope**; per-source license verification; provenance
on every assertion; no medical advice; any patient-facing output is education-only, sourced, carries
a "not medical advice" notice, and requires **oncologist + patient-advocate sign-off** (risk tier
`high`). The harmonization dataset itself is risk tier **medium**.

**Honest status:** No partner organization or beneficiary commitment is secured yet. `verifiedNeed`
is **false** across the backlog until a named partner (e.g., a Ewing/sarcoma research consortium or
a patient-advocacy foundation) confirms the need and agrees to consume the output. This is a
foundational, low-blast-radius data project that can begin (schema, license register, pilot
extraction) before a partner is secured, but it must **not be promoted as family-facing guidance**
until the high-risk review chain and a steward are in place.

## Problem & beneficiaries

**The problem.** Published ES outcome data is fragmented and silently incomparable:

- **Definitional drift.** "Event-free survival," "relapse-free survival," "disease-free survival,"
  and "overall survival" are used inconsistently; the *definition of an event* (local recurrence?
  second malignancy? death from any cause?) varies by study and is often buried in methods.
- **Population heterogeneity.** Localized vs. metastatic vs. primary-refractory vs. relapsed disease;
  pelvic/axial vs. extremity primaries; pediatric vs. adolescent-and-young-adult (AYA) vs. adult;
  differing risk stratification schemes (tumor volume, histologic response, metastatic burden).
- **Treatment-era effects.** Outcomes from VACD-era trials are not comparable to interval-compressed
  VDC/IE (AEWS0031) or VIDE/VAI (Euro-EWING) regimens; local control by surgery vs. radiotherapy vs.
  both shifts results.
- **Statistical reporting variance.** Point estimates with/without confidence intervals, Kaplan–Meier
  vs. crude proportions, differing timepoints (3-, 5-, 10-year), variable median follow-up.

The result: meta-analysts re-extract the same numbers repeatedly with no shared, audited source of
truth; advocates cite figures without context; and family-facing resources reproduce single numbers
stripped of the population and era they came from.

**Beneficiaries (in priority order).**

1. **ES / pediatric-sarcoma researchers and meta-analysts** — get a reusable, citable, harmonized
   substrate that removes weeks of duplicated extraction and reduces extraction error.
2. **Patient-advocacy foundations and advocate-researchers** (e.g., Ewing-focused charities,
   sarcoma alliances) — get evidence they can stand behind when arguing for trials, funding, and
   standards of reporting.
3. **Clinician-educators and the authors of family-facing materials** — get sourced, contextualized
   numbers to build *education* on (downstream, gated, expert-reviewed).
4. **Families and patients (indirect)** — benefit only through expert-reviewed education derived from
   the dataset, never from the raw dataset presented as guidance.

**Verified need:** **TO BE SECURED.** No partner has yet confirmed they will consume this output.
Candidate partners include sarcoma research consortia (e.g., COG sarcoma committee–adjacent academic
groups), Euro Ewing collaborators, and ES patient foundations. Until a named partner signs on,
`verifiedNeed=false` and the project runs as a foundational data effort, not a delivered service.

**Partner org:** TO BE SECURED.

## Goals and non-goals

**Goals**

- Produce a **versioned, open, machine-readable dataset** of aggregate ES outcomes extracted from
  published literature and public registries, with provenance on every record.
- Define and publish an **outcome ontology + data dictionary** that makes cross-study comparison
  explicit (what each measure means; what is and isn't comparable).
- Establish a **rigorous, double-extraction, source-cited pipeline** with conservative
  inclusion/exclusion rules and a per-source license register.
- Ship a **reproducible analysis notebook** that demonstrates correct, caveat-laden comparison
  (e.g., forest-style visualization of 5-year EFS by disease extent and era) — for researchers, not
  patients.
- Provide a **gated path** to a high-risk, expert-reviewed, plain-language explainer on *how to read
  ES survival statistics* (education only).

**Non-goals**

- **No patient-level / individual-level data** of any kind, ever (see Scope and Compliance).
- **No medical advice, prognosis, or treatment recommendation** for any individual.
- **No novel statistical claims or de novo meta-analysis conclusions** in the core dataset — we
  harmonize and cite what was published; any pooled estimate is clearly labeled, methodologically
  documented, and expert-reviewed before publication.
- **No re-analysis of raw genomic/transcriptomic data** (that is `ewing-expression-reanalysis`'s
  scope, not this project's).
- **No scraping of paywalled full text** beyond what license terms permit; no redistribution of
  copyrighted tables/figures (we extract facts + cite, see Compliance).
- **No clinical decision support tool.**

## Success metrics (outcomes)

Outcome-based and beneficiary-centric, not vanity counts. Baselines are effectively zero (no such
open harmonized resource exists for ES today).

| Outcome | Baseline | Target (12 mo) | How measured |
|---|---|---|---|
| Researchers reuse the dataset in real work | 0 | ≥3 independent reuses (citation, fork, or written acknowledgement) | Citation/issue/fork tracking + partner attestations |
| Coverage of landmark ES outcome studies | 0 | ≥30 studies / ≥150 harmonized outcome records spanning localized, metastatic, and relapsed disease | Dataset row count vs. a pre-registered study list |
| Extraction accuracy (vs. independent re-extraction) | n/a | ≥95% field-level agreement on a 10% audited sample; 100% of discrepancies adjudicated | Double-extraction QA log |
| Provenance completeness | n/a | 100% of records cite source down to table/figure + DOI/PMID + license | Automated provenance validator (CI gate) |
| Partner adoption | none | ≥1 named partner consuming the dataset; `verifiedNeed` flips true | Signed partner confirmation |
| Family-facing benefit (indirect) | none | ≥1 expert-reviewed explainer published, oncologist + advocate signed | Sign-off records |
| Harm avoided | n/a | 0 incidents of patient-level data ingestion; 0 unsourced numbers shipped | Compliance audit log |

A deliberately **negative** success metric is included (zero patient-level ingestions, zero
unsourced numbers): for a cancer data project, *not* causing harm is itself an outcome we measure.

## Scope

**In scope**

- Extraction of **aggregate** outcome statistics already published in the peer-reviewed literature
  and public trial registries (group-level rates, point estimates, confidence intervals, sample
  sizes, median follow-up).
- Structured metadata: study design, trial/registry ID, accrual era, disease extent, primary site,
  age band, risk group, chemotherapy backbone, local-control modality, outcome measure + event
  definition, timepoint.
- An outcome ontology, mapping rules, and a published data dictionary.
- Provenance graph linking every record to its source (DOI/PMID/NCT, page/table/figure, license).
- A reproducible, researcher-facing analysis notebook with explicit comparability caveats.
- A **gated** downstream high-risk explainer (separate deliverable, separate review chain).

**Out of scope (explicit)**

- **Any patient-level / individual-level data.** No dbGaP, no EGA, no individual biobank records, no
  per-patient survival rows, no individual-patient-data (IPD) meta-analysis.
- **Identifiable data of any kind**, including small published subgroups that could be re-identifying
  (see minimum-cell-size rule in Compliance).
- Case reports and case series below a sample-size floor (low aggregate validity *and* higher
  re-identification risk).
- Paywalled full-text redistribution; reproduction of copyrighted figures/tables verbatim.
- Genomic/transcriptomic re-analysis; biomarker discovery; drug-target claims.
- Any individualized prognostic tool, calculator, or advice surface.
- Pooling/meta-analytic *conclusions* presented as authoritative without expert sign-off.

## Solution approach & architecture

This is primarily a **data-curation pipeline + open dataset**, with light tooling. Stack follows
Elyos conventions: TypeScript/ESM for tooling, pnpm workspace; Python (pandas/lifelines optional)
permitted in a clearly-scoped `analysis/` subdir for the reproducible notebook; data published as
CSV + Parquet + JSON Schema-validated JSON.

**Pipeline stages**

1. **Source identification & screening.** A pre-registered search strategy (PubMed/PMC, Cochrane,
   ClinicalTrials.gov, SEER aggregate tables) → candidate study list → inclusion/exclusion screening
   against documented criteria → frozen, versioned "included studies" register.
2. **License triage (gate).** Before any extraction, each source's license/access status is recorded
   in a **per-source license register**; a source cannot enter extraction until its row is complete
   and its access is confirmed open/permissible (see Compliance).
3. **Structured extraction.** Human/agent extracts aggregate outcomes into the record schema using a
   controlled template; **facts only** (numbers, definitions), with verbatim quotes limited to short
   methods snippets needed to pin down definitions. Every field carries provenance.
4. **Double extraction & adjudication.** A 100% second-pass on a defined core set and ≥10% random
   audit on the rest; discrepancies logged and adjudicated; agreement metrics tracked.
5. **Normalization / harmonization.** Map raw outcome labels to the ontology; normalize era,
   regimen, disease extent, age band, and event definitions via documented, versioned mapping rules;
   flag non-comparable records explicitly rather than forcing them into one bucket.
6. **Validation (CI gate).** JSON Schema validation, provenance completeness check, controlled-vocab
   checks, minimum-cell-size check, and a "no unsourced number" check run in CI on every change.
7. **Publication.** Versioned release (semver + dataset DOI via Zenodo), data dictionary, provenance
   graph, changelog, and the researcher notebook.
8. **(Gated) education layer.** Downstream explainer authored from the dataset, routed through the
   high-risk review chain before any publication.

**Harmonized outcome record — core data model (one row = one reported outcome)**

| Field | Type / vocab | Notes |
|---|---|---|
| `record_id` | string | Stable ID |
| `study_id` | string | FK to study register |
| `citation` | object | `{ doi, pmid, nct, title, year, journal }` |
| `source_license` | enum | e.g. `CC-BY-4.0`, `CC-BY-NC`, `public-domain-US-gov`, `publisher-© facts-extracted` |
| `source_locator` | string | Table/figure/page the number came from |
| `disease_extent` | enum | `localized` \| `metastatic` \| `primary-refractory` \| `relapsed` \| `mixed/unspecified` |
| `primary_site` | enum | `skeletal-axial` \| `skeletal-appendicular` \| `extraskeletal` \| `pelvic` \| `mixed/unspecified` |
| `age_band` | enum | `pediatric(<15)` \| `AYA(15–39)` \| `adult(≥40)` \| `mixed/unspecified` |
| `risk_group` | string | As defined by source; original scheme recorded verbatim |
| `regimen_backbone` | enum + free | e.g. `VDC/IE`, `VIDE/VAI`, `VACD`, `other`, `unspecified` |
| `local_control` | enum | `surgery` \| `radiotherapy` \| `surgery+RT` \| `mixed/unspecified` |
| `n` | integer | Subgroup sample size (subject to min-cell rule) |
| `outcome_measure` | enum | `OS` \| `EFS` \| `RFS` \| `DFS` \| `LRFS` \| `PFS` \| `other` |
| `event_definition` | string | Verbatim/paraphrased definition of "event" |
| `timepoint_years` | number | e.g. 3, 5, 10 |
| `estimate_pct` | number | Point estimate |
| `ci_low_pct`,`ci_high_pct` | number | If reported |
| `estimate_method` | enum | `kaplan-meier` \| `crude-proportion` \| `other/unspecified` |
| `median_followup_months` | number | If reported |
| `accrual_period` | string | e.g. `2000–2008` (era) |
| `comparability_flags` | array | e.g. `non-standard-event-def`, `mixed-population`, `small-n` |
| `provenance` | object | `{ extractor, verifier, extraction_date, method, second_pass }` |
| `notes` | string | Caveats |

**Key decisions**

- **Record granularity = one reported outcome**, not one study, so comparability is explicit.
- **Never coerce non-comparable data into a shared bucket**; flag and preserve the source framing.
- **Facts-not-expression extraction** to respect copyright (numbers/definitions are facts; we cite,
  we don't redistribute copyrighted tables).
- **Schema-validated in CI**; provenance is a hard gate, not a nicety.
- **Pooled estimates are opt-in, labeled, and expert-reviewed** — the default dataset reports what
  studies reported.

## Data, licensing & compliance

**THIS SECTION LEADS. The cancer guardrails are binding and override convenience.**

**Aggregate-only, de-identified-only, open-access-only.**
- The project ingests **only** group-level, already-published aggregate statistics.
- **Controlled-access repositories (dbGaP, EGA), individual-level biobanks, and any
  individual-patient data (IPD) are OUT OF SCOPE.** No exceptions without authorized access + IRB,
  which is outside this project's charter.
- **No identifiable patient data.** A **minimum-cell-size rule** applies: any reported subgroup with
  `n` below a documented floor (default `n < 5`, configurable per review) is treated as potential
  re-identification risk and **excluded** even if published, and small subgroups are flagged.
- Case reports and small case series are excluded (validity + privacy).

**Per-source license register (gate before extraction).** Every source gets a row recording: access
status (open / paywalled-abstract-only / public-domain), license (CC-BY, CC-BY-NC, publisher ©,
US-gov PD), and what we are permitted to do. Extraction cannot start until the row is complete.

| Source | Typical license / status | Our use |
|---|---|---|
| PubMed Central Open Access subset (PMC-OA) | Per-article: CC-BY, CC-BY-NC, CC-BY-NC-ND, etc. | Read + extract facts; record per-article license; attribute |
| ClinicalTrials.gov (registry + results) | US Government — public domain | Extract aggregate results; cite NCT |
| SEER aggregate statistics / SEER*Explorer | US Government — public domain (aggregate) | Use aggregate incidence/survival; cite |
| GLOBOCAN / IARC aggregate | Open, attribution terms apply | Aggregate only; attribute per IARC terms |
| Cochrane / published meta-analyses | Publisher © | Extract facts (numbers) + cite; no verbatim table redistribution |
| Closed-access journal articles | Publisher © | Extract factual numbers + cite **only where lawfully accessible**; never redistribute copyrighted expression; prefer OA version when one exists |
| COSMIC / OncoKB | Non-commercial license | **Not used** (genomic; out of scope here) — listed to make the boundary explicit |

**Copyright stance.** Quantitative facts (a survival percentage, a sample size, an event definition)
are not themselves copyrightable; the *expression and arrangement* (prose, the specific table/figure
layout) is. We therefore **extract facts and cite them**; we **do not** copy-paste or redistribute
copyrighted tables/figures/prose. Short verbatim quotes are limited to what is necessary to pin down
a definition and are clearly quoted and attributed (fair-use-minimal). Where an open-access version
exists, we prefer it. If access to a number requires bypassing a paywall, we **do not** ingest it.

**Output licensing.** Compiled dataset: **CC-BY-4.0** (a database of facts with substantial curated
schema/provenance; CC-BY chosen for maximal reuse with attribution). Code/tooling: **MIT**. Data
dictionary & docs: **CC-BY-4.0**. We will additionally publish a CC0 statement for the *purely
factual* numeric values where appropriate, while the curated structure/annotations remain CC-BY.

**Provenance model.** Every record links to `{ DOI/PMID/NCT, source_locator (table/figure/page),
source_license, extractor, verifier, extraction_date, mapping_rule_version }`. A provenance
completeness check is a **CI-blocking gate**: no record ships without it.

**Privacy / PII.** None ingested by construction (aggregate-only + min-cell rule). A documented
intake checklist requires each source to be affirmatively classified as aggregate before extraction.

**Medical-advice stance.** The dataset is a research substrate, not guidance. The repository README,
dataset metadata, and any derived material carry a prominent **"Not medical advice"** notice.
Patient-facing education is a **separate high-risk deliverable** requiring oncologist +
patient-advocate sign-off (below).

## Quality, review & risk gates

**Risk tier:** **medium** for the harmonization dataset and tooling (domain accuracy required;
reviewer with sarcoma-oncology / clinical-epidemiology skill). **high** for any patient-facing
education derived from it (credentialed expert sign-off required before publication).

**Required review before a deed is "done":**
- **Data/extraction tasks (medium):** double extraction + adjudication; schema/provenance CI green;
  review by a reviewer with clinical-epidemiology or sarcoma-oncology literacy.
- **Harmonization/ontology tasks (medium):** mapping rules reviewed by domain reviewer; comparability
  flags audited; sign-off that no non-comparable data was silently merged.
- **Any pooled/meta estimate (high-sensitivity):** statistical method documented + reviewed by a
  biostatistician/epidemiologist; clearly labeled as a derived estimate.
- **Patient-facing explainer (high):** **oncologist sign-off + patient-advocate review**, "not
  medical advice" framing, every claim sourced — before merge/publication.

**Definition of Shipped (project-level):** A versioned, CC-BY dataset release where (1) 100% of
records pass schema + provenance + min-cell CI gates, (2) the audited sample meets ≥95% field
agreement with discrepancies adjudicated, (3) the data dictionary and provenance graph are published,
(4) the reproducible notebook runs clean from the released data, (5) a domain reviewer has signed the
release, and (6) — for any patient-facing layer — the high-risk review chain has signed off. A deed
is "delivered, not merged" only when a real beneficiary (partner researcher/advocate) can and does
use it, or for the education layer, when it reaches families through a vetted channel.

## Roadmap & milestones

**M0 — Foundation & cold-start (thin, no partner required).**
Goal: prove the schema and pipeline on a small, fully-open pilot.
- Define outcome ontology + data dictionary; stand up the record schema + CI validators (schema,
  provenance, min-cell, no-unsourced-number).
- Build the per-source license register and the source-intake/aggregate-only checklist.
- Pilot-extract a handful of landmark **open-access** ES outcome studies (double-extracted).
- **Exit criteria:** ≥3 landmark OA studies harmonized; 100% records pass all CI gates; data
  dictionary v0.1 published; pilot reviewed and signed by a domain reviewer; license register
  complete for all piloted sources.

**M1 — Corpus & extraction at scale.**
Goal: build the included-studies register and extract broadly.
- Pre-registered search strategy; screening against inclusion/exclusion; frozen study register.
- Extraction templates + double-extraction QA workflow; extract across localized/metastatic/relapsed.
- **Exit criteria:** ≥30 studies / ≥150 outcome records; ≥10% audited at ≥95% agreement, all
  discrepancies adjudicated; every source license-registered before extraction.

**M2 — Harmonization & normalization.**
Goal: make cross-study comparison explicit and defensible.
- Versioned mapping rules (era, regimen, extent, age, event definitions); comparability flags;
  optional, clearly-labeled pooled views (expert-reviewed).
- **Exit criteria:** ontology v1.0 + mapping rules reviewed; 100% records normalized + flagged; any
  pooled estimate documented and biostatistician-reviewed; non-comparability audit passes.

**M3 — Publication & access.**
Goal: release the open dataset and researcher tooling.
- Versioned release (semver + Zenodo DOI); data dictionary v1.0; provenance graph; reproducible
  notebook; contribution guide; "not medical advice" notices.
- **Exit criteria:** public CC-BY release; notebook runs clean from released artifacts; domain
  reviewer signs release; Definition of Shipped met for the dataset.

**M4 — Sustainability, partner & (gated) education.**
Goal: keep it alive and, if/when a partner and reviewers exist, enable family-facing education.
- Update cadence + maintenance docs; outcome tracking; partner handoff; **gated** high-risk explainer
  (oncologist + advocate signed) only if review chain + steward secured.
- **Exit criteria:** documented update process + named maintainer; ≥1 partner consuming the dataset
  (`verifiedNeed` → true); if pursued, ≥1 expert-signed explainer published.

## Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`**, organized by the M0–M4 milestones above
with per-milestone task tables (ID, type, size, risk, deliverable, dependencies, reviewer),
acceptance criteria for the most important tasks, a Definition of Done per milestone, a sized backlog,
and a complete schema-valid example Task JSON for the first M0 task.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the repo, schema, releases, and CI gates.
- **Domain reviewer(s) (medium tier):** clinical epidemiologist / sarcoma-oncology-literate reviewer
  on a rotation; reviews extraction, ontology, and releases.
- **Biostatistician/epidemiologist:** reviews any pooled/derived estimate.
- **Credentialed expert reviewers (high tier):** **pediatric/sarcoma oncologist + patient advocate**
  — required sign-off for any patient-facing education. TO BE SECURED.
- **Steward (last-mile owner):** ensures the dataset reaches a real beneficiary and that education,
  if any, reaches families through a vetted channel. TO BE SECURED.
- **Partner / requestor:** sarcoma research consortium or ES patient-advocacy foundation. TO BE
  SECURED (drives `verifiedNeed`).
- **Conflict-of-interest:** reviewers and partners disclose COI per Elyos governance; for-profit
  primary benefit is disqualifying.

## Dependencies & integrations

- **Data sources:** PMC-OA, ClinicalTrials.gov, SEER/SEER*Explorer (aggregate), GLOBOCAN/IARC,
  published meta-analyses/Cochrane (facts + citation only).
- **Publishing/archival:** Zenodo (dataset DOI + versioning), GitHub (repo, CI), CC-BY/MIT licenses.
- **Tooling:** TypeScript/ESM + pnpm (validators, build), JSON Schema (record validation), optional
  Python (pandas/lifelines) in `analysis/` for the reproducible notebook.
- **Elyos pieces:** Task schema (`packages/schema`), CLI workspace prep + PR flow (donated lane);
  governance/review process; this project depends on no funded-lane runner unless a funded extraction
  task is explicitly budgeted.
- **Upstream/sibling projects:** `ewing-open-data-catalog` and `ewing-literature-corpus` (shared
  source lists / corpus); `ewing-family-guide` / `ewing-info-translations` are downstream consumers
  of any education layer.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Patient-level / identifiable data ingested by mistake | Low | Critical | Aggregate-only intake checklist; min-cell-size rule; CI cannot pass with per-patient fields; documented exclusions | Maintainer |
| Copyright violation (redistributing tables/figures) | Medium | High | Facts-not-expression extraction; cite, don't copy; OA-preferred; license register gate | Maintainer + reviewer |
| Silent merging of non-comparable outcomes | Medium | High | One-row-per-outcome model; comparability flags; ontology review; non-comparability audit gate | Domain reviewer |
| Extraction error (wrong number/definition) | Medium | High | Double extraction + ≥10% audit; ≥95% agreement gate; provenance to source locator | Domain reviewer |
| Family-facing material misread as advice/prognosis | Medium | Critical | Education-layer gated as `high`; oncologist + advocate sign-off; "not medical advice"; no individualized output | Expert reviewers + steward |
| Pooled estimate over-interpreted as authoritative | Medium | High | Pooled views opt-in, labeled, method-documented, biostatistician-reviewed | Biostatistician |
| No partner / no real beneficiary (`verifiedNeed` stays false) | Medium | Medium | Proceed only as foundational data work; do not promote as guidance; active partner outreach; flip only on signed confirmation | Maintainer + steward |
| Source link rot / unverifiable provenance | Medium | Medium | Store DOI/PMID/NCT + locator + archived snapshot reference; provenance CI gate | Maintainer |
| Reviewer scarcity (sarcoma expertise is rare) | High | Medium | Build reviewer rotation early; partner with consortium; queue work that doesn't block on `high` review | Maintainer |
| Outdated data as new trials report | High | Medium | Documented update cadence; versioned releases; changelog | Maintainer |

## Security & privacy

- **Threat surface:** primarily *data-integrity and disclosure* risk, not classic appsec — the
  harm vector is wrong/unsourced numbers or accidental identifiability, not RCE.
- **PII / patient privacy:** none ingested by construction; aggregate-only + min-cell-size rule;
  exclusion of small case series; documented intake classification.
- **Secrets:** no API keys or tokens in logs, receipts, or commits (per CLAUDE.md). Public data
  sources need no secret credentials; if a funded extraction task is ever used, the runner enforces a
  hard budget cap and writes no secrets.
- **Abuse/misuse prevention:** prominent "not medical advice" framing; no individualized prognostic
  surface; refusal guardrails honored (no surveillance, no targeting, no for-profit primary benefit,
  no license/privacy violation). If a request tries to steer this toward individualized prognosis or
  controlled-access data, **stop and flag** rather than proceed.
- **Integrity:** CI gates (schema, provenance, min-cell, no-unsourced-number); content-addressed/
  versioned releases; changelog; double-extraction audit trail.

## Sustainability & maintenance

- **Maintenance owner:** the maintainer post-delivery; a documented update cadence (e.g., review new
  ES trial reports each quarter) and a contribution guide so external researchers can submit
  source-cited records that pass the same gates.
- **Outcome tracking:** track reuse (citations, forks, partner attestations), coverage growth, audit
  agreement over time, and the negative-harm metrics (zero patient-level ingestions, zero unsourced
  numbers). Publish a short "what's covered / what's missing" ledger per release.
- **Longevity:** Zenodo DOIs + versioned releases ensure the dataset persists and is citable even if
  the repo goes quiet; CC-BY/MIT ensure others can carry it forward.
- **Handoff:** if a partner consortium adopts it, transfer or co-maintain; document the steward's
  role for any education layer reaching families.

## Open questions

1. **Partner & steward:** which sarcoma consortium or ES foundation will commit to consuming the
   dataset (and provide the high-tier reviewers)? Until answered, `verifiedNeed=false`.
2. **Minimum-cell-size floor:** is `n<5` the right default, or should the domain reviewer set a
   different threshold for published aggregates?
3. **Pooled estimates:** does the project publish any labeled pooled views in v1, or strictly report
   per-study outcomes and leave pooling to downstream meta-analysts?
4. **Closed-access facts:** how aggressively do we include factual numbers from closed-access papers
   we can lawfully read? (Conservative default: prefer OA; cite facts; never redistribute expression.)
5. **Education layer:** do we attempt the high-risk family-facing explainer at all without a secured
   oncologist + advocate reviewer pair? (Default: no.)
6. **Funded lane:** is any extraction worth funding (metered API) with a hard budget cap, or is the
   donated lane sufficient?

## References

- Elyos `CLAUDE.md` (work rules, lanes, guardrails, quality bar).
- Elyos `docs/good-deed-definition.md` (5 criteria + risk tiers).
- Elyos `packages/schema/src/schemas.ts` (Task JSON schema).
- Elyos `planning/ROADMAP.md` (Track 8 cancer guardrails; `ewing-outcomes-harmonization` entry).
- Public data sources: PubMed Central Open Access subset; ClinicalTrials.gov; SEER / SEER*Explorer;
  GLOBOCAN / IARC; Cochrane Library (facts + citation).
- Landmark ES outcome literature to be enumerated in the M1 included-studies register (e.g.,
  INT-0091, AEWS0031 interval-compression, EICESS/Euro-EWING and rEECur reports) — cited per record
  with DOI/PMID; listed here as exemplars, not yet extracted.

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during drafting and have each been **applied**
to the plan above (and carried into `TASKS.md`). They are not aspirational — each points to concrete
text that exists in this document.

1. **Families-first framing added** — an explicit "For the families reading this" note opens the doc,
   setting the standard of care for every number. *(Applied: top block.)*
2. **Compliance section leads in spirit and is reinforced** — the executive summary states the binding
   guardrails up front and the Data/licensing/compliance section is the most detailed in the plan.
3. **Minimum-cell-size rule (default `n<5`) added** — converts the abstract "no identifiable data"
   rule into an enforceable, CI-checkable threshold against re-identification.
4. **"No unsourced number" CI gate added** — provenance completeness is a blocking gate, not advice.
5. **One-row-per-outcome data model** — chosen specifically so non-comparability is explicit and
   cannot be hidden, with `comparability_flags`.
6. **Facts-not-expression copyright stance** — explicit, conservative position on extracting facts
   while never redistributing copyrighted tables/figures; OA-preferred.
7. **Per-source license register as a pre-extraction gate** — extraction cannot start before the
   license row is complete; added as pipeline stage 2 and an M0 task.
8. **Negative success metric added** — zero patient-level ingestions / zero unsourced numbers is
   measured as an outcome, recognizing that "do no harm" is a deliverable here.
9. **Explicit separation of medium (dataset) vs high (education) risk tiers** — the harmonization work
   is medium; any patient-facing layer is high and separately gated.
10. **Event-definition captured as a first-class field** — the single biggest source of silent
    incomparability in ES outcomes is made explicit per record.
11. **Treatment-era + regimen-backbone fields** — encodes the VACD → VIDE/VAI → VDC/IE evolution so
    era effects aren't conflated.
12. **Disease-extent + primary-site + age-band normalization** — localized/metastatic/relapsed and
    axial/pelvic/extremity and pediatric/AYA/adult are structured, not free text.
13. **Pooled estimates made opt-in, labeled, and biostatistician-reviewed** — prevents the dataset
    from accidentally manufacturing meta-analytic conclusions.
14. **Double-extraction + ≥10% audit + ≥95% agreement** — concrete, checkable QA with adjudication
    logging, baked into Definition of Shipped and metrics.
15. **Honest `verifiedNeed=false` throughout** — no invented partner; explicit "TO BE SECURED" for
    partner, steward, and high-tier reviewers.
16. **Reviewer-scarcity risk surfaced** — sarcoma expertise is rare; mitigation queues non-`high`
    work so the backlog doesn't stall on expert availability.
17. **Zenodo DOI + semver versioning** — makes the dataset citable, persistent, and survivable beyond
    repo activity (sustainability).
18. **Reproducible researcher notebook (not patient-facing)** — demonstrates *correct* caveat-laden
    comparison and doubles as an extraction-error smoke test.
19. **Sibling-project integration** — explicitly links to `ewing-open-data-catalog`,
    `ewing-literature-corpus`, and downstream `ewing-family-guide`/`-info-translations`.
20. **"Not medical advice" notice required on README, dataset metadata, and all derivatives** — not
    just the explainer.
21. **Refusal/stop-and-flag behavior wired in** — Security & privacy and Non-goals state that attempts
    to steer toward individualized prognosis or controlled-access data are refused and flagged.
22. **COSMIC/OncoKB explicitly listed as NOT used** — makes the non-commercial-license boundary
    visible even though they're out of scope, preventing future scope creep.
23. **CC-BY-4.0 dataset / MIT code / CC0 for pure facts** — a deliberate, defensible licensing split
    matched to what is and isn't copyrightable.
24. **"What's covered / what's missing" ledger per release** — honest coverage transparency, mirroring
    the roadmap's atltuae-style coverage discipline.
25. **Pre-registered search + frozen included-studies register** — guards against cherry-picking
    studies and makes coverage auditable and reproducible.

---

## Review sign-off

**Reviewer:** Senior staff engineer + TPM (self-review pass), 2026-06-28.

**Completeness check.** All 17 required H2 sections present and in spec order: Executive summary;
Problem & beneficiaries; Goals and non-goals; Success metrics; Scope; Solution approach &
architecture; Data, licensing & compliance; Quality, review & risk gates; Roadmap & milestones; Work
breakdown; Governance, roles & stakeholders; Dependencies & integrations; Risks & mitigations;
Security & privacy; Sustainability & maintenance; Open questions; References. Metadata header present.
Appendix A lists 25 applied improvements. `TASKS.md` is the itemized backlog companion.

**Correctness check.**
- Guardrails: aggregate-only / open-access-only / no controlled-access / no identifiable data are
  stated, enforced via min-cell rule + intake checklist + CI gates, and match ROADMAP Track 8 and
  CLAUDE.md. ✔
- Risk tiers: dataset = medium, patient-facing education = high with oncologist + advocate sign-off,
  consistent with `good-deed-definition.md`. ✔
- Honesty: no fabricated partner/need; `verifiedNeed=false`; partner/steward/high-tier reviewers
  marked TO BE SECURED. ✔
- Licensing: per-source register, facts-not-expression stance, CC-BY/MIT outputs, COSMIC/OncoKB
  explicitly excluded. ✔
- Schema alignment: `TASKS.md` maps to every required Task field and includes a schema-valid example.
  Verified against `packages/schema/src/schemas.ts` (required fields, enums, funded→budget rule). ✔

**Fixes applied during review.**
- Tightened the `source_license` enum to include a `publisher-© facts-extracted` value so closed-access
  factual extraction is representable without implying redistribution rights.
- Added the explicit `comparability_flags` controlled values to the data model.
- Clarified that the reproducible notebook is researcher-facing and doubles as an extraction smoke
  test (avoids it being mistaken for a patient tool).
- Made the negative-harm metric (zero patient-level ingestions / zero unsourced numbers) appear in
  both Success metrics and Sustainability tracking for consistency.

**Sign-off:** Plan is internally consistent, schema-aligned, and compliant with Elyos cancer
guardrails. Ready for maintainer adoption as **v0.1.0 Draft**. Blocking items for promotion beyond a
foundational data effort: secure a named partner (`verifiedNeed`), a steward, and the oncologist +
patient-advocate reviewer pair before any patient-facing layer.
