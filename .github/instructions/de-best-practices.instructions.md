---
applyTo: "**"
---


# EDW & dbt Modeling Best Practices (Data Scientists + Data Engineers)

This document summarizes the Confluence page **EDW & dbt Modeling Guidelines** for practical use in this repository.

- Source: https://theknotww.atlassian.net/wiki/spaces/DATATEAM/pages/5085036672/EDW+dbt+Modeling+Guidelines
- Scope: standards for organizing, naming, modeling, and governing dbt artifacts in `tkww-dbt`
- Audience: Data Scientists (DS) and Data Engineers (DE)

> Note: This summary is based on the indexed Confluence content available from tooling. It is designed to be actionable for day-to-day work and PR reviews.

## 1) Ownership and Scope

### Data Engineering (EDW-owned)
DE owns EDW layers and enterprise dimensional artifacts:
- `models/1_ods`
- `models/2_intermediate`
- `models/3_dimensional`

DE-owned dimensional outputs (facts/dimensions) should follow enterprise conventions and be stable for downstream consumption.

### Data Science (DS-owned)
DS-owned modeling logic should live under Data Science folders (by pillar/business area), not in EDW core layers.

Guideline reflected in migration docs referencing this standard:
- DS logic belongs in `/models/data_science/<pillar>` (or equivalent repo DS path conventions)
- Avoid introducing DS-specific reporting logic into `1_ods`, `2_intermediate`, or `3_dimensional`

## 2) Layering Principles

Keep transformations in clear layers with single-purpose models:

- **ODS/Staging (close to source):** flatten/semi-structured cleanup, dedupe, rename, type-cast, and lightweight standardization.
- **Intermediate:** reusable business transformation steps, decomposition of complex logic, and DRY refactoring of repeated patterns.
- **Dimensional:** star/snowflake-ready dimensions and facts optimized for analytics.
- **BI/Gold (where applicable):** consumption-ready marts/semantic outputs.

Design objective: each model should have a clear role, with logic pushed as close as reasonable to the appropriate layer.

## 3) Naming Conventions

Use consistent model prefixes and file names:

- Staging/ODS models: `stg_[source]__[entity]s.sql`
- Intermediate models: `int_[entity]s_[verb]s.sql`
- Dimensional models: `dim_*`, `fct_*` patterns aligned to grain and domain

General rules:
- lower_snake_case only
- names should communicate grain + business meaning
- avoid overloaded terms (especially naming DS reporting models as enterprise facts)

## 4) Fact/Dimension Modeling Standards

For EDW dimensional models:
- Facts should contain foreign keys that resolve to dimensions.
- Dimensions should support stable joins and business entity context.
- Include standard audit columns on materialized tables (not views):
  - `dw_created_at`
  - `dw_updated_at`

When handling unknown or unresolved dimensional mappings, use enterprise-safe handling patterns (e.g., known default strategy per team standards) so fact loads remain robust.

## 5) History and Snapshots

When source systems do not provide full history tables, use dbt snapshots for historical tracking.

Preferred practice:
- apply history capture close to source where practical
- snapshot at individual table level when possible (instead of overly broad mixed-history models)
- choose history strategy based on business need (e.g., overwrite vs historical versioning)

## 6) Materialization and Performance

Pick materialization based on behavior and SLA:
- tables for stable reusable datasets
- incremental models for large append/update workloads with clear merge keys/logic
- views for lightweight/non-persistent transformations where performance allows

Keep model SQL readable and composable; split large logic into intermediate building blocks rather than monolithic queries.

## 7) Testing, Documentation, and Review Expectations

Minimum PR-ready quality bar:

- Add/maintain schema YAML documentation for models and key columns.
- Add dbt tests appropriate to model type:
  - uniqueness / not_null on primary identifiers
  - relationship tests on fact-to-dimension keys
  - accepted values/domain tests where business rules are explicit
- Confirm local execution for changed nodes (`dbt run`, `dbt test` for selected scope).
- Keep lineage clean (`ref()`/`source()` usage aligned with ownership boundaries).

For major models, include or update:
- model overview/domain context
- ERD references
- lineage references
- implementation notes and QA/signoff evidence where required by team workflow

## 8) Practical Guardrails by Role

### For Data Scientists
- Build DS-specific reporting/modeling assets in DS-owned folders only.
- Reuse curated upstream inputs through `ref()` from stable models where possible.
- Do not place DS artifacts in EDW core layers or name DS outputs as enterprise `fct_`/`dim_` unless explicitly approved.
- Preserve clarity: explicit grain, clear naming, and documented assumptions.

### For Data Engineers
- Enforce layer boundaries and ownership in PR review.
- Keep ODS/intermediate/dimensional semantics strict and reusable.
- Standardize audit columns (`dw_created_at`, `dw_updated_at`) for materialized EDW tables.
- Validate dimensional integrity (grain, keys, joins, history handling).
- Reject duplicate or conflicting logic that should be centralized in shared models/macros.

## 9) Quick PR Checklist

Before requesting review:

- [ ] Model is in the correct folder/layer for its ownership and purpose.
- [ ] Naming follows prefix and grain conventions.
- [ ] SQL logic is layered (not unnecessarily monolithic).
- [ ] Materialization matches workload and usage.
- [ ] Audit columns included where required.
- [ ] Tests and docs updated for changed models.
- [ ] Lineage uses `ref()`/`source()` correctly and avoids ownership violations.
- [ ] DS vs DE boundaries are respected.

---

## 10) Team Operating Note

If a requirement appears to conflict with this file, the **Confluence EDW & dbt Modeling Guidelines page is the source of truth**. Update this summary when Confluence standards evolve.