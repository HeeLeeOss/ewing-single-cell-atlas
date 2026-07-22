# ewing-single-cell-atlas — TASKS

> **Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 · **Owner:** TBD (maintainer) ·
> **Lane:** donated · **Project risk:** medium (education tasks: high)
>
> Itemized, schema-mapped backlog for `ewing-single-cell-atlas`. Read alongside `PLAN.md`.
> **Every task carries `verifiedNeed: false`** until a research/advocacy partner is secured (PLAN §2).

---

## How these tasks map to Hee-Lee Oss

Each row below becomes a **Task JSON** validated against `packages/schema/src/schemas.ts`. Field mapping:

- **`id`** — stable slug `ewing-atlas-<area>-NNN` (areas: `part` partnership, `infra`, `verify`, `data`,
  `anno`, `integ`, `rel` release, `edu` education, `ops` sustainability).
- **`title`** — the table Title.
- **`project`** — `"ewing-single-cell-atlas"`.
- **`type`** — schema enum `code | research | writing | data | design-spec | maintenance` (see "Type" col).
- **`lane`** — `"donated"` for all v1 tasks (no funded escrow). Funded tasks would need `fundedBudgetUsd`.
- **`priority`** — `high | medium | low`.
- **`domain`** — array, e.g. `["oncology","bioinformatics","single-cell","open-data"]`.
- **`riskTier`** — `low | medium | high` (education = high; license/data = medium; docs = low).
- **`urgent`** — boolean (almost all `false`; no time-critical beneficiary deadline yet).
- **`deliverable`** — schema enum `pr | dataset | document | translation` (mapped per task).
- **`tokenEstimate`** — `small | medium | large` (= the "Size" column).
- **`status`** — `open | in-progress | review | delivered | done` (all start `open`).
- **`context` / `objective` / `acceptanceCriteria[]` / `output`** — filled per task.
- **`resources[]`** — source/standard links.
- **`requestor`** — `"TBD"` until partner secured.
- **`verifiedNeed`** — **`false`** (no partner yet).
- **`outputLicense`** — `MIT` (code), `CC-BY-4.0` (data/docs), or `link-only` where redistribution barred.

Reviewer roles in tables: **CR** compliance reviewer · **BR** bioinformatics reviewer · **DR** domain
(Ewing/single-cell) reviewer · **MR** maintainer · **ONC** oncologist · **ADV** patient-advocate · **ST**
steward.

---

## Milestone M0 — Foundation & cold-start

Goal: repo, standards, guardrails, and a **verified open-data source inventory** — no partner required.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-infra-001 | Scaffold repo: README, CONTRIBUTING, LICENSE(s), CI skeleton | code | small | low | pr | — | MR |
| ewing-atlas-infra-002 | Define `SourceRecord` + `Datasheet` + provenance JSONL schemas | design-spec | medium | low | document | infra-001 | BR, MR |
| ewing-atlas-infra-003 | Write refusal/guardrail + compliance doc (cancer guardrails, exclude-list) | writing | small | medium | document | infra-001 | CR, MR |
| ewing-atlas-verify-001 | Discover + triage ≥15 candidate Ewing scRNA-seq sources | research | medium | medium | document | infra-002 | CR |
| ewing-atlas-verify-002 | License + access-tier + de-id verification per candidate (hard gate) | research | large | medium | document | verify-001, infra-003 | CR |
| ewing-atlas-part-001 | Partner/beneficiary outreach (research lab and/or advocacy org) | writing | small | low | document | infra-001 | ST, MR |

**Acceptance criteria — key tasks**

- **ewing-atlas-verify-002** (license/access verification — the gate):
  - [ ] Each candidate has a `SourceRecord` with: license verbatim + tag, access tier, de-id evidence,
        redistribution-allowed flag, retrieval URL, checksum-on-fetch plan.
  - [ ] Every **controlled-access** (dbGaP/EGA/TARGET-controlled/biobank) or **identifiable** source is
        marked **excluded** with the reason recorded.
  - [ ] **COSMIC/OncoKB/non-commercial** resources flagged and excluded from derivatives.
  - [ ] Output splits into `verified-open` vs `excluded` lists; CR sign-off recorded.
  - [ ] No accession is marked "approved" without a passing recorded check.
- **ewing-atlas-infra-002** (schemas):
  - [ ] `SourceRecord`, `Datasheet`, `ProvenanceEvent`, `AnnotationAssertion` schemas defined + example
        instances validate.
  - [ ] Provenance schema requires per-assertion source citation field.
- **ewing-atlas-infra-003** (guardrail doc):
  - [ ] Documents open-access-only rule, exclude-list, "not medical advice," provenance mandate, and the
        "when in doubt, stop and surface" escalation path.

**Definition of Done (M0):** repo + docs live; four schemas merged + validating; ≥15 sources triaged
with recorded license/access decisions; verified-open vs excluded lists published; ≥1 partner-outreach
attempt logged; CI runs schema + license-gate checks.

---

## Milestone M1 — Standardize (pilot)

Goal: prove the pipeline end-to-end on 1–3 verified-open datasets.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-infra-004 | Reproducible env + pipeline skeleton (pinned, deterministic) | code | medium | low | pr | infra-002 | BR |
| ewing-atlas-data-001 | Acquire + checksum pilot verified-open dataset(s); record provenance | data | medium | medium | dataset | verify-002, infra-004 | CR, BR |
| ewing-atlas-data-002 | Standardize pilot dataset → CELLxGENE-schema `.h5ad` + QC report | data | large | medium | dataset | data-001 | BR |
| ewing-atlas-data-003 | Author datasheet (Gebru-style) for each pilot dataset | writing | medium | low | document | data-002 | BR, MR |

**Acceptance criteria — key tasks**

- **ewing-atlas-data-002** (standardize):
  - [ ] Output passes `cellxgene-schema validate` in CI.
  - [ ] Gene IDs normalized to HGNC/Ensembl; obs/var fields ontology-mapped (CL/UBERON/MONDO).
  - [ ] QC report (cells, genes, mito%, doublet handling) included; thresholds documented.
  - [ ] Provenance JSONL complete (source → transforms → output hash); pipeline re-runs reproducibly.
- **ewing-atlas-data-001** (acquire):
  - [ ] Only `verified-open` sources fetched; SHA-256 recorded; no controlled/identifiable data present.
  - [ ] Scan for identifier-like fields passes (or quarantine + exclude).

**Definition of Done (M1):** ≥1 (target 3) verified-open dataset standardized + schema-valid in CI; each
with complete datasheet + provenance; clean-machine reproducibility confirmed by an independent runner.

---

## Milestone M2 — Annotate

Goal: ontology-backed cell labels + cited program scores, every label sourced.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-anno-001 | Assemble citable, license-clean EWSR1-FLI1/ETS gene signatures | research | medium | medium | document | verify-002 | DR, CR |
| ewing-atlas-anno-002 | Cell-type annotation (Cell Ontology) for pilot datasets | data | large | medium | dataset | data-002, anno-001 | DR, BR |
| ewing-atlas-anno-003 | Per-assertion annotation-provenance + confidence; CI completeness check | code | medium | medium | pr | anno-002 | BR |

**Acceptance criteria — key tasks**

- **ewing-atlas-anno-002** (annotation):
  - [ ] Every cell/cluster label is a valid CL term; method + confidence recorded.
  - [ ] **100% of labels carry a source citation** (paper/marker-set); DR sign-off recorded.
  - [ ] Program scores reference only license-clean, citable signatures (from anno-001).
- **ewing-atlas-anno-001** (signatures):
  - [ ] Each signature traced to a citable source; redistribution of the gene list license-checked.
  - [ ] Non-commercial-sourced lists excluded from CC-BY output.

**Definition of Done (M2):** pilot datasets annotated; CI blocks any release with an unsourced label;
program scores documented; DR sign-off recorded.

---

## Milestone M3 — Integrate

Goal: cross-dataset reference atlas with an honest limitations statement.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-integ-001 | Integrate ≥3 datasets (deterministic batch correction) | data | large | medium | dataset | anno-002 | BR, DR |
| ewing-atlas-integ-002 | Integration report + explicit limitations statement | writing | medium | medium | document | integ-001 | BR, DR |

**Acceptance criteria — ewing-atlas-integ-001 / 002:**
- [ ] ≥3 standardized+annotated datasets integrated; method + params + tool versions recorded.
- [ ] Pre-/post-integration shown; batch-effect handling documented.
- [ ] **Limitations statement** explicitly warns against reading integration artifacts as biology.
- [ ] Reproducible by independent runner; BR + DR sign-off.

**Definition of Done (M3):** integrated atlas artifact + report merged; reproducible; dual sign-off.

---

## Milestone M4 — Release v1

Goal: public, browseable, DOI-archived atlas under correct open licenses.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-rel-001 | Release bundle: CELLxGENE deploy + download + Zenodo DOI + CHANGELOG | code | medium | medium | dataset | integ-002 | MR, CR, ST |
| ewing-atlas-rel-002 | License + attribution audit of full release bundle (0-violation gate) | research | medium | medium | document | rel-001 | CR |

**Acceptance criteria — ewing-atlas-rel-001 / 002:**
- [ ] Atlas browseable (CELLxGENE) + downloadable + archived with a Zenodo DOI.
- [ ] Per-source attribution + license correct; mixed/non-redistributable data link-only, not re-hosted.
- [ ] **0 license/access violations** (CR audit); release reproducible by an independent runner.
- [ ] README links the release as the beneficiary-facing home (Definition of Shipped met).

**Definition of Done (M4):** v1 atlas publicly available + DOI'd + linked; provenance/license/schema CI
all green; steward confirms it reaches the researcher beneficiary path.

---

## Milestone M5 — Education layer (CONDITIONAL — gated on partner + dual experts)

Goal: sourced, "not medical advice" explainer for families/advocates. **Does not start without an
advocacy partner AND credentialed oncologist + patient-advocate reviewers.**

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-edu-001 | Draft "what single-cell research is learning about Ewing" explainer | writing | medium | **high** | document | rel-001, part-001 | DR |
| ewing-atlas-edu-002 | Dual expert review + comprehension test ("not medical advice") | writing | medium | **high** | document | edu-001 | **ONC, ADV** |

**Acceptance criteria — ewing-atlas-edu-001 / 002:**
- [ ] Every statement sourced; plain language; prominently labeled **"not medical advice."**
- [ ] No prognosis, treatment, or patient-specific guidance.
- [ ] Comprehension-tested with an advocate panel; **0 instances read as medical advice.**
- [ ] **Both** oncologist and patient-advocate sign-off recorded before merge (veto = block).

**Definition of Done (M5):** explainer published only after dual expert sign-off + comprehension test;
otherwise milestone is **formally deferred** and recorded as such.

---

## Milestone M6 — Sustainability

Goal: refresh cadence, maintenance, and outcome tracking.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-atlas-ops-001 | Quarterly refresh runbook + new-dataset intake process | maintenance | small | low | document | rel-001 | MR |
| ewing-atlas-ops-002 | Outcome-metrics dashboard + "covered/missing" public ledger | code | medium | low | pr | rel-001 | MR, ST |

**Acceptance criteria — ops-001 / 002:**
- [ ] Runbook covers gated intake of new open datasets via the same pipeline.
- [ ] Dashboard tracks PLAN §4 metrics incl. **license-violation count (target 0)**.
- [ ] Public ledger shows what's curated vs. what's still missing.

**Definition of Done (M6):** runbook + dashboard + ledger live; maintainer + steward confirmed.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Note |
|---|---|---|---|---|---|
| ewing-atlas-data-010 | Scale standardization to all verified-open datasets | data | large | medium | After pilot proven |
| ewing-atlas-integ-010 | Evaluate scVI vs harmonypy integration (reproducibility tradeoff) | research | medium | medium | Open question §16 Q2 |
| ewing-atlas-anno-010 | Reconcile conflicting cross-study cell-type labels | research | medium | medium | Tie-break policy (§16 Q4) |
| ewing-atlas-edu-010 | Translate vetted explainer (links to ewing-info-translations) | writing | medium | high | Only post-M5; dual-expert per language |
| ewing-atlas-ops-010 | Spatial / multi-omic extension scoping | design-spec | medium | medium | v2 candidate |
| ewing-atlas-infra-010 | Funded-lane reanalysis option (metered, budget-capped) | code | medium | medium | Needs `fundedBudgetUsd`; only if escrow exists |

> **Fan-out note (`ewing-atlas-edu-010`):** target languages are intentionally **not pre-enumerated**.
> Translation is `type:"writing"` + `deliverable:"translation"` and expands into per-language tasks only
> on advocacy-partner/scope confirmation, with **dual-expert (oncologist + patient-advocate) sign-off per
> language**. The JSON is a single representative task until languages are secured (no fabricated languages).
> **Fan-out note (`ewing-atlas-infra-010`):** kept `lane:"donated"` — it *builds* the optional funded path;
> no escrow exists for v1, so any *future* funded reanalysis task must declare `fundedBudgetUsd` itself.

---

## Example task JSON

Schema-valid (`packages/schema/src/schemas.ts`) Task JSON for the first M0 task. `verifiedNeed: false`
(no partner secured), `lane: donated` (so `fundedBudgetUsd` not required).

```json
{
  "id": "ewing-atlas-infra-001",
  "title": "Scaffold ewing-single-cell-atlas repo: README, CONTRIBUTING, licenses, CI skeleton",
  "project": "ewing-single-cell-atlas",
  "type": "code",
  "lane": "donated",
  "priority": "high",
  "domain": ["oncology", "bioinformatics", "single-cell", "open-data"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "pr",
  "tokenEstimate": "small",
  "status": "open",
  "context": "New Hee-Lee Oss cancer-research good-deed project to curate and annotate ONLY open-access / aggregate / de-identified public Ewing sarcoma scRNA-seq data. Controlled-access (dbGaP, EGA, individual-level biobanks) and any identifiable patient data are out of scope. No partner/beneficiary is secured yet, so verifiedNeed is false. This first task creates the repository foundation that every later gate (license verification, provenance, schema validation) hangs off of.",
  "objective": "Scaffold the repository with README, CONTRIBUTING, dual licensing (MIT for code, CC-BY-4.0 for data/docs), a CODEOWNERS/reviewer-roles stub, and a CI skeleton that will later run cellxgene-schema validation plus license and provenance gates.",
  "acceptanceCriteria": [
    "README states scope, the binding cancer guardrails (open-access only; no controlled-access or identifiable data), and 'not medical advice'.",
    "LICENSE files present: MIT for code, CC-BY-4.0 for data/docs; licensing policy documented.",
    "CONTRIBUTING.md documents the Stage-1 license/access-verification gate and the 'when in doubt, stop and surface' escalation path.",
    "CI skeleton runs on PRs and has placeholder jobs for schema-validate, license-gate, and provenance-completeness checks.",
    "No secrets, tokens, or large data binaries committed; .gitignore excludes raw data.",
    "Commit signed off per DCO."
  ],
  "resources": [
    "C:/code/hee-lee-oss/CLAUDE.md",
    "C:/code/hee-lee-oss/docs/good-deed-definition.md",
    "C:/code/hee-lee-oss/planning/ROADMAP.md (Track 8 cancer guardrails)",
    "https://github.com/chanzuckerberg/single-cell-curation (CELLxGENE schema)"
  ],
  "output": "A pull request adding the repository scaffold (README, CONTRIBUTING, LICENSE files, reviewer-roles stub, CI skeleton) for ewing-single-cell-atlas.",
  "requestor": "TBD",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

---

## Generated task index

Every backlog row above is now backed by a schema-valid `tasks/<id>.json` (validated against
`packages/schema/src/schemas.ts`; all 27 pass). All tasks: `lane: donated`, `status: open`,
`verifiedNeed: false`, `requestor: TBD`.

- **M0:** `ewing-atlas-infra-001` (seed), `ewing-atlas-infra-002`, `ewing-atlas-infra-003`,
  `ewing-atlas-verify-001`, `ewing-atlas-verify-002`, `ewing-atlas-part-001`
- **M1:** `ewing-atlas-infra-004`, `ewing-atlas-data-001`, `ewing-atlas-data-002`, `ewing-atlas-data-003`
- **M2:** `ewing-atlas-anno-001`, `ewing-atlas-anno-002`, `ewing-atlas-anno-003`
- **M3:** `ewing-atlas-integ-001`, `ewing-atlas-integ-002`
- **M4:** `ewing-atlas-rel-001`, `ewing-atlas-rel-002`
- **M5 (conditional, high-risk):** `ewing-atlas-edu-001`, `ewing-atlas-edu-002`
- **M6:** `ewing-atlas-ops-001`, `ewing-atlas-ops-002`
- **Backlog:** `ewing-atlas-data-010`, `ewing-atlas-integ-010`, `ewing-atlas-anno-010`,
  `ewing-atlas-edu-010`, `ewing-atlas-ops-010`, `ewing-atlas-infra-010`

`outputLicense`: `MIT` for `pr` deliverables (code/CI), `CC-BY-4.0` for documents/datasets, and
source-compatible for the translation task. M5/edu and `edu-010` carry `riskTier: high` with the
oncologist + patient-advocate dual sign-off requirement preserved verbatim in their `context`/criteria.

## Summary counts

- **Milestones:** 7 (M0–M6; M5 conditional).
- **Scheduled tasks:** 21 across M0–M6 · **Backlog:** 6 · **Total:** 27.
- **By risk:** high 3 (all education); medium ~15; low ~9.
- **All tasks:** `lane: donated`, `verifiedNeed: false` until a partner is secured.
- **Hardest gates:** Stage-1 license/access verification (CR veto) and dual oncologist+advocate sign-off
  on any family-facing content.
