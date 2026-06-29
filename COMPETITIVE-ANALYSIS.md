# Competitive & Improvement Analysis — `ewing-outcomes-harmonization`

> Scope reviewed: `PLAN.md` v0.1.0 (2026-06-28) + `TASKS.md` (M0–M2 read in full, M3–M4 summarized).
> Project = harmonizing **published, aggregate** Ewing Sarcoma (ES) outcome statistics (survival/response
> rates from publications, registries) into a provenance-tracked, comparable dataset. **No patient-level
> data, no controlled-access, no clinical conclusions.** Cancer guardrails are binding.

Analyst note on grounding: competitor and method claims below are cited to real sources retrieved via web
search. Numeric examples (e.g., AEWS0031 5-yr EFS 73% vs 65%) are quoted to illustrate the comparability
problem, not endorsed as canonical.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on compliance, honesty (`verifiedNeed=false`), and the one-row-per-outcome
model. It is genuinely above the bar for a cancer-data project. The gaps below are about **methodological
rigor of harmonization**, not safety framing.

### 1.1 Strengths to preserve
- One-row-per-reported-outcome granularity with `comparability_flags` is the right call — it makes
  incomparability explicit rather than averaging it away.
- `event_definition` as a first-class verbatim field directly targets the single biggest source of silent
  incomparability in ES (EFS vs RFS vs DFS vs PFS; what counts as an "event"). Correct and well-motivated.
- Facts-not-expression copyright stance + per-source license register as a pre-extraction gate is defensible
  and enforceable.
- Negative success metric (zero patient-level ingestions, zero unsourced numbers) is exactly right for the
  domain.

### 1.2 Material gaps / errors / weak spots

**(a) No risk-of-bias / study-quality instrument is specified.** The plan captures provenance and
comparability flags but never names a risk-of-bias (RoB) or quality framework. For a substrate meant to feed
meta-research, every included study needs a structured quality/RoB assessment (e.g., RoB 2 for RCTs, ROBINS-I
for non-randomized cohorts, QUIPS for the many prognostic-factor studies, and Newcastle-Ottawa for
registry/cohort work). Without this, a downstream meta-analyst still has to redo the most contentious step.
This is the largest single omission. The `comparability_flags` enum should be complemented by a separate,
instrument-based `risk_of_bias` object.

**(b) Kaplan–Meier reconstruction (Guyot) is implied by the domain but not addressed at all.** Many ES papers
report only a KM curve, a median, and numbers-at-risk — not a clean "5-yr EFS = X% [CI]". The plan's record
schema assumes a tidy point estimate exists. In practice extractors will face the choice: (i) skip
curve-only studies (coverage loss, selection bias toward studies that happen to tabulate), or (ii) digitize
curves. If curve digitization is ever used, the Guyot algorithm reconstructs pseudo-IPD from published KM
curves ([Guyot 2012, BMC Med Res Methodol](https://pmc.ncbi.nlm.nih.gov/articles/PMC5796634/);
[PubMed 22297116](https://pubmed.ncbi.nlm.nih.gov/22297116/)) — but it has real limits: error from curve
publication quality and digitization clicks, sensitivity to whether numbers-at-risk and censoring marks are
reported, and degraded accuracy at small n (the method is noticeably better than alternatives at n≈100 but
all methods need n≈500 for reliable mean survival;
[Rogula 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC8808036/)). **Critical tension:** reconstructed
pseudo-IPD is *derived individual-level data*, which sits uncomfortably against the "aggregate-only / no IPD"
guardrail. The plan must take an explicit position: either (1) ban curve reconstruction entirely and accept
the coverage gap (cleaner for guardrails), or (2) permit reading-off survival probabilities at fixed
timepoints from a curve as an *aggregate* extraction (a single % at t=5y is aggregate), while explicitly
forbidding full pseudo-IPD reconstruction. Right now this is undefined and an extractor will improvise.

**(c) Double-counting / overlapping cohorts is not handled.** ES is rare; the same patients recur across
publications — e.g., a primary trial report, a long-term follow-up, a pooled Euro-EWING/COG analysis, and a
SEER study can all describe overlapping individuals. AEWS0031 alone has a primary 2012 report and a 2023
10-year follow-up ([JCO 2012](https://ascopubs.org/doi/10.1200/JCO.2011.41.5703);
[PubMed 37651654](https://pubmed.ncbi.nlm.nih.gov/37651654/)). The schema has `study_id` but no concept of a
**cohort/patient-population identity** (trial ID, accrual window, enrolling group) to let a meta-analyst
detect and avoid pooling overlapping populations. Add a `cohort_id` / `population_lineage` field and a
"supersedes / extends" relation between records. Without it the dataset actively enables double-counting —
the opposite of its purpose.

**(d) SEER and registry data are conflated with trial data without an explicit data-provenance type.**
Population-based SEER survival (e.g., pediatric 5-yr ~73.9%, localized vs metastatic ~84.7% vs ~50.4%;
[SEER cohort study](https://pmc.ncbi.nlm.nih.gov/articles/PMC9879550/)) measures something fundamentally
different from per-protocol trial EFS: different denominators (all-comers vs eligible-and-consented), staging
captured differently, no standardized event definition, and observed (not cause-specific unless stated)
survival. The schema needs a `study_design`/`data_source_type` enum (population-registry / prospective-trial
/ single-institution-cohort / meta-analysis) as a **first-class comparability axis**, not buried in free-text
metadata. Comparing SEER OS to trial EFS is a classic apples-to-oranges trap the dataset should structurally
prevent.

**(e) Staging-system and era normalization is asserted but under-specified.** The plan lists era/regimen/
extent fields but there is no canonical crosswalk. ES has shifting risk schemes (tumor volume thresholds,
histologic necrosis response %, metastatic burden, more recently molecular/ctDNA), and "localized" in a
1990s VACD trial is not the same selected population as "localized" post-PET/MRI staging. Mapping rules
(M2/eoh-mapping-201) need a documented, versioned crosswalk *with the limits of each mapping stated* — and
records that cannot be mapped must stay flagged, not coerced. The plan says this in principle ("never
coerce") but provides no concrete crosswalk artifact or its validation.

**(f) "≥95% field-level agreement" metric is undefined and possibly easy to game.** Agreement on what
denominator? Counting trivially-objective fields (DOI, year) inflates agreement; the fields that matter
(event_definition normalization, disease_extent classification, which CI was reported) are exactly the
hard-to-agree ones. Specify per-field agreement (Cohen's/Gwet's kappa for categorical fields, not raw %),
report the hard fields separately, and treat numeric fields (estimate_pct, n) as exact-match. Raw percent
agreement on a mix of easy and hard fields is a weak metric.

**(g) No handling of competing risks / what the % actually denotes.** A "5-yr OS of 70%" can be KM
net-survival, observed survival, or cause-specific; relapse and second-malignancy are competing risks in ES.
The schema has `estimate_method` (kaplan-meier / crude) but not a `survival_type` (overall vs cause-specific
vs disease-specific vs relative/net). SEER routinely reports cause-specific and relative survival; trials
report OS/EFS. This must be a captured field or the dataset will silently mix incompatible quantities.

**(h) "Comparability_flags" is a flat array, but comparability is multi-dimensional and pairwise.** Two
records are comparable or not *relative to each other and to a question*. A flat per-record flag list cannot
express "comparable to record X on extent and era but not on event definition." Consider, at minimum, a
controlled, exhaustive flag vocabulary plus documentation that comparability is a downstream join decision —
and that the dataset does **not** assert any two records are poolable. The plan should explicitly state the
dataset makes *no* comparability assertions, only descriptive flags.

**(i) CI / variance reconstruction is unaddressed.** Many older ES papers give a point estimate with no CI.
Meta-analysis needs a variance/standard error. The plan should state whether it (a) leaves CI null (honest,
limits downstream pooling) or (b) permits documented SE estimation from n and event counts — and if (b),
that it is flagged as derived. Currently silent.

**(j) Pre-registration is named but PROSPERO/OSF registration is not committed.** M1 says "pre-registered
search strategy" but does not say *where* it is registered or that the protocol follows PRISMA / PRISMA-IPD
reporting. For credibility with the very researchers who are the beneficiaries, register the protocol
publicly (PROSPERO or OSF) and follow PRISMA-2020 flow reporting. This is cheap and high-trust.

**(k) Minimum-cell-size rule (`n<5`) is reasonable for privacy but conflicts with rare-disease coverage and
is mis-targeted.** For *aggregate published* rates, re-identification risk from a published n=4 subgroup rate
is low (the publisher already cleared it), while excluding small subgroups disproportionately drops the
metastatic/relapsed/rare-site strata that are the whole point. The privacy rationale for excluding published
aggregates is weak; the *statistical-validity* rationale (unstable rates at small n) is strong. Reframe the
rule as a **statistical-reliability flag** (`small-n`, already present) rather than a privacy-driven hard
exclusion, and let the domain reviewer set the floor. As written, Open Question #2 already hints at this; the
plan's privacy justification for the exclusion is the weaker of the two arguments.

**(l) Clinical-conclusion firewall is stated in prose but not enforced in schema/CI.** The plan forbids
clinical conclusions, but there is no machine-checkable guard (e.g., the dataset has no "recommended
regimen" or "best treatment" field — good — but the *notebook* could drift into implying superiority). Add an
explicit CI/lint check or review checklist item that the notebook's narrative contains no comparative
effectiveness claim and every figure carries the caveat banner.

**(m) Success metric "≥3 independent reuses in 12 mo" with no secured partner is optimistic.** Honest, but
the dependency chain (no partner → reviewer scarcity → adoption) means the headline outcome metric is at
risk. Consider an interim leading indicator (e.g., dataset registered, DOI minted, ≥1 consortium in active
conversation) so progress is measurable before the lagging "reuse" outcome lands.

### 1.3 Smaller issues
- `regimen_backbone` enum lists VDC/IE, VIDE/VAI, VACD — but omits the relapse/refractory regimens (TC, IT,
  GD, high-dose ifosfamide) now standardized by rEECur ([UCL rEECur](https://www.ucl.ac.uk/medical-sciences/divisions/cancer/centres-and-networks/euro-ewing-consortium/clinical-trials/reecur)).
  If relapsed disease is in scope (it is, per M1/eoh-extract-104), the enum is incomplete.
- No field for **response rate / RECIST** despite the brief naming "response rates." Schema is survival-centric
  (`outcome_measure` = OS/EFS/RFS/DFS/LRFS/PFS) and lacks ORR/CR/PR. rEECur's primary endpoints include
  objective response — uncapturable today.
- `accrual_period` as free string ("2000–2008") will fragment; make it structured start/end years for
  era-binning.
- No explicit treatment of **abstract-only / conference data** (much ES long-term data first appears as ASCO
  abstracts, e.g., AEWS0031 10-yr) — include/exclude rule and a `publication_status` field needed.

---

## 2. Competitive landscape

There is **no existing open, living, provenance-tracked aggregate-ES-outcomes dataset** — the plan's
"baseline = 0" claim is essentially correct. But there is a dense field of adjacent resources the project
must position against; each solves part of the problem and none solves the whole.

### A. Population-based aggregate outcomes — SEER / SEER\*Explorer
- **What:** US population cancer registry; SEER\*Explorer and SEER\*Stat give aggregate incidence/survival
  by stage, site, age. ES studies routinely mine it (e.g., pediatric 5-yr ~73.9%; localized vs metastatic
  ~84.7% vs ~50.4%; pelvic tumors lowest OS). Public domain.
  ([SEER cohort](https://pmc.ncbi.nlm.nih.gov/articles/PMC9879550/);
  [SEER prognostic factors](https://pubmed.ncbi.nlm.nih.gov/25595632/))
- **Strengths:** Authoritative, large-n, public domain, standardized, already aggregate.
- **Gaps:** Registry-only (no per-protocol trial outcomes, no chemotherapy backbone, no histologic response);
  US-only; observed/cause-specific survival not EFS; staging coarse; not harmonized *against* trial
  literature. It is one **input** to this project, not a competitor for the harmonization angle.

### B. Published ES meta-analyses & systematic reviews
- **What:** Topic-specific syntheses — EFS/OS surrogate-correlation meta-analysis
  ([BMC Cancer 2020](https://link.springer.com/article/10.1186/s12885-020-06871-9);
  [PMC7201711](https://pmc.ncbi.nlm.nih.gov/articles/PMC7201711/)); prognostic-factor systematic review
  ([ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0960740418301786)); local-control and
  delayed-local-therapy reviews ([pelvis review](https://pmc.ncbi.nlm.nih.gov/articles/PMC12390909/)).
- **Strengths:** Peer-reviewed, expert-interpreted, answer specific clinical questions.
- **Gaps:** Static (frozen at publication), narrow PICO each, extraction not released as reusable structured
  data, no shared provenance graph, redundant re-extraction across reviews, paywalled. Exactly the
  "re-extract the same numbers repeatedly" pain the plan names.

### C. Trial literature & consortia (INT-0091, AEWS0031, EURO-EWING, rEECur)
- **What:** The primary outcome sources. AEWS0031 interval-compression (5-yr EFS 73% vs 65%; 10-yr 70% vs
  61%) ([JCO 2012](https://ascopubs.org/doi/10.1200/JCO.2011.41.5703);
  [PubMed 37651654](https://pubmed.ncbi.nlm.nih.gov/37651654/)); rEECur, the largest relapsed/refractory RCT
  (>650 pts, 17 countries; IT arm RR 20%, mPFS 4.7mo, mOS 13.9mo)
  ([ASCO](https://www.asco.org/abstracts-presentations/ABSTRACT297607);
  [UCL](https://www.ucl.ac.uk/medical-sciences/divisions/cancer/centres-and-networks/euro-ewing-consortium/clinical-trials/reecur)).
- **Strengths:** Gold-standard primary data, define the regimens/eras the project must encode.
- **Gaps:** Each reports its own way; consortia hold IPD privately (controlled access — out of scope here);
  no cross-trial harmonized public table. These are sources, and the consortia are also the *ideal partners*.

### D. ClinicalTrials.gov results DB + AACT
- **What:** Structured registry of trial protocols + summary results (participant flow, outcome measures,
  adverse events); AACT gives daily bulk download of the whole registry (CTTI/Duke)
  ([data-api](https://clinicaltrials.gov/data-api); [AACT](https://aact.ctti-clinicaltrials.org/points_to_consider)).
  US-government public domain.
- **Strengths:** Already-structured aggregate outcomes, API/bulk access, public domain, machine-readable.
- **Gaps:** Coverage and quality of posted results are uneven (many trials report results only in journals,
  not on CTG); no ES-specific harmonization; no event-definition normalization; outcome measures as free
  text. A strong **automated feed** into this project, not a substitute.

### E. Vivli (and similar IPD-sharing platforms — YODA, CSDR)
- **What:** Brokered sharing of de-identified **individual** participant data from completed trials
  ([Vivli](https://vivli.org/about/overview/)).
- **Strengths:** Enables true IPD meta-analysis (the most powerful synthesis).
- **Gaps / boundary:** Individual-level + controlled/gated access = **explicitly out of scope** for this
  project. Vivli is the thing this project is *not* — and that contrast is a feature: the project delivers an
  open aggregate layer for the 99% of researchers who will never get Vivli access for ES.

### F. R2 Genomics Analysis & Visualization Platform
- **What:** Web platform, 300k+ genomics profiles, dedicated ES module, KM survival scans over expression
  data ([R2 EWS](https://hgserver1.amc.nl/cgi-bin/r2/main.cgi?dscope=EWS&option=main)).
- **Strengths:** Mature, widely cited, interactive, free.
- **Gaps:** Genomics/expression-centric; survival is gene-expression-linked, not a harmonized clinical-
  outcomes registry across the trial literature. Adjacent, not overlapping (and explicitly the
  `ewing-expression-reanalysis` sibling's territory, not this one's).

### G. Meta-analysis tooling — metafor, `meta`, RevMan
- **What:** `metafor` (2.1M downloads) and `meta` in R, plus Cochrane's RevMan, are the synthesis engines
  ([metafor CRAN](https://cran.r-project.org/package=metafor);
  [tutorial](https://pmc.ncbi.nlm.nih.gov/articles/PMC10231495/)).
- **Strengths:** Validated statistics, heterogeneity (I², τ²), forest/funnel plots, the standard of the field.
- **Gaps:** They consume *already-extracted* tidy data; they do nothing for the upstream extraction,
  provenance, and harmonization that is this project's core. Complement, not competitor — the project should
  output data that drops straight into metafor.

### H. Systematic-review extraction tooling — Covidence, SRDR+, RevMan
- **What:** Screening + extraction workflow managers; SRDR+ is a free open archive of extracted SR data
  ([Covidence](https://www.covidence.org/blog/how-to-extract-study-data-for-your-systematic-review/); SRDR+).
- **Strengths:** Mature dual-extraction/adjudication workflows; SRDR+ is genuinely an open data archive.
- **Gaps:** Generic (not ES- or outcomes-ontology-aware), project-scoped (data siloed per review), no
  living/versioned provenance graph, no domain comparability model. SRDR+ is the closest "open extracted
  data" analog and worth studying as prior art — but it is not a curated, harmonized, living ES outcomes
  resource.

### I. KM-reconstruction / pseudo-IPD methods (Guyot; RESOLVE-IPD)
- **What:** Methods to recover pseudo-IPD from published curves
  ([Guyot 2012](https://pmc.ncbi.nlm.nih.gov/articles/PMC5796634/);
  [RESOLVE-IPD 2025](https://arxiv.org/pdf/2511.01785)).
- **Strengths:** Unlock pooling from curve-only papers.
- **Gaps / boundary:** Produce *individual-level* pseudo-data — a guardrail edge the project must rule on
  (see 1.2b). Relevant as a method to explicitly *bound*, not adopt wholesale.

**Bottom line:** SEER, the meta-analyses, and the trial/consortium literature are **sources**; metafor,
Covidence/SRDR+, RevMan, Vivli, R2 are **adjacent tools/platforms**. The open niche — a living, open,
provenance-per-number, comparability-aware *aggregate* outcomes substrate for one rare cancer — is genuinely
unoccupied.

---

## 3. Gaps we can fill

1. **The missing "tidy, sourced, comparable" layer between papers and metafor.** Everyone re-extracts; nobody
   publishes the extraction as a reusable, versioned, provenance-complete artifact. This project is that
   layer for ES.
2. **Provenance-per-number, not per-study.** SRDR+ and meta-analyses cite at the study level; this project
   cites down to table/figure + the exact extracted value + license + extractor/verifier. That granularity is
   the differentiator and is reusable.
3. **A living, versioned resource** (Zenodo DOI + semver + changelog) vs frozen-at-publication reviews. New
   trials (rEECur arms, EE2012 follow-ups) land continuously; a living dataset captures them.
4. **Explicit comparability scaffolding** (event_definition, design type, survival type, era, extent,
   response vs survival) so a downstream analyst can *filter to comparable subsets* instead of trusting a
   prose claim — something no current resource offers in machine-readable form.
5. **Bridging registry (SEER) and trial outcomes in one schema** with the design-type axis that keeps them
   from being naively pooled — a structural guardrail against a common error.
6. **An open, rare-disease-specific outcomes commons** that the very small ES research community can
   co-maintain (contribution guide + same CI gates), turning duplicated effort into shared infrastructure.
7. **A reproducible, caveat-first analysis notebook** as a teaching artifact for "how to (not) compare ES
   survival numbers" — useful even before broad coverage.

---

## 4. Differentiators to win

1. **Provenance as a hard CI gate, per number.** "No unsourced number ships" enforced in CI is a trust
   primitive no meta-analysis or registry offers. This is the single strongest differentiator.
2. **Comparability is structural, not editorial.** The dataset asserts *flags*, never *poolability* — letting
   researchers make defensible joins. This is honest in a way static reviews can't be.
3. **Living + citable + open (CC-BY/MIT + Zenodo DOI).** Persists and stays current; survivable beyond repo
   activity; drops straight into `metafor`/`meta`.
4. **Rare-disease focus done deeply.** Generic tools (Covidence, SRDR+) are broad and shallow on ES;
   consortia are deep but closed. This is deep *and* open for one disease.
5. **Guardrail-native design as a credibility signal.** Aggregate-only, min-cell/small-n flagging, "not
   medical advice," expert-gated education — the discipline itself is a trust differentiator with clinicians
   and advocates wary of careless data projects.
6. **Generalizable engine.** The schema/pipeline is a template for *any* rare cancer (see §7), so winning ES
   establishes a reusable platform, not a one-off.

---

## 5. Claude API leverage

Evidence base: collaborative two-LLM extraction (GPT-4-turbo + Claude-3-Opus) reached ~0.94–0.97 accuracy on
SR data extraction, **outperforming either model alone**, and explicitly mimics the two-reviewer process
([Collaborative LLMs, medRxiv/PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11469465/)); single-model
accuracy is lower and task-dependent (Claude > ChatGPT overall in one sleep-medicine study at ~71.5%; GPT-4
ahead on pure extraction in another) — i.e., **LLM extraction is an assistive first pass requiring human
verification, never an authority.** That maps perfectly onto this project's double-extraction model.

### Where Claude clearly helps (assistive, human-verified)
1. **First-pass structured extraction** of outcome numbers + study metadata from OA full text into the record
   schema, returning *every value with its source locator* (table/figure/page) and a verbatim quote for the
   event definition. Claude becomes "extractor A"; a human is "verifier B" — fitting the plan's
   double-extraction QA and matching the collaborative-LLM evidence.
2. **Normalizing endpoint definitions.** Map heterogeneous "EFS/RFS/DFS/PFS/LRFS" prose to the ontology and
   *surface the verbatim event definition* so the human can confirm the mapping — directly attacks the #1
   incomparability source. Claude proposes; reviewer disposes.
3. **Flagging comparability issues.** Claude can auto-suggest `comparability_flags` (mixed population,
   non-standard event def, registry-vs-trial, era mismatch, small-n) and explain *why*, giving the reviewer a
   checklist rather than a blank slate.
4. **Drafting risk-of-bias / quality assessments** (RoB 2 / ROBINS-I / QUIPS items) as a structured draft
   with quoted justification per signaling question — human finalizes. Closes gap §1.2a cheaply.
5. **Triage & screening assist.** Title/abstract pre-screening against inclusion/exclusion criteria, and
   detecting likely cohort overlap (same trial ID/accrual window/center) to flag double-counting candidates
   (gap §1.2c).
6. **License/access classification draft** of each source (OA vs publisher © vs US-gov PD) for the register —
   proposed, then human-confirmed.
7. **Consistency linting** of the dataset (e.g., CI bounds straddling the estimate, timepoint vs
   median-follow-up implausibility, unit/percentage errors) as natural-language QA.
8. **Drafting the data dictionary, mapping-rule docs, and caveat language** for the notebook and README.

### Where Claude must NOT decide (hard boundaries)
- **No fabricated or inferred numbers.** Every extracted value must trace to a source locator; if the paper
  doesn't state it, the field is null — Claude must not "estimate" a missing rate or CI. CI-enforce
  source_locator presence.
- **No final extraction authority.** Every Claude-extracted value requires independent human verification
  against the cited table/figure (the evidence shows LLM extraction errs; collaboration/human check is what
  delivers ~0.97).
- **No statistical synthesis / pooling.** Heterogeneity, pooled estimates, and forest plots go through proper
  tools (`metafor`/`meta`) and a biostatistician/epidemiologist — never an LLM "average." Claude may *draft
  method text*, not compute the pooled result as truth.
- **No clinical conclusions or comparative-effectiveness claims.** Claude must not say one regimen is "better";
  it extracts what was reported. Notebook narrative reviewed against this.
- **No license/copyright final call.** Claude drafts the license classification; a human verifies before any
  source enters extraction (legal risk).
- **No KM-curve reading-off without human verification**, and only within whatever the maintainer rules in
  §1.2b — Claude must not generate pseudo-IPD.

---

## 6. Ten concrete optimizations

1. **Add a structured `risk_of_bias` object** with an instrument tag (RoB 2 / ROBINS-I / QUIPS / Newcastle-
   Ottawa) and per-item judgments. Biggest rigor gap; cheap to add.
2. **Add `data_source_type` / `study_design` and `survival_type` as first-class enums** (registry / trial /
   single-institution / meta-analysis; OS / cause-specific / disease-specific / relative-net) to prevent
   SEER-vs-trial and net-vs-observed mixing (gaps §1.2d, §1.2g).
3. **Add `cohort_id` / `population_lineage` + a "supersedes/extends" record relation** so overlapping cohorts
   and follow-up reports are detectable; prevents double-counting (gap §1.2c).
4. **Take an explicit, documented stance on KM-curve extraction** (ban full reconstruction; permit
   human-verified read-off of a single timepoint % as aggregate, flagged `from-curve`). Resolves the
   guardrail tension (gap §1.2b).
5. **Replace raw "≥95% agreement" with per-field reliability** (kappa for categorical, exact-match for
   numeric, hard fields reported separately). Stronger, less gameable QA (gap §1.2f).
6. **Extend `outcome_measure` to include response endpoints** (ORR / CR / PR / clinical-benefit-rate) and add
   relapse/refractory regimens (TC, IT, GD, hi-dose IFOS) to `regimen_backbone` so rEECur-era data fits
   (gaps §1.3).
7. **Publicly register the protocol (PROSPERO or OSF) and follow PRISMA-2020 flow reporting**; publish the
   PRISMA diagram per release. High-trust, low-cost (gap §1.2j).
8. **Build a structured ClinicalTrials.gov/AACT ingest** for trials with posted results (public-domain,
   already-structured outcome_measures) to seed records with strong provenance and reduce manual extraction
   ([AACT](https://aact.ctti-clinicaltrials.org/points_to_consider)).
9. **Reframe the `n<5` privacy exclusion as a `small-n` reliability flag** with reviewer-set floor, so rare
   metastatic/relapsed strata aren't structurally dropped (gap §1.2k).
10. **Ship a comparability-aware query/export helper** (e.g., a function that, given an outcome question,
    returns only records sharing design type / extent / event definition and *refuses* to auto-pool) plus a
    direct `metafor`-ready export — operationalizing the project's core value and feeding the standard tool.

---

## 7. Parallel & perpendicular spin-offs

- **`ewing-research-landscape`** (sibling): the harmonized outcomes dataset is the quantitative spine of a
  living "state of ES evidence" map (which questions are answered, which strata are thin). Two-way feed.
- **`ewing-trial-finder`**: link `cohort_id`/NCT records to active trials (rEECur arms, EE2012) so a family/
  clinician sees outcomes context *and* open trials — education-gated, not advice.
- **`systematic-review-assist`**: generalize the Claude extractor + provenance CI + double-extraction harness
  into a reusable, disease-agnostic SR data-extraction toolkit (competes/complements Covidence/SRDR+ but with
  provenance-per-number and LLM-assist baked in).
- **`incidence-explorers`**: pair outcomes with SEER/GLOBOCAN incidence into an open ES burden+outcomes
  dashboard (aggregate, public-domain inputs).
- **Generalized `aggregate-outcomes-harmonization` engine for rare cancers**: the schema, ontology pattern,
  CI gates, and license register are disease-neutral. Rare cancers (osteosarcoma, rhabdomyosarcoma,
  neuroblastoma, DSRCT) share the exact pain. ES is the reference implementation; the engine is the platform
  play.
- **MCP server** exposing the dataset (query by extent/era/regimen/endpoint, fetch provenance, export to
  metafor) so any agent/researcher can pull *sourced, comparability-flagged* ES outcome numbers
  programmatically — with a hard "no synthesis / not medical advice" contract. Natural fit for Claude/agent
  ecosystems and a clean distribution channel.
- **Perpendicular: a "reporting-quality" advocacy artifact** — the dataset's own coverage/heterogeneity
  ledger becomes evidence for *better outcome-reporting standards* in ES trials (advocacy beneficiary #2),
  e.g., arguing for standardized EFS event definitions.

---

## 8. Open questions for the maintainer

1. **KM-curve policy (decisive):** Are curve-only studies excluded, or is single-timepoint read-off allowed
   as aggregate? Is any pseudo-IPD (Guyot) ever permitted, or hard-banned by the no-IPD guardrail? This
   determines coverage *and* guardrail posture and is currently undefined.
2. **Risk-of-bias instrument:** Will the project adopt RoB 2 / ROBINS-I / QUIPS, and is reviewer bandwidth
   available to apply them (given the acknowledged sarcoma-reviewer scarcity)?
3. **Registry vs trial scope:** Is SEER/population data in or out of the core dataset, or a clearly separated
   companion table? Mixing them in one file invites the apples-to-oranges error even with a design-type flag.
4. **Response endpoints:** Is relapsed/refractory *response* (rEECur-style ORR/PFS) in scope v1, or strictly
   survival? Affects schema and regimen vocabulary now.
5. **Double-counting policy:** How will overlapping cohorts be represented and surfaced to downstream users —
   is the project willing to assert "supersedes" relations, or only flag and leave it to the analyst?
6. **Pooling stance (Open Q#3 sharpened):** Given the strong field of existing topic meta-analyses, does
   publishing *any* pooled view add value, or does the project win precisely by being the **un-pooled,
   comparability-flagged substrate** that those meta-analysts consume?
7. **Partner sequencing:** Euro-Ewing Consortium / COG-adjacent groups / a sarcoma foundation are the natural
   partners *and* reviewer sources *and* the holders of the trial data — is outreach to one of them the true
   M0 critical path, ahead of broad extraction?
8. **`n<5` rule:** Privacy exclusion or statistical-reliability flag? (Recommend the latter for a rare
   disease.)

---

### Source list (key URLs cited)
- SEER ES survival: https://pmc.ncbi.nlm.nih.gov/articles/PMC9879550/ · https://pubmed.ncbi.nlm.nih.gov/25595632/
- EFS/OS surrogate meta-analysis: https://link.springer.com/article/10.1186/s12885-020-06871-9 · https://pmc.ncbi.nlm.nih.gov/articles/PMC7201711/
- Prognostic-factor systematic review: https://www.sciencedirect.com/science/article/pii/S0960740418301786
- Local-therapy systematic review (pelvis): https://pmc.ncbi.nlm.nih.gov/articles/PMC12390909/
- AEWS0031: https://ascopubs.org/doi/10.1200/JCO.2011.41.5703 · https://pubmed.ncbi.nlm.nih.gov/37651654/
- rEECur / Euro-Ewing: https://www.ucl.ac.uk/medical-sciences/divisions/cancer/centres-and-networks/euro-ewing-consortium/clinical-trials/reecur · https://www.asco.org/abstracts-presentations/ABSTRACT297607
- ClinicalTrials.gov / AACT: https://clinicaltrials.gov/data-api · https://aact.ctti-clinicaltrials.org/points_to_consider
- Vivli: https://vivli.org/about/overview/
- R2 platform (ES): https://hgserver1.amc.nl/cgi-bin/r2/main.cgi?dscope=EWS&option=main
- metafor / meta-analysis tooling: https://cran.r-project.org/package=metafor · https://pmc.ncbi.nlm.nih.gov/articles/PMC10231495/
- Covidence / SRDR+ extraction tooling: https://www.covidence.org/blog/how-to-extract-study-data-for-your-systematic-review/
- Guyot KM reconstruction: https://pmc.ncbi.nlm.nih.gov/articles/PMC5796634/ · https://pubmed.ncbi.nlm.nih.gov/22297116/ · https://pmc.ncbi.nlm.nih.gov/articles/PMC8808036/ · https://arxiv.org/pdf/2511.01785
- LLM extraction accuracy: https://pmc.ncbi.nlm.nih.gov/articles/PMC11469465/
