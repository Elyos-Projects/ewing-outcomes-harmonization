# ewing-outcomes-harmonization

> Ewing Sarcoma (ES) outcomes are reported across decades of clinical trials and cohort studies, but the published numbers are **not directly comparable**. Studies differ in how they define an "event" (  ·  **Risk tier:** med  ·  **Status:** planning

Ewing Sarcoma (ES) outcomes are reported across decades of clinical trials and cohort studies, but the published numbers are **not directly comparable**. Studies differ in how they define an "event" (EFS vs. RFS vs. DFS), which disease extent and risk groups they include (localized vs. metastatic vs. relapsed; axial vs. appendicular primary), which chemotherapy backbone and local-control modality they used, the treatment era and follow-up duration, and how they computed and reported confidence intervals. A 5-year EFS of "73%" in one paper and "68%" in another may be measuring different things on different populations. This heterogeneity slows meta-analysis, muddies advocacy, and produces family-facing materials that quote a single scary or rosy number out of context.

**Definition of shipped:** records pass schema + provenance + min-cell CI gates, (2) the audited sample meets ≥95% field agreement with discrepancies adjudicated, (3) the data dictionary and provenance graph are published, (4) the reproducible notebook runs clean from the released data, (5) a domain review

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/ewing-outcomes-harmonization --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
