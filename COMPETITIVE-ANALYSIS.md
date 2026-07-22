# Competitive + Improvement Analysis — `ewing-single-cell-atlas`

> Analyst review, 2026-06-29. Grounded in web research with cited sources. Scope: an open,
> harmonized single-cell (scRNA-seq) atlas/index of Ewing sarcoma built only from open-access
> datasets, with uniform reprocessing/annotation + provenance. Outputs = derived embeddings,
> annotations, and code — not patient data.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually disciplined on governance, licensing, and "delivered not merged." Those parts
are strong and need no defense. The gaps are almost all **scientific/feasibility** gaps that the
governance framing partly masks.

### 1.1 The single biggest correctness error: "open-access ⇒ de-identified ⇒ safe" is false for scRNA-seq

The plan's §7.3 treats inputs as "already de-identified, aggregate single-cell expression" and frames
re-identification as something that happens only if a file contains obvious identifiers (names, dates,
geo). This is the most important factual error in the document. **A scRNA-seq count matrix is itself
quasi-genotype data.** Gene-expression levels at eQTL genes leak the donor's genotype; a recent *Cell*
paper demonstrated a linkage attack recovering donor identity from public scRNA count matrices with
~99.9% accuracy by predicting genotypes from expression and matching to a reference genotype panel
([Cell 2024, "Private information leakage from single-cell count matrices"](https://www.cell.com/cell/fulltext/S0092-8674(24)01030-4);
[Oxford NDT 2025 review](https://academic.oup.com/ndt/article/40/6/1077/7951974);
[policy framework, PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11938814/)). Implications the plan does
not yet address:
- A dataset being *posted under an open license on GEO* does not mean redistributing a **derived,
  re-harmonized count matrix** is privacy-neutral. The genotype-leakage risk travels with the matrix.
- The plan's safest defensible outputs are **embeddings, cluster-level/pseudobulk summaries, and
  annotations** — not raw or lightly-processed per-cell count matrices. The plan currently promises
  redistributable per-dataset `.h5ad` (Stage 3/6) as the default; that default should be inverted to
  "link to upstream raw; redistribute only reduced/aggregated derivatives," with an explicit
  genotype-leakage risk note in each datasheet. This is a substantive scope change, not a footnote.
- "Quarantine if a file contains identifiers" (§7.3) is necessary but nowhere near sufficient, because
  the leakage is in the expression values, not a metadata column.

### 1.2 Open Ewing scRNA-seq is genuinely scarce — and the plan never states the honest count

The plan repeatedly says "dozens of valuable scRNA-seq datasets" (§1) sit in archives. For Ewing
sarcoma specifically that is an **overstatement.** From the literature the realistic universe of
*primary-tumor* Ewing scRNA-seq studies is small and several are PDX or cell-line, not patient tumor:
- Aynaud et al. 2020, *Cell Reports* — 5 PDX, ~600–3,600 cells each
  ([ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2211124720300747)).
- EWSR1/FLI1-knockdown **cell line** scRNA (GSE171205)
  ([Springer](https://link.springer.com/article/10.1007/s13402-021-00640-x)).
- Visser/clonal-evolution cohort, 7 untreated patients, *Clin Cancer Res* 2025 / bioRxiv 2024
  ([CCR](https://aacrjournals.org/clincancerres/article/31/10/2010/762225/);
  [bioRxiv](https://www.biorxiv.org/content/10.1101/2024.01.18.576251v1.full)).
- Antigen-presenting-cell cohort, 18 samples/11 patients (GSE243347)
  ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC10595530/)).
- Bladder Ewing scRNA + spatial (GSE250523), 2024
  ([iScience](https://www.cell.com/iscience/fulltext/S2589-0042(24)02146-1)).
- Multimodal fusion-program study, 5 PDX + 6 patient tumors, eLife 2025
  ([eLife](https://elifesciences.org/reviewed-preprints/109124)).

That is on the order of **~6–10 studies**, a meaningful fraction PDX/cell-line, several small
(hundreds–few-thousand malignant cells), and a non-trivial share of the richest primary-tumor genomic
cohorts sit under **controlled access** (e.g. dbGaP phs000768) which the plan correctly excludes. The
plan's success metrics (§4) and milestones (≥3 datasets to integrate, "≥80% of verified-open
candidates") are achievable in count, but the plan never asks the load-bearing question: *is the
integrable malignant-cell population large and diverse enough to produce a reference atlas that adds
value beyond the individual papers?* This belongs in the executive summary, not buried. (See §8 Open
Questions: index + reprocessing-recipe scope may be the honest deliverable.)

### 1.3 Batch integration: method choice is under-justified and the metric is wrong

- The plan locks **harmonypy/Scanpy** for v1 and defers scVI/scANVI "for reproducibility/determinism"
  (§6, L-decisions, §16 Q2). But the benchmark literature (scIB, *Nature Methods* 2022) finds that when
  cell-type labels exist, **scANVI/scGen and scVI/Scanorama outperform Harmony on harder tasks**, and
  Harmony tends to **over-correct** — a serious hazard when the biological signal of interest (malignant
  EWSR1-FLI1 states across patients) is confounded with patient/batch
  ([Nature Methods](https://www.nature.com/articles/s41592-021-01336-8);
  [scIB site](https://theislab.github.io/scib-reproducibility/)). For tumor data, over-correction can
  literally erase the malignant heterogeneity the atlas exists to show.
- Determinism is overstated as a reason to avoid scVI: scVI is reproducible under a pinned seed +
  pinned CUDA/torch; "bit-for-bit" is the wrong reproducibility target for a stochastic-but-seeded
  model. Metric #5 ("bit-for-bit, or within tolerance") should be **explicitly tolerance-based** for
  integration outputs, with the tolerance defined.
- Missing: the plan has **no integration-quality benchmark.** It should adopt scIB-style metrics
  (batch mixing vs. bio-conservation: kBET, iLISI/cLISI, ARI, silhouette) and report them, rather than
  asserting a "limitations statement." Right now "honest limitations statement" is qualitative where it
  should be quantitative.

### 1.4 Malignant-vs-normal calling is entirely missing — a critical omission

A Ewing atlas's central scientific act is **separating malignant cells from TME/normal cells**, yet the
pipeline (Stages 3–5) never mentions CNV-based malignant calling (inferCNV, CopyKAT, SCEVAN, Numbat).
Without it, "annotate cell types against Cell Ontology" is insufficient — Cell Ontology has weak
coverage for *malignant* states, and cross-study malignant identification is exactly where errors
mislead researchers. The plan needs an explicit malignant-calling step with a named method, a normal
reference strategy, and known caveats (inferCNV needs designated normal references; CopyKAT/SCEVAN
self-identify diploids; allele-aware Numbat is best when raw data exist)
([tool comparison, MDPI](https://www.mdpi.com/2227-9059/12/8/1759);
[benchmark, Oxford BIB](https://academic.oup.com/bib/article/26/2/bbaf076/8051529)). Note the tension:
allele-aware methods want BAM/genotype-level data, which collides with the re-id guardrail — another
reason to foreground §1.1.

### 1.5 Cell-type annotation validity & reference mapping are under-specified

- The plan commits to Cell Ontology labels with per-label citations but does not say **how** labels are
  derived (marker-based, reference-mapping via Azimuth/scArches, or manual). Citing a source per label
  is provenance, not validity. There is no held-out validation, no confusion-matrix/agreement metric,
  no inter-annotator or method-vs-method concordance.
- No reference-mapping strategy is named. Azimuth/scArches/CELLxGENE Census label-transfer are the
  field standard and would let the atlas *map onto* a healthy reference to flag malignant/disease states
  ([Azimuth](https://satijalab.org/azimuth/); [scArches, Nat Biotech 2021](https://www.nature.com/articles/s41587-021-01001-7)).
  There is no pediatric/young-adult healthy bone/soft-tissue reference called out — and that absence is
  itself a finding worth stating.

### 1.6 QC / doublets / ambient RNA: named gate but no thresholds or methods

Stage 3 mentions "QC harmonization" and a "doublet handling" checkbox (TASKS data-002), but no method
(scDblFinder/DoubletFinder), no ambient-RNA correction (SoupX/CellBender), and no harmonized thresholds.
Harmonizing QC *across studies with different chemistries and depths* is one of the hardest, most
opinion-laden parts of atlas building and deserves its own documented, defensible recipe
([scDblFinder, PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC9204188/);
[sc-best-practices QC](https://www.sc-best-practices.org/preprocessing_visualization/quality_control.html)).

### 1.7 Compute scale is hand-waved

"Donated lane → near-zero marginal cost" (§15) is plausible *only because* the Ewing data is small. The
plan should say so explicitly (a few studies × thousands of cells fits on a laptop/free CI), and should
note where it would break (if scope expands to pan-sarcoma or spatial). Conversely, scVI/scANVI need GPU
— another input to the integration-method decision the plan currently resolves by assertion.

### 1.8 Other gaps
- **PDX/cell-line vs primary tumor** must be a first-class metadata axis and a hard caveat; mixing PDX
  and patient malignant cells in one embedding without flagging is a biology error waiting to happen.
- **Gene-ID/version harmonization across studies** (Ensembl release drift, gene symbol aliasing) is
  named but the failure modes (dropped genes, many-to-one mapping) are not.
- **No competitor/prior-art scan in the plan.** It does not mention ScPCA, 3CA, or CELLxGENE Census as
  existing harmonized resources — a serious completeness gap for a project whose entire value prop is
  harmonization (see §2–§3). This is arguably tied with §1.1 as the most important finding.
- **Metric #1 (≥250 downloads)** is a vanity-adjacent proxy the plan elsewhere disavows; tie reuse to
  *named* reuse events instead.

---

## 2. Competitive landscape (researched, cited)

### A. ScPCA — Single-cell Pediatric Cancer Atlas (CCDL / Alex's Lemonade Stand) — **most direct competitor**
- **What:** A data portal of **uniformly processed** 10x single-cell/single-nuclei RNA-seq from **700+
  samples across 55 pediatric cancer types — explicitly including Ewing sarcoma, osteosarcoma,
  rhabdomyosarcoma** — with open-source processing tools, downloadable objects, and a community-contribution
  intake. ([portal](https://scpca.alexslemonade.org/); [CCDL](https://www.ccdatalab.org/scpca);
  [manuscript, bioRxiv](https://www.biorxiv.org/content/10.1101/2024.04.19.590243v1)).
- **Strengths:** Exactly the harmonization-tax solution the plan proposes, already shipped, funded,
  maintained, pediatric-focused, open, with a brand families already trust (ALSF). Uniform pipeline +
  portal + tooling.
- **Weaknesses/gaps:** Pan-pediatric, **not Ewing-specialized** — no EWSR1-FLI1/ETS program scoring, no
  Ewing-specific annotation vocabulary, no cross-study *integrated* Ewing reference (it's per-sample
  uniform processing, not a fused atlas). Provenance is pipeline-level, not the per-assertion,
  license-verbatim model Hee-Lee Oss proposes. Re-id/genotype-leakage framing not foregrounded.
- **This is the comparator the plan must name and differentiate against.** The plan currently doesn't
  mention it at all.

### B. 3CA — Curated Cancer Cell Atlas (Weizmann, Tirosh/Gavish lab) — **key pan-cancer comparator**
- **What:** Curated, consistently re-annotated scRNA-seq across **124 datasets, 40+ cancer types, ~2,836
  samples**, with recurrent malignant expression programs (meta-programs), cell-cycle quantification,
  freely downloadable. ([3CA site](https://www.weizmann.ac.il/sites/3CA/);
  [Nature Cancer 2025](https://www.nature.com/articles/s43018-025-00957-8);
  [bioRxiv](https://www.biorxiv.org/content/10.1101/2024.10.11.617836v1)).
- **Strengths:** Gold-standard *malignant-program* curation, cross-cancer comparability, strong
  methods reputation, free download.
- **Weaknesses/gaps:** Breadth over depth — a rare cancer like Ewing gets shallow coverage (likely a
  handful of datasets, not the rare-cancer-specialist treatment). Curation is expert-manual and not
  built around machine-readable per-source license/provenance or re-id guardrails. Not pediatric-aware.

### C. CZ CELLxGENE Discover / Census (CZI) — the standard the plan builds on
- **What:** Free platform aggregating **44M+ human cells** under an enforced standardized schema
  (gene/cell/assay/donor), browsable + programmatic (TileDB-SOMA, AnnData/Seurat slices).
  ([portal](https://cellxgene.cziscience.com/); [NAR 2025](https://academic.oup.com/nar/article/53/D1/D886/7912032);
  [Census](https://chanzuckerberg.github.io/cellxgene-census/)).
- **Strengths:** De facto schema + hosting + browser the plan adopts; huge reach; CC-BY norm.
- **Weaknesses/gaps:** A general aggregator, not curator — no disease-specific harmonization, no
  malignant calling, no Ewing program scoring; quality is contributor-dependent. (Ally, not rival.)

### D. Human Cell Atlas — primarily a *normal*-tissue reference
- **What:** Global consortium + Data Portal of standardized healthy human cell reference maps.
- **Strengths:** Reference-grade normal atlases (the "what is normal bone/marrow/soft-tissue" backdrop a
  tumor atlas needs); strong metadata standards (shared with CELLxGENE).
- **Weaknesses/gaps:** Not cancer-focused; won't deliver malignant Ewing states. Best used *as the
  healthy reference* for mapping. ([CELLxGENE/HCA schema overview](https://github.com/chanzuckerberg/single-cell-curation)).

### E. Broad Single Cell Portal — host, not curator
- **What:** Hosting/visualization for study-level scRNA-seq; some Ewing/sarcoma studies live here.
- **Strengths:** Easy author deposition + interactive views.
- **Weaknesses/gaps:** **Mixed licenses, some require agreement** (the plan already flags this); no cross-
  study harmonization, no uniform reprocessing, per-study schemas. A *source*, not a competitor product.

### F. UCSC Cell Browser — viewer, not atlas
- **What:** Lightweight browser to publish/visualize processed scRNA-seq collections.
- **Strengths:** Trivial to stand up an Ewing browser; good for the "browse" last mile.
- **Weaknesses/gaps:** No harmonization, annotation standard, or provenance model — purely presentation.

### G. Tabula Sapiens / Tabula Muris — normal multi-organ references
- **What:** Benchmark multi-organ healthy cell atlases (CZI-funded).
- **Strengths:** Clean, well-annotated normal references for label transfer / malignant contrast.
- **Weaknesses/gaps:** No cancer, no pediatric-specific bone/soft-tissue; reference, not competitor.

### H. Method/tooling layer (not products, but they set the quality bar the atlas is judged against)
- **scVI / scANVI** (scvi-tools): probabilistic integration + label transfer; top-tier on hard tasks
  ([scIB, Nat Methods](https://www.nature.com/articles/s41592-021-01336-8)).
- **Harmony / harmonypy:** fast linear integration; risk of over-correction on tumor data (the plan's v1 pick).
- **Azimuth** (Satija lab): reference-mapping annotation app, h5ad-compatible
  ([Azimuth](https://satijalab.org/azimuth/)).
- **scArches** (Theis lab): transfer-learning reference mapping *without sharing raw data* — directly
  relevant to the re-id guardrail ([Nat Biotech 2021](https://www.nature.com/articles/s41587-021-01001-7)).
- **inferCNV / CopyKAT / SCEVAN / Numbat:** malignant-vs-normal calling — the missing pipeline stage
  ([benchmark, Oxford BIB](https://academic.oup.com/bib/article/26/2/bbaf076/8051529)).
- **GEO / ArrayExpress:** the raw archives the inventory draws from.

---

## 3. Gaps we can fill (the Ewing-specific, open, harmonized, re-id-aware angle)

3CA and **ScPCA are the key comparators** — and crucially, *neither closes these gaps*:
1. **Rare-cancer depth, not breadth.** Be the single best-curated *Ewing* resource: every public open
   study found, harmonized, and integrated — depth ScPCA (pan-pediatric, per-sample) and 3CA (pan-cancer)
   structurally won't provide.
2. **EWSR1-FLI1/ETS program scoring with provenance.** Neither competitor scores cited fusion-target
   programs per cell with a citation per signature. This is the distinctly-Ewing value layer.
3. **A genuinely *integrated* cross-study Ewing reference** (mapping target via Azimuth/scArches/Census),
   versus ScPCA's per-sample uniform processing.
4. **Re-identification-aware release engineering.** A first-class, documented stance on genotype leakage
   from count matrices — redistributing aggregated/derived layers, not raw matrices — that *no* existing
   atlas foregrounds. This is both a safety contribution and a differentiator.
5. **Machine-readable per-source license + per-assertion provenance** (Hee-Lee Oss's model) — stronger and more
   auditable than 3CA's manual curation or ScPCA's pipeline-level provenance.
6. **An honest "what's covered / what's missing" coverage ledger** for a rare cancer — the field has
   nothing like it for Ewing.

---

## 4. Differentiators to win

1. **Rare-cancer specialist + re-id-aware** — the intersection nobody occupies: Ewing depth *and* a
   defensible genotype-leakage posture (derived layers only, documented per dataset).
2. **Provenance as a product, not a footnote** — per-assertion citations + per-source verbatim licenses,
   CI-enforced, release-blocking. Auditable in a way curated atlases are not.
3. **EWSR1-FLI1 program layer** — citable, license-clean fusion-target signatures scored per cell, tying
   the atlas to the disease's defining biology (and to sibling projects `ewsr1-fli1-knowledge-graph`).
4. **Reproducible-from-open-inputs pipeline** an independent runner can re-execute — stronger
   reproducibility contract than "download our objects."
5. **Honest scope + coverage ledger** — explicitly an *index + reprocessing recipes where redistribution
   is barred*, which builds trust precisely because it doesn't overclaim.
6. **Built to be copied** — a rare-cancer atlas *template* (see §7) other small-cohort cancers reuse.

---

## 5. Claude API leverage

### Where Claude clearly helps (with human verification gates)
1. **Metadata harmonization** — map heterogeneous study metadata (free-text tissue/disease/assay/donor
   fields, supplementary spreadsheets) to CELLxGENE/HCA schema + CL/UBERON/MONDO/NCIT ontology terms,
   emitting candidate mappings *with the ontology ID and a confidence + rationale* for human review.
   High-volume, judgment-light-but-tedious — Claude's sweet spot.
2. **Pipeline / annotation / QC code generation** — draft the scanpy/anndata reprocessing, QC, doublet
   (scDblFinder), ambient (SoupX/CellBender), malignant-calling (inferCNV/CopyKAT) and integration
   scaffolds; write `cellxgene-schema` validators, provenance-JSONL emitters, and CI gates.
3. **Marker-set & EWSR1-FLI1 signature curation *with provenance*** — extract gene signatures from the
   literature, attach the citation + license status to each list, and flag non-commercial-sourced lists
   (COSMIC/OncoKB) for exclusion. Output is a *sourced candidate*, expert-confirmed before use.
4. **License/access triage drafting** — read a GEO/ArrayExpress/SCP record and draft the `SourceRecord`
   (license verbatim + SPDX-ish tag, access tier, redistribution flag, de-id evidence) for the compliance
   reviewer to approve — never auto-approve.
5. **QC report, datasheet (Gebru-style), integration-report, and "limitations" drafting** — turn pipeline
   outputs + metrics into readable, sourced prose for human sign-off.
6. **Coverage-ledger and changelog maintenance**, and drafting the conditional education explainer
   *for* (never instead of) oncologist + advocate review.

### Where Claude must NOT decide (hard lines)
- **Cell-type and malignant-vs-normal calls** come from validated methods (CNV inference, reference
  mapping, marker evidence) **+ domain-reviewer sign-off** — *not* an LLM's read of expression. Claude may
  summarize and cross-check; it does not assign the final label.
- **Re-identification / consent / access-tier determinations** are human-verified. Claude drafts the
  triage; a compliance reviewer makes the call. The genotype-leakage risk decision (what is safe to
  redistribute) is explicitly a human gate.
- **No fabricated results** — Claude must never invent a citation, a marker, a metric, or a dataset
  accession. Every assertion it emits carries a checkable source or is labeled a hypothesis for review.
- **No clinical/biological novelty claims** and **nothing patient-facing without dual expert sign-off.**
- See `claude-api` skill for current model IDs/pricing if any funded-lane reanalysis is ever scoped.

---

## 6. Ten concrete optimizations

1. **Invert the redistribution default to "derived/aggregated layers only"** and add a genotype-leakage
   risk note to every datasheet (addresses §1.1). Redistribute embeddings, annotations, pseudobulk, and
   reduced/normalized layers; link to upstream raw count matrices rather than re-hosting them.
2. **Add an explicit malignant-vs-normal calling stage** (Stage 3.5) with a named method (e.g. CopyKAT or
   SCEVAN as primary, inferCNV cross-check), a documented normal-reference strategy, and PDX/cell-line
   handling.
3. **Replace "harmony, because deterministic" with a benchmarked choice**: run Harmony *and* scVI/scANVI,
   report scIB metrics (iLISI/cLISI, kBET, ARI, bio-conservation), and pick per evidence; pin seeds and
   define a numeric tolerance instead of "bit-for-bit."
4. **Adopt a reference-mapping backbone** (Azimuth or scArches onto a healthy bone/soft-tissue + immune
   reference) so labels are transferred + validated, not just asserted — and so scArches's "no raw-data
   sharing" property reinforces the re-id stance.
5. **Specify the QC recipe**: scDblFinder doublets, SoupX/CellBender ambient correction, harmonized
   mito%/min-genes/min-counts thresholds *documented per chemistry*, with before/after cell counts logged.
6. **Add quantitative integration-quality metrics** to the release (not just a prose limitations
   statement); block release if batch-mixing erases known malignant heterogeneity.
7. **Name ScPCA, 3CA, and CELLxGENE Census in the plan** as prior art, and define the differentiation +
   an interop path (e.g. publish to a CELLxGENE collection; cross-link ScPCA Ewing samples).
8. **Make "primary tumor vs PDX vs cell line" a required obs field and a hard caveat** in every
   integrated view; never silently co-embed.
9. **Reframe metric #1** from raw downloads to *named reuse events* + a CELLxGENE collection that records
   real sessions; demote count metrics consistently with the plan's own anti-vanity stance.
10. **Front-load a feasibility gate in M0**: before M1, produce a one-page "is there enough integrable
    open malignant-cell data for a true atlas, or is this an index + recipes?" decision memo
    (see §1.2, §8), and let it set v1 scope.

---

## 7. Parallel & perpendicular spin-offs

- **`ewing-expression-reanalysis` (sibling, bulk):** the open *bulk* RNA-seq counterpart; pseudobulk from
  this atlas validates against bulk cohorts and vice-versa — shared harmonization + provenance tooling.
- **`ewsr1-fli1-knowledge-graph` (sibling):** consumes this atlas's per-cell EWSR1-FLI1/ETS program scores
  as evidence edges; the atlas becomes a single-cell evidence layer for the graph. Tight, natural coupling.
- **`ewing-open-data-catalog` (sibling):** the verified `SourceRecord` inventory *is* a catalog feed —
  factor source discovery/verification into a shared catalog the whole Ewing portfolio reuses.
- **Generalized rare-cancer single-cell atlas template:** package the gated pipeline (verify → standardize
  → QC → malignant-call → annotate → integrate → release) as a reusable template other small-cohort
  cancers (rhabdomyosarcoma, DSRCT, osteosarcoma) instantiate — directly leveraging the §4 "built to be
  copied" differentiator and mirroring ScPCA's community-contribution model.
- **MCP server serving annotated embeddings:** an MCP endpoint exposing the atlas (query cells/programs,
  return annotated embeddings + provenance, *not* raw matrices) so agents/tools consume it safely — the
  re-id-aware design makes an API surface defensible.
- **Healthy young-adult/pediatric bone & soft-tissue reference (perpendicular):** if no good reference
  exists for malignant-vs-normal mapping, curating one (from open HCA/Tabula data) is a valuable spin-off
  benefiting many pediatric-cancer atlases.
- **scRNA re-identification-risk checklist/tool (perpendicular):** generalize the genotype-leakage stance
  into an open checklist + linter for *any* single-cell redistribution — a methods contribution beyond Ewing.

---

## 8. Open questions for the maintainer

1. **Is there enough OPEN Ewing scRNA-seq to build a meaningful *atlas*?** Honest assessment: the
   integrable, primary-tumor, open universe is roughly **6–10 studies, several PDX/cell-line, several
   small, with the richest primary cohorts often controlled-access** (§1.2). That is likely enough for a
   valuable **index + uniform reprocessing recipes + program-scoring layer**, but probably **not yet
   enough for a definitive integrated reference atlas** that beats reading the source papers. Strong
   recommendation: **scope v1 as "verified index + reprocessing recipes + harmonized derivatives +
   EWSR1-FLI1 program layer," and treat the fused reference atlas as a stretch goal gated on a feasibility
   memo.** Re-title accordingly so the name doesn't overpromise.
2. **Genotype-leakage posture:** given the *Cell* 2024 linkage result, do we redistribute *any* per-cell
   count matrix, or only embeddings/aggregates + links? (Recommend the latter.) Who is the human owner of
   that determination, and is it added as a hard CI/release gate?
3. **Malignant calling:** which method is primary, and what is the normal-reference strategy when a study
   ships no normal cells? How are PDX (mouse-contaminated) and cell-line data handled/flagged?
4. **Integration method:** are we willing to run the scVI/scANVI-vs-Harmony benchmark and let evidence
   (not determinism dogma) decide? Do we have GPU access if scANVI wins?
5. **Reference for mapping:** is there an acceptable open pediatric/young-adult bone & soft-tissue + immune
   reference, or do we need to assemble one (spin-off in §7)?
6. **Relationship to ScPCA and 3CA:** do we *complement* (Ewing-deep layer + program scoring on top of
   ScPCA-processed objects) or *re-process independently*? Complementing may be faster, more honest, and
   avoid duplicated effort — but creates a dependency. Which?
7. **Program-score licensing (§16 Q8 in plan):** which specific EWSR1-FLI1/ETS signatures are citable *and*
   license-clean to redistribute as gene lists (avoiding COSMIC/OncoKB contamination)?
8. **Conflicting cross-study labels & PDX-vs-patient differences:** what is the tie-break policy, and is
   disagreement recorded as data (not silently resolved)?
9. **Partner:** ScPCA/ALSF and Ewing/sarcoma foundations are obvious outreach targets — could one double
   as both the `verifiedNeed` partner *and* a data/interop collaborator?
