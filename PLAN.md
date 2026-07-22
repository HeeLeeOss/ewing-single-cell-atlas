# ewing-single-cell-atlas — PLAN

> **Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 · **Owner:** TBD (maintainer) ·
> **Lane:** donated · **Risk tier (project):** medium — *patient-facing education artifacts are `high` and
> gated behind oncologist + patient-advocate sign-off (see §8).*
>
> A curated, openly-licensed, reproducibly-annotated single-cell RNA-seq **atlas of Ewing sarcoma**,
> built only from open-access / aggregate / de-identified public data, with provenance on every
> assertion. For the families living with Ewing sarcoma, and the researchers trying to end it.

---

> **Why we are careful here.** Ewing sarcoma is a rare bone and soft-tissue cancer that strikes
> mostly children, teenagers, and young adults. Behind every dataset row is a child and a family.
> This plan treats *data dignity, licensing honesty, and "not medical advice"* as non-negotiable
> identity constraints, not checkboxes. If a step would risk re-identifying a patient, overstate a
> finding, or imply clinical guidance, we do not ship it. **When in doubt, we stop and surface the
> concern.** That posture is the whole point of doing this inside Hee-Lee Oss rather than as an unowned
> side project.

---

## 1. Executive summary

Ewing sarcoma research is fragmented: dozens of valuable single-cell RNA-seq (scRNA-seq) datasets sit
in public archives (GEO, ArrayExpress, the Broad Single Cell Portal, Human Cell Atlas, Zenodo) under
inconsistent metadata, incompatible formats, mixed licenses, and no shared cell-type vocabulary. A
researcher who wants to ask "which cell states express *EWSR1-FLI1* target programs across all public
Ewing tumors?" must first re-download, re-process, and re-annotate everything by hand — a weeks-long
tax that is paid over and over by every lab, and that is largely inaccessible to advocacy groups,
students, and computational volunteers.

**ewing-single-cell-atlas** removes that tax. It is a **content/data curation project** (donated lane)
that:

1. **Inventories** every *open-access* Ewing scRNA-seq dataset and verifies each one's license and
   access tier before a single byte is reused.
2. **Standardizes** the open ones into a single, validated schema (AnnData `.h5ad` conforming to the
   CELLxGENE / HCA dataset schema) with a machine-readable **datasheet** and full **provenance** per
   dataset.
3. **Annotates** cells against a shared, ontology-backed vocabulary (Cell Ontology) and scores
   published, citable *EWSR1-FLI1* / ETS transcriptional programs — every annotation tied to a source.
4. **Integrates** the standardized datasets into a browseable, openly-licensed reference atlas, with an
   honest "known limitations" boundary on what integration can and cannot claim.
5. **Releases** the atlas, the pipeline, and (only after expert sign-off) a *strictly educational*,
   sourced, "not medical advice" explainer for families and patient advocates.

The hard gates that define this project: **open-access data only** (controlled-access and any
identifiable patient data are out of scope), **per-source license verification** (TCGA/GDC open-tier and
GEO are open; COSMIC/OncoKB and similar are non-commercial/custom and are flagged, not silently reused),
**provenance on every assertion**, and **oncologist + patient-advocate review** on anything a family
might read. Success is measured in researcher and family outcomes — reuse, time saved, citations,
reproducibility — not in tasks merged.

No research partner or beneficiary org is secured yet, so **`verifiedNeed = false`** across the backlog
until one is confirmed (see §2, §11).

---

## 2. Problem & beneficiaries

### The problem
- **Fragmentation.** Public Ewing scRNA-seq lives in many archives, in raw FASTQs, loom files, 10x
  matrices, author-specific `.h5ad`/`.rds`, and supplementary spreadsheets — with no common gene-ID
  convention, no shared cell-type labels, and frequently missing or ambiguous license statements.
- **Re-work tax.** Every lab re-processes the same data. This wastes scarce rare-cancer research
  capacity and makes cross-study comparison (the entire promise of single-cell data) slow and error-
  prone.
- **Access asymmetry.** Students, citizen-scientists, computational volunteers, and patient-advocacy
  scientists cannot easily engage with the data at all, because the on-ramp is too steep.
- **Provenance rot.** Findings get repeated without their source; license terms get lost; de-
  identification assumptions go unchecked.

### Who is helped (beneficiaries)
1. **Ewing sarcoma researchers** (academic, especially small / rare-cancer labs): a ready-to-analyze,
   harmonized, citable reference atlas + reproducible pipeline.
2. **Computational biology students & volunteers:** a clean, well-documented teaching dataset and an
   approachable contribution surface.
3. **Patient-advocacy organizations & families** (indirectly, and only via the `high`-risk education
   layer): a sourced, plain-language, "not medical advice" explainer of what single-cell research is
   learning about Ewing sarcoma — to help them follow and trust the science.
4. **The broader rare-cancer methods community:** a worked, auditable example of compliant open-data
   curation that other rare cancers can copy.

### Verified need — **TO BE SECURED**
There is a strong *inferred* need (the re-work tax is real and widely reported), but **no partner
research lab, advocacy org, or named beneficiary has confirmed this specific deliverable yet.** Per
Hee-Lee Oss honesty rules, every task carries **`verifiedNeed = false`** until we secure at least one of:
- a **research-partner** lab that commits to using/reviewing the atlas, **or**
- a **patient-advocacy partner** (e.g. a Ewing/sarcoma foundation) for the education layer, **or**
- documented, dated demand signals (issue requests, citations of intent, a letter of support).

**Partner org:** none yet. Outreach is task `ewing-atlas-part-001` (M0). Do not promise beneficiaries
in any public copy until secured.

---

## 3. Goals and non-goals

### Goals
- A **verified open-data source inventory** of Ewing scRNA-seq, each entry license- and access-tier-
  checked, with the controlled-access ones explicitly excluded and *why* recorded.
- A **standardized, validated dataset collection** (CELLxGENE/HCA-schema `.h5ad`) with per-dataset
  **datasheets** and machine-readable provenance.
- **Ontology-backed cell annotations** + **citable program scores**, every label traceable to a source.
- A **reproducible, open-source pipeline** (deterministic, pinned, re-runnable from raw open inputs).
- An **integrated reference atlas** with an explicit, honest statement of integration limitations.
- A **public release** (CELLxGENE-browsable + downloadable + archived with a DOI) under an open license.
- A **strictly educational, expert-signed-off explainer** for families/advocates — *if* a partner and
  reviewers are secured; otherwise this scope is deferred, not faked.

### Non-goals (explicit)
- **Not** a clinical or diagnostic tool. **No medical advice, no prognosis, no treatment guidance.**
- **Not** a re-identification or linkage effort. We never attempt to link cells/donors to identities or
  to external records.
- **Not** a user of controlled-access data (dbGaP, EGA, individual-level biobanks) or *any* identifiable
  patient data — that requires authorized access + IRB and is **out of scope for donated AI tasks**.
- **Not** primary wet-lab or sequencing — we curate existing *public* data only; we generate no new
  patient data.
- **Not** a novel-biology discovery claim engine. We harmonize and annotate; we report what published
  sources say, with provenance. We do not assert new clinical findings.
- **Not** a redistribution of any dataset whose license forbids redistribution/derivatives — those are
  *linked and described*, never re-hosted.
- **Not** a for-profit product, and not built to primarily benefit any company.

---

## 4. Success metrics (outcomes)

Beneficiary-centric, outcome-based. Baselines are ~0 (greenfield); targets are first-12-month.

| # | Outcome | Metric | Baseline | Target |
|---|---|---|---|---|
| 1 | Researchers reuse the atlas | Independent downloads / CELLxGENE sessions; reuse logged in issues | 0 | ≥ 250 downloads; ≥ 5 documented external reuses |
| 2 | Time saved | Self-reported hours saved vs. DIY (partner survey) | n/a | ≥ 3 partner labs report ≥ 1 week saved each |
| 3 | Citability | Atlas/pipeline cited or acknowledged in a preprint/paper | 0 | ≥ 1 citation or acknowledgement |
| 4 | Coverage | # open datasets verified-included / total open candidates | 0 | ≥ 80% of verified-open candidates curated |
| 5 | Reproducibility | Pipeline re-run reproduces release artifacts bit-for-bit (or within tolerance) on a clean machine | n/a | 100% of releases reproducible by an independent runner |
| 6 | Provenance completeness | % of cell-annotation assertions with a cited source | n/a | 100% (hard gate; releases blocked otherwise) |
| 7 | Compliance integrity | # of license/access violations in shipped artifacts | n/a | **0** (any >0 is a sev-1 incident) |
| 8 | Family/advocate trust (only if education layer ships) | Advocate-panel comprehension + "felt trustworthy / not advice" rating | n/a | ≥ 80% positive; **0** instances read as medical advice |

> Vanity metrics we will **not** treat as success: PRs merged, stars, commit counts, dataset *count*
> alone (a count without verified licenses and provenance is a liability, not a win).

---

## 5. Scope

### In scope
- Discovery + **license/access verification** of open Ewing scRNA-seq datasets.
- Format standardization to validated `.h5ad` (CELLxGENE/HCA schema); gene-ID normalization (HGNC/
  Ensembl); QC harmonization.
- Per-dataset **datasheets** (Gebru-style) + machine-readable provenance records.
- Ontology-backed **cell-type annotation** (Cell Ontology) + **published-program scoring**
  (e.g. *EWSR1-FLI1*-up/-down signatures from cited papers), each label sourced.
- Cross-dataset **integration** into a reference atlas with documented batch-correction method and an
  explicit limitations statement.
- Reproducible **pipeline** + CI validation + public **release** (browser + download + DOI archive).
- *Conditional:* a `high`-risk, expert-reviewed, "not medical advice" **education explainer**.

### Out of scope (will NOT do)
- Any **controlled-access** data (dbGaP, EGA, individual-level biobanks) or **identifiable patient
  data** — full stop.
- **Re-identification / linkage** of donors; demographic inference about individuals.
- **Redistribution** of non-redistributable datasets (link + describe only).
- **New clinical claims**, prognosis, biomarkers-for-decisions, or treatment recommendations.
- **Patient-specific** anything; any 1:1 advice.
- Use of **non-commercial / custom-license** resources (e.g. COSMIC, OncoKB) inside openly-licensed
  outputs — these are flagged and excluded from derivatives unless terms demonstrably permit it.
- Wet-lab work, new sequencing, or generating new patient data.
- Spatial / multi-omic / proteomic integration (possible future track; not v1).

---

## 6. Solution approach & architecture

This is a **data/content curation pipeline**, not an application. Humans run their own agents (donated
lane); the repo prepares workspaces and opens PRs. The "architecture" is the pipeline, schemas,
provenance model, and review gates.

### Pipeline stages (each stage = reviewable PR artifacts)
```
[0 DISCOVER]   search GEO / ArrayExpress / SCP / HCA / Zenodo for Ewing scRNA-seq
      │            → candidate list (raw)
[1 VERIFY]     per-source license + access-tier + de-identification check  ← HARD GATE
      │            → verified-open inventory  +  excluded list (with reasons)
[2 ACQUIRE]    fetch ONLY open-access matrices/metadata; checksum; record provenance
      │            → raw/ (gitignored or DVC/pointer) + provenance.jsonl
[3 STANDARDIZE]→ AnnData .h5ad; gene IDs→HGNC/Ensembl; obs/var to CELLxGENE schema; QC
      │            → standardized/<dataset>.h5ad  + datasheet.md  + qc-report
[4 ANNOTATE]   cell-type labels (Cell Ontology) + cited program scores; provenance per label
      │            → annotated/<dataset>.h5ad  + annotation-provenance.jsonl
[5 INTEGRATE]  batch-correct + integrate; document method + limitations
      │            → atlas/ewing-atlas.h5ad  + integration-report.md
[6 RELEASE]    CELLxGENE deploy + download bundle + Zenodo DOI + CHANGELOG + LICENSE
      │            → public release
[7 EDUCATE*]   (conditional, high-risk) sourced "not medical advice" explainer  ← EXPERT GATE
```
`*` Stage 7 only runs after a patient-advocacy partner **and** oncologist + advocate reviewers are
secured. Without them, stage 7 is deferred (not produced).

### Tech stack
- **Language/runtime:** Python 3.11 (the single-cell ecosystem standard) for the pipeline; the **Hee-Lee Oss
  CLI/repo conventions** govern task packaging. *(Note: this is a content/data project, so the
  TS/ESM/pnpm rule in CLAUDE.md applies to Hee-Lee Oss platform code, not to scientific pipeline code; this
  exception is recorded as a locked decision below.)*
- **Core libs:** `scanpy`, `anndata`, `cellxgene-schema` (validator), `pandas`, `numpy`; integration via
  `scanpy`/`harmonypy` (deterministic, well-understood) — `scVI` considered but deferred for
  reproducibility/determinism reasons (see Open Questions).
- **Ontologies/IDs:** Cell Ontology (CL), HGNC, Ensembl, UBERON (tissue), MONDO/NCIT (disease) — all
  versioned and pinned.
- **Reproducibility:** pinned environment (`conda-lock`/`uv` lockfile), deterministic seeds, content
  hashing; large artifacts via pointer (DVC or git-LFS or Zenodo-by-reference) — **no large binaries in
  git**.
- **Provenance:** append-only JSONL records (PROV-style: source URL, accession, license, retrieval
  date, checksum, transform, tool versions) accompanying every artifact.
- **CI:** GitHub Actions runs `cellxgene-schema validate`, provenance-completeness check, license-gate
  check, and pipeline smoke test on every PR.

### Locked decisions
- **L1 — Open-access only, verified per source.** Stage 1 is a hard gate; nothing proceeds without a
  recorded, passing license + access-tier check.
- **L2 — Standard schema = CELLxGENE/HCA `.h5ad`.** One target schema, validated in CI. No bespoke
  formats in releases.
- **L3 — Provenance is mandatory, machine-readable, per assertion.** A release with any unsourced
  annotation does not ship.
- **L4 — Education layer is `high`-risk and gated.** No family-facing content without oncologist +
  patient-advocate sign-off and "not medical advice" framing. If reviewers aren't secured, it is cut.
- **L5 — Scientific pipeline is Python; Hee-Lee Oss platform/tooling stays TS/ESM** (recorded CLAUDE.md
  exception, see §16/§17).
- **L6 — No re-hosting of non-redistributable data.** Link + describe only.

### Data model (key objects)
- **`SourceRecord`** — accession, archive, title, study DOI, license (verbatim + SPDX-ish tag), access
  tier (open / controlled — controlled ⇒ excluded), de-id assertion + evidence, redistribution-allowed
  (bool), retrieval URL.
- **`Datasheet`** — Gebru-style: motivation, composition, collection, preprocessing, uses, distribution,
  maintenance; + counts (cells/genes/donors), assays, tissue, disease state.
- **`AnnotationAssertion`** — cell/cluster → label (CL term), method, confidence, **source citation**
  (required), reviewer.
- **`ProvenanceEvent`** — append-only: input hashes → tool+version → params → output hash → timestamp.

---

## 7. Data, licensing & compliance  *(THIS IS THE CRITICAL SECTION — read first)*

### 7.0 Cancer guardrails (binding — these lead, and override everything below if in tension)
1. **Open-access / aggregate / de-identified data ONLY.** **Controlled-access sources (dbGaP, EGA,
   individual-level biobanks) and ANY identifiable patient data are OUT OF SCOPE** — they require
   authorized access + IRB and are **not** appropriate for donated AI tasks. They are never downloaded,
   never staged, never referenced as inputs.
2. **Verify each source's license before any reuse.** TCGA/GDC **open-tier** and **GEO** are open;
   **COSMIC, OncoKB**, and similarly **non-commercial / custom-licensed** resources are **flagged and
   excluded from openly-licensed derivatives** unless their terms demonstrably permit the specific reuse.
   Verification is recorded, dated, and reviewable.
3. **No medical advice.** Any patient-facing content is **education only**, **sourced**, explicitly
   labeled **"not medical advice,"** and **gated behind oncologist + patient-advocate review
   (`riskTier: high`).**
4. **Provenance on every assertion.** Every annotation, score, and claim carries a citable source. No
   provenance ⇒ does not ship.
5. **When in doubt, stop and surface.** Ambiguous license, unclear de-identification, possible
   re-identification risk → halt and flag for human/expert decision; do not proceed.

### 7.1 Candidate sources (open-access archives) — *all entries pending Stage-1 verification*
| Source | What | Default access posture | License note |
|---|---|---|---|
| **NCBI GEO / GEO2R** | scRNA-seq series (e.g. Ewing tumor & cell-line studies) | Open (verify per series) | Public-domain-ish (US Gov) data; *per-series supplementary files may carry author terms* — verify |
| **EBI ArrayExpress / BioStudies** | scRNA-seq series | Open (verify) | Per-study; verify |
| **Broad Single Cell Portal** | processed scRNA-seq | Mixed | Per-study license; verify, some require agreement |
| **Human Cell Atlas (HCA) Data Portal** | standardized scRNA-seq | Open | Typically CC-BY / CC0; verify |
| **Zenodo / figshare** | author-deposited processed data | Mixed | Per-deposit license (often CC-BY); verify |
| **GDC / TCGA (open tier)** | mostly bulk; pediatric sarcoma scRNA-seq rare | Open tier only | Open-tier reusable; **never** controlled tier |
| **cellxgene Census / CZ CELLxGENE Discover** | standardized atlases | Open | CC-BY; verify per collection |

> **Explicitly excluded by default:** dbGaP, EGA, **TARGET/GDC controlled tier**, any individual-level
> biobank, and any dataset whose de-identification status cannot be confirmed. **COSMIC, OncoKB** and
> other non-commercial/custom resources are excluded from derivatives unless terms demonstrably permit.
>
> Specific GEO/SCP accession IDs are deliberately **not pre-listed as "approved"** here — listing an
> accession is the *output* of Stage 1 verification, not an input. Treat any accession in working notes
> as "candidate, unverified" until its `SourceRecord` passes the gate.

### 7.2 Provenance model
Every artifact ships with append-only JSONL provenance: source accession + URL + archive, study DOI,
**license verbatim + tag**, access tier, retrieval date, **SHA-256 checksum**, every transform with tool
+ version + params, and (for annotations) the **citation** backing each label. Provenance completeness is
CI-enforced; a release with gaps is blocked.

### 7.3 Privacy / PII stance
- Inputs are **already de-identified, aggregate single-cell expression** — but we **independently
  confirm** the de-identification claim per source and record the evidence.
- We perform **no linkage, no re-identification, no individual demographic inference.**
- If any file contains apparent direct/indirect identifiers (donor names, dates beyond year, free-text
  notes, geographic micro-data), it is **quarantined and the source excluded** pending human review.
- No secrets, tokens, or keys in logs/receipts/commits (Hee-Lee Oss rule).

### 7.4 Attribution & output licensing
- **Code/pipeline:** MIT.
- **Curated data & annotations (our derivatives):** **CC-BY-4.0** *where every upstream license permits
  derivative + redistribution under CC-BY*; otherwise **link-and-describe only** (no re-host) and clearly
  mark the entry.
- **Datasheets / docs / education explainer:** **CC-BY-4.0**.
- **Upstream attribution** (study, authors, accession, DOI, license) is reproduced for every dataset.
- **Share-alike / ODbL / non-commercial** upstream terms are honored exactly; mixed-license datasets are
  segregated so an incompatible term never contaminates a CC-BY release bundle.

---

## 8. Quality, review & risk gates

### Risk tier
- **Project default: `medium`** (domain accuracy: bioinformatics correctness, license compliance,
  ontology fidelity → reviewer with relevant skill required).
- **Education / patient-facing artifacts: `high`** → **credentialed oncologist + patient-advocate
  sign-off required before merge**, plus "not medical advice" framing and full sourcing.
- **License/access-verification tasks** are treated as `medium` but with a **compliance-reviewer veto**
  (a failed verification blocks all downstream work regardless of other approvals).

### Required reviews before a deed is "done"
| Artifact type | Required reviewers | Gate |
|---|---|---|
| Source verification (Stage 1) | Compliance reviewer | License + access-tier pass recorded; controlled/identifiable excluded |
| Standardized dataset (Stage 3) | Bioinformatics reviewer | `cellxgene-schema validate` passes; datasheet complete; provenance complete |
| Annotation (Stage 4) | Domain (Ewing/single-cell) reviewer | Every label sourced; ontology terms valid; confidence recorded |
| Integration/atlas (Stage 5) | Bioinformatics + domain reviewer | Method + limitations documented; reproducible |
| Education explainer (Stage 7) | **Oncologist + patient-advocate (both)** | "Not medical advice"; sourced; comprehension-tested; **veto = block** |
| Any release | Maintainer + steward | Outcome handed to a real beneficiary path; CI green |

### Definition of Shipped (delivered, not merged)
A deed is **shipped** when: acceptance criteria met **+** CI green (schema + provenance + license gates)
**+** required human/expert sign-off recorded **+** the artifact is actually published to its public
home (CELLxGENE collection / download bundle / Zenodo DOI) **and** linked from the project README — i.e.
genuinely usable by a beneficiary, not just merged.

---

## 9. Roadmap & milestones

Each milestone has a goal + measurable **exit criteria**. M0 is a thin, no-partner cold-start that
produces value (the verified inventory + standards) before any heavy lifting.

| Milestone | Goal | Exit criteria (measurable) |
|---|---|---|
| **M0 — Foundation & cold-start** | Stand up repo, standards, guardrails, and a **verified source inventory** — no partner required | Repo + CONTRIBUTING + license/guardrail docs live; datasheet + provenance + `SourceRecord` schemas merged; **≥ 15 candidate sources triaged, each with a recorded license + access-tier decision**; ≥ 1 partner-outreach attempt logged; refusal/guardrail doc merged |
| **M1 — Standardize (pilot, 1–3 datasets)** | Prove the pipeline end-to-end on a few verified-open datasets | ≥ 1 (target 3) verified-open dataset standardized to schema; `cellxgene-schema validate` green in CI; datasheet + complete provenance per dataset; pipeline re-runs reproducibly on a clean machine |
| **M2 — Annotate** | Ontology-backed cell labels + cited program scores | Pilot datasets annotated; **100% of labels sourced**; CL/HGNC terms validated; domain reviewer sign-off; annotation-provenance in CI |
| **M3 — Integrate** | Cross-dataset reference atlas with honest limitations | ≥ 3 datasets integrated; integration method + **limitations statement** documented; reproducible; bioinformatics + domain sign-off |
| **M4 — Release v1** | Public, browseable, DOI-archived atlas under open license | CELLxGENE-browsable + download bundle + Zenodo DOI; LICENSE/attribution correct per source; README + CHANGELOG; release reproducible by an independent runner; **0 license/access violations** |
| **M5 — Education layer (CONDITIONAL)** | Sourced, "not medical advice" family/advocate explainer | **Only if** advocacy partner + oncologist + advocate reviewers secured; explainer drafted, fully sourced, comprehension-tested, **dual expert sign-off**; else formally deferred |
| **M6 — Sustainability** | Refresh cadence + maintenance + outcome tracking | Quarterly refresh runbook; new-dataset intake process; outcome-metrics dashboard; maintainer + steward confirmed |

**Dependencies:** M1 depends on M0's verified inventory; M2→M1; M3→M2 (≥3 standardized+annotated);
M4→M3; **M5 depends on securing a partner + dual experts (may never start)**; M6→M4.

---

## 10. Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`** — milestone tables (`ID | Title | Type |
Size | Risk | Deliverable | Depends on | Reviewer`), acceptance criteria for the key tasks per
milestone, each milestone's Definition of Done, a backlog of sized-but-unscheduled items, and a complete
schema-valid example Task JSON. All tasks carry `verifiedNeed = false` until a partner is secured (§2).

---

## 11. Governance, roles & stakeholders

| Role | Responsibility | Status |
|---|---|---|
| **Maintainer** | Owns repo, roadmap, releases, final merge | **TBD** |
| **Compliance reviewer** | License + access-tier verification veto (Stage 1) | **TBD** (rotation of ≥2) |
| **Bioinformatics reviewer** | Schema, pipeline, integration correctness | **TBD** |
| **Domain reviewer (Ewing / single-cell)** | Annotation + biological-claim accuracy | **TBD** |
| **Oncologist reviewer** | `high`-risk education sign-off (credentialed) | **TBD — required before M5** |
| **Patient-advocate reviewer** | `high`-risk education sign-off; lived-experience lens | **TBD — required before M5** |
| **Steward (last-mile owner)** | Ensures releases reach beneficiaries; tracks outcomes | **TBD** |
| **Partner / requestor** | Research lab and/or advocacy org confirming the need | **NONE — TO BE SECURED** |

Reviewers rotate; no single person approves their own work. Edge cases / scope disputes go to the Hee-Lee Oss
board + community per the good-deed definition's published COI + veto checklist.

---

## 12. Dependencies & integrations

- **Data archives:** GEO, ArrayExpress/BioStudies, Broad SCP, HCA Data Portal, CZ CELLxGENE Discover,
  Zenodo/figshare, GDC open tier.
- **Ontologies/refs:** Cell Ontology, HGNC, Ensembl, UBERON, MONDO/NCIT (pinned versions).
- **Tooling:** `scanpy`, `anndata`, `cellxgene-schema`, `harmonypy`; `conda-lock`/`uv`; DVC or git-LFS;
  GitHub Actions.
- **Publishing:** CZ CELLxGENE (hosting/browse), Zenodo (DOI archive), GitHub (code + docs).
- **Hee-Lee Oss platform:** task registry/schema (`packages/schema`), CLI workspace prep + PR flow, governance
  proposal + COI/veto process, registry entry.
- **Upstream studies:** the original authors/papers behind each dataset (attribution + provenance).

---

## 13. Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Controlled-access or identifiable data slips into inputs | Low | **Critical** | Stage-1 hard gate; CI license/access check; quarantine-on-suspicion; compliance veto; default-exclude | Compliance reviewer |
| License misread → non-redistributable data re-hosted | Medium | **High** | Verbatim license capture + tag; link-and-describe fallback; segregate mixed licenses; release-bundle license check | Compliance reviewer |
| Non-commercial resource (COSMIC/OncoKB) contaminates CC-BY output | Medium | High | Explicit exclude-list; derivative-license check in CI; flag-don't-use default | Maintainer |
| Annotation/biology error misleads researchers | Medium | High | Per-label provenance; domain reviewer; confidence scores; "research, not clinical" framing | Domain reviewer |
| Education content read as medical advice | Medium (if M5 runs) | **Critical** | `high` tier; dual expert sign-off; "not medical advice" framing; comprehension testing; defer if no reviewers | Oncologist + advocate |
| Integration artifacts (batch effects) misinterpreted as biology | Medium | Medium | Documented method + explicit limitations statement; show pre-/post-integration | Bioinformatics reviewer |
| No partner secured → unclear real need | **High (now)** | Medium | `verifiedNeed=false` everywhere; M0 outreach; deliver researcher value (inventory/standards) regardless | Maintainer/steward |
| Reproducibility drift (deps, data moves) | Medium | Medium | Pinned envs; checksums; DOI snapshots; CI smoke test; independent re-run gate | Bioinformatics reviewer |
| Upstream data withdrawn/changed | Medium | Medium | Snapshot checksums + Zenodo archive of *derivatives only where permitted*; record source state | Maintainer |
| Volunteer burnout / bus factor | Medium | Medium | ≥2 reviewers per role; runbooks; small reviewable PRs | Maintainer |
| Overclaiming in public copy | Medium | High | Honesty rule; no beneficiary promises pre-partner; review of all external copy | Steward |

---

## 14. Security & privacy

- **Threat surface:** mostly data-handling, not a running service. Primary threats are (a) ingesting
  prohibited data, (b) leaking license/provenance errors into releases, (c) supply-chain (deps), (d)
  malicious PRs.
- **Prohibited-data controls:** default-deny; Stage-1 gate; CI scans new inputs for identifier-like
  fields and unverified accessions; quarantine + human review on any hit.
- **Secrets:** none should be required (public data). No tokens/keys in logs, receipts, or commits;
  secret-scanning in CI; archive credentials (if ever needed for open downloads) stay in the runner's
  environment, never committed.
- **Supply chain:** pinned, lock-filed deps; dependency review; reproducible builds; signed releases.
- **PR safety:** CI runs untrusted PRs without secrets; maintainer review before merge; provenance/
  license gates cannot be bypassed by a PR.
- **Abuse/misuse prevention:** refuse + flag any attempt to use the atlas for re-identification, clinical
  decision-making, or to attach identity to cells; "research / not advice" framing throughout; refusal
  guardrails (CLAUDE.md) bind every contributing agent.

---

## 15. Sustainability & maintenance

- **Maintainer + steward** own the repo post-v1; ≥2 reviewers per role to avoid bus-factor.
- **Refresh cadence:** quarterly intake of newly-published open Ewing scRNA-seq via the same gated
  pipeline; versioned, DOI'd releases; CHANGELOG.
- **Outcome tracking:** lightweight dashboard for the §4 metrics (downloads, reuse reports, citations,
  reproducibility pass rate, **license-violation count = 0**); a public "what's covered / what's
  missing" ledger.
- **Decommission plan:** if unmaintained, releases stay archived on Zenodo (DOIs persist) and the README
  is marked "unmaintained — last verified <date>"; no silent rot.
- **Cost:** donated lane → near-zero marginal cost (volunteer compute + free hosting tiers). No funded
  escrow needed for v1.

---

## 16. Open questions

1. **Partner/beneficiary:** who confirms the need — a research lab, a sarcoma foundation, or both? (Blocks `verifiedNeed=true` and M5.)
2. **Integration method:** deterministic `harmonypy`/Scanpy (reproducible) vs. `scVI` (often better, but stochastic) — how do we weigh integration quality vs. bit-reproducibility? Default: deterministic for v1.
3. **Redistribution boundary:** for CC-BY-but-ambiguous datasets, do we re-host derivatives or link-only? Default conservative (link) until license is unambiguous.
4. **Cell-type label authority:** where published labels conflict across studies, what is our tie-break + how do we record disagreement?
5. **Education layer go/no-go:** do we have credentialed oncologist + advocate reviewers? If not by M4, M5 is formally deferred.
6. **Pipeline language exception:** confirm with Hee-Lee Oss maintainers that a Python scientific pipeline is acceptable under the TS/ESM convention (recorded as L5; this is the norm for single-cell work).
7. **Hosting longevity:** CELLxGENE hosting terms + Zenodo quota for repeated large releases.
8. **Scope of "program scores":** which published *EWSR1-FLI1*/ETS signatures are citable + license-clean to redistribute as gene lists?

---

## 17. References

- Hee-Lee Oss: `CLAUDE.md` (work rules, lanes, refusal guardrails); `docs/good-deed-definition.md` (5 criteria
  + risk tiers); `packages/schema/src/schemas.ts` (Task schema); `planning/ROADMAP.md` (Track 8 cancer
  guardrails + portfolio).
- Exemplar plan depth: `C:\code\Ofelia\plan.md`. Planning spec: `PLAN_SPEC.md`.
- Standards: CELLxGENE dataset schema; Human Cell Atlas metadata schema; Cell Ontology (CL); HGNC;
  Ensembl; UBERON; MONDO/NCIT; Gebru et al., "Datasheets for Datasets."
- Archives: NCBI GEO; EBI ArrayExpress/BioStudies; Broad Single Cell Portal; HCA Data Portal; CZ
  CELLxGENE Discover; Zenodo; NCI GDC (open tier).
- *(Specific dataset accessions are produced by Stage-1 verification, not pre-approved here — see §7.1.)*

---

## Appendix A — Improvements applied

Twenty-five specific improvements made to the draft above, each already applied in-document:

1. **Led §7 with the binding cancer guardrails** (§7.0) so compliance is structurally first, not buried.
2. **Made Stage-1 license/access verification a hard gate** in the architecture (§6), with a compliance-reviewer **veto** (§8/§11), so no downstream work can bypass it.
3. **Refused to pre-list "approved" accessions** (§7.1): an accession is the *output* of verification, preventing accidental laundering of an unverified ID into "approved" status.
4. **Explicit exclude-by-default list** (dbGaP, EGA, TARGET/GDC controlled, biobanks; COSMIC/OncoKB non-commercial) in §5 and §7.1.
5. **Set `verifiedNeed=false` project-wide** and stated it in the summary, §2, §10, and risk table — honest about the missing partner.
6. **Added a human "why we are careful" preamble** acknowledging that each row is a child/family — per the prompt's "take real care."
7. **Defined Definition of Shipped as published-and-linked**, not merged (§8) — matching Hee-Lee Oss "delivered, not merged."
8. **Made provenance completeness a CI-enforced, release-blocking gate** (100% target, metric #6) rather than aspirational.
9. **Added a license-segregation rule** so a non-CC-BY upstream term can never contaminate a CC-BY release bundle (§7.4, risk table).
10. **Recorded the TS/ESM→Python exception explicitly** (L5, §16 Q6) instead of silently violating CLAUDE.md's engineering convention.
11. **Separated project risk tier (medium) from education risk tier (high)** with distinct gates (§8), avoiding a single misleading tier.
12. **Added an explicit integration-limitations requirement** (M3 exit, §6, risk table) so batch effects aren't mistaken for biology.
13. **Made reproducibility an independent-runner gate** (metric #5, M1/M4 exit), not just "we ran it once."
14. **Added a quarantine-on-suspicion control** for identifier-like fields (§7.3, §14) — defensive default.
15. **Chose deterministic integration (harmonypy) for v1** over stochastic scVI, with the tradeoff logged as an open question (§6, §16).
16. **Added "0 license/access violations" as a sev-1 outcome metric** (#7) — making compliance a measured success criterion.
17. **Specified output licensing precisely** (MIT code / CC-BY data+docs / link-only when redistribution not permitted) (§7.4).
18. **Added a decommission/persistence plan** (Zenodo DOIs persist; "unmaintained" labeling) so the work can't silently rot (§15).
19. **Built a thin, partner-independent M0** that delivers value (verified inventory + standards) before any heavy curation — de-risks the no-partner state.
20. **Added a partner-outreach task and gate** (§2, M0) so securing a beneficiary is tracked work, not a hope.
21. **Tightened non-goals** to bar new clinical claims, prognosis, linkage, and re-identification explicitly (§3, §5).
22. **Added comprehension-testing + "read as advice = 0" metric** for the education layer (metric #8) — measurable safety, not vibes.
23. **Made M5 formally conditional/deferrable** rather than assumed — no faked patient-facing scope without experts (L4, §9).
24. **Added a per-assertion `AnnotationAssertion` data object** requiring a source citation field (§6) so provenance is structural, not narrative.
25. **Added a public "what's covered / what's missing" ledger** to sustainability (§15) for honest coverage transparency, mirroring the portfolio's atltuae discipline.

---

## Review sign-off

**Reviewed for completeness & correctness on 2026-06-28.**

- **All 17 required sections present and in order** (§1–§17) per `PLAN_SPEC.md`; depth matches the Ofelia
  exemplar (positioning preamble, locked decisions L1–L6, explicit non-goals, phased roadmap with exit
  criteria, constraints-as-identity framing).
- **Cancer guardrails lead the data/compliance section** (§7.0) and are restated in summary, scope, and
  risk table — verified consistent: open-access only; controlled-access + identifiable data out of scope;
  per-source license verification; COSMIC/OncoKB flagged; "not medical advice" + oncologist & advocate
  review for `high`; provenance on every assertion.
- **Honesty checks:** `verifiedNeed=false` and "partner TO BE SECURED" stated wherever relevant; no
  beneficiary over-promises; CLAUDE.md TS/ESM deviation recorded as an explicit exception, not hidden.
- **Schema alignment:** roadmap milestones map 1:1 to `TASKS.md`; risk tiers, deliverable types, and
  lanes use schema-valid enums.
- **Corrections applied during review:** (a) clarified that listing an accession is an *output* of
  verification (removed any implication of pre-approved IDs); (b) added the compliance-reviewer veto so a
  failed license check blocks downstream work regardless of other approvals; (c) added explicit "0
  violations = sev-1 incident" language to tie metric #7 to the risk process; (d) made M5's deferral a
  formal, documented state rather than an implicit one.
- **Outstanding items requiring a human decision:** secure a partner/beneficiary (unblocks
  `verifiedNeed`); confirm oncologist + patient-advocate reviewers (gates M5); confirm the Python-
  pipeline exception with Hee-Lee Oss maintainers (§16 Q6).

**Sign-off:** Draft is internally consistent and safe to circulate for maintainer + compliance review.
Not yet approved for execution — execution is blocked until a maintainer and compliance reviewer are
assigned (§11).
