# Competitive & Improvement Analysis — `ewing-literature-corpus`

*Prepared 2026-06-29. Web-grounded; sources cited inline. Scope: the Hee-Lee Oss cancer-research good-deed
project that assembles a license-clear, provenanced, machine-readable corpus of Ewing sarcoma
open-access literature (PMC OA Subset / Europe PMC) with grounded structured extraction, for
text-mining / meta-research. Cancer guardrails: open-access / license-clear text only; provenance +
license per item; no paywalled full text; no patient data.*

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on compliance framing — it already leads with binding cancer guardrails,
a per-article license bucket table, a deterministic groundedness re-location check, a three-tier risk
model, and quality-gated (not vanity) metrics. The findings below are the *remaining* gaps, errors,
and under-specified pieces. Several are material.

### A. License-tier model has real but fixable errors

1. **The NoDerivatives row is mis-bucketed and internally inconsistent — this is the single most
   important license error.** NCBI's own PMC OA license grouping puts **CC-BY-ND in the
   *Commercial Use Allowed* group** (alongside CC0/CC-BY/CC-BY-SA), and CC-BY-NC-ND in the
   *Non-Commercial* group — the ND axis is orthogonal to the commercial axis
   ([PMC OA list](https://pmc.ncbi.nlm.nih.gov/tools/openftlist/)). The plan's table treats ND as a
   near-exclusion ("metadata + short grounding quotes only, no full text"). That is a *defensible
   conservative policy choice*, but the plan must say so explicitly rather than implying ND means
   "not redistributable" — because **the ND full text *is* in the OA Subset bulk packages and *is*
   redistributable verbatim; what ND forbids is distributing a *derivative*.** A text-mining corpus
   that stores the verbatim full text (no modification) is arguably permitted even under ND, while
   the *extraction layer* (a derivative) is not. Conversely, the plan's claim that NC content is fine
   because "our commons output is non-commercial" is shaky: the corpus is released **CC-BY-4.0**,
   which **permits commercial reuse** — mixing NC source text into a CC-BY-labeled artifact is a
   latent license conflict. The corpus must be a faithfully *mixed-license* artifact where NC text
   stays NC and ND text stays non-derivative; a blanket "our added content is CC-BY-4.0" is not
   enough. **Action: split the policy into two orthogonal axes (derivatives y/n, commercial y/n),
   decide separately for (a) verbatim full text and (b) the derived extraction, and stop implying the
   whole corpus can be CC-BY.**

2. **"PMC OA Subset is the only full-text source" silently excludes — but the plan must name the
   trap — the PMC Author Manuscript Collection (NIHMS).** These NIH-public-access author manuscripts
   are in PMC but **not in the OA Subset**, **frequently carry no license statement**, and are
   available only "to read, text mine, and for other uses consistent with the principles of Fair
   Use" — *not* for redistribution ([PMC FAQ](https://pmc.ncbi.nlm.nih.gov/about/faq/),
   [PMC Copyright](https://pmc.ncbi.nlm.nih.gov/about/copyright/)). NCBI explicitly states "systematic
   downloading … from the main PMC website … is prohibited." A naive Entrez/PMC query for "Ewing
   sarcoma" will return many author-manuscript and non-OA records; the plan needs an explicit,
   tested **OA-membership check** (is this PMCID in the OA Subset file list / returned by the OA Web
   Service?) as a hard precondition, *separate from* license parsing. This is currently implied but
   not made into a discrete, testable gate step.

3. **"Facts aren't copyrightable" is invoked to justify ND grounding quotes — partly right, but the
   reasoning is loose.** Individual facts aren't copyrightable, but **verbatim quoted expression
   is**, and the plan stores *verbatim quotes* (≤25 words) from ND articles. The defense is therefore
   *fair use / fair dealing of short quotation for provenance*, **not** "facts aren't copyrightable."
   The 25-word/200-word caps are reasonable but **legally arbitrary** until a reviewer signs them;
   the plan already flags this as an open question — good — but it should reframe the legal basis as
   "minimal quotation for citation/verification" rather than the copyrightability-of-facts argument,
   which doesn't cover the quoted text itself.

### B. Source-coverage and dedup gaps

4. **Europe PMC is named in the project brief but absent from the plan's pipeline.** The plan's
   retrieval is PMC-OA-only (NCBI). Europe PMC has a *larger* OA full-text set, its own **Annotations
   API / SciLite** (gene, disease, organism, chemical annotations under CC-BY/CC-BY-NC/CC0/OA only —
   [Europe PMC Annotations](https://europepmc.org/Annotations)), and a published **annotated
   full-text gold corpus** ([Nature Sci Data 2023](https://www.nature.com/articles/s41597-023-02617-x)).
   Either Europe PMC is in scope (then dedup across NCBI-PMC and Europe PMC by PMCID/PMID/DOI is
   needed and its TDM terms must be captured per-article) or it is explicitly deferred. The plan
   should not leave it ambiguous, and should note that Europe PMC's annotations could seed the gold
   set rather than annotating from scratch.

5. **Dedup is under-specified for the hard cases.** The plan dedups "by PMCID/DOI." Real
   duplicates/near-duplicates include: preprint↔published version pairs (different DOIs, same
   content); PMC vs Europe PMC copies of the same article; corrections/errata; versioned preprints;
   and articles with no DOI. Dedup needs a **clustering key hierarchy** (PMID > PMCID > normalized
   DOI > title+author+year fuzzy match) and a documented rule for which copy is canonical (and which
   license governs when copies differ). This matters because **license can differ between two copies
   of the "same" paper.**

6. **Retraction/erratum handling is mentioned only in "Sustainability," not in ingestion.** A
   corpus for meta-research must flag retracted/withdrawn/expression-of-concern articles *at build
   time*, not just on refresh. PubMed carries `RetractionIn`/`CommentsCorrections`; this should be a
   first-class field in the corpus record schema (it currently is not).

### C. Annotation / extraction-schema gaps

7. **Inter-annotator agreement (IAA) target is named but the protocol is thin.** The plan sets
   Cohen's κ ≥ 0.7 on the gold set — but Cohen's κ is for **two** annotators on **categorical** labels
   and is a poor fit for **span-based NER** (boundary disagreements, nested entities) and for **>2
   annotators**. For NER the field standard is **span-level F1 between annotators** and/or
   **Krippendorff's α**; relation/assertion IAA needs its own measure. The plan should specify: unit
   of agreement (entity span? normalized ID? assertion?), the measure per task, adjudication
   protocol, and a double-annotated fraction. As written, "κ ≥ 0.7" is likely to be applied to the
   wrong unit.

8. **Entity-normalization F1 ≥ 0.85 lacks a denominator definition.** Is F1 measured on (a) mention
   detection, (b) correct entity *typing*, or (c) correct *ontology ID linking* (the hard part)? The
   target almost certainly means normalization/linking, which is much harder than 0.85 for some types
   (cell lines and drugs link well via Cellosaurus/ChEBI; novel fusion variants and ambiguous gene
   symbols do not). One blended 0.85 number hides this. Report **per-entity-type** P/R/F1 with
   per-type targets.

9. **The in-repo "controlled fusion-type vocabulary" is a maintenance and correctness liability.**
   Hand-maintaining EWSR1-FLI1 type 1/2, EWSR1-ERG, etc. mapped to HGNC is fine, but the plan should
   (a) cite an authoritative source for the breakpoint typing, (b) define how a *novel* fusion
   reported in a paper is handled (flagged `unmapped`, not silently dropped), and (c) decide whether
   fusions normalize to a recognized standard (e.g., a gene-fusion nomenclature) rather than a bespoke
   list that downstream consumers must re-learn.

10. **"Aggregate vs individual" is a binary, but the boundary is fuzzy.** The schema enforces
    `aggregateOnly:true`, yet a case *series* of n=3, or "a 14-year-old male with EWSR1-FLI1 …,"
    blurs aggregate vs individual. The PII methodology is good (quasi-identifier flagging) but the
    schema offers no field for *cohort size* or *case-series* disposition, and no rule for the n≈2–5
    grey zone. Add an explicit small-n policy and a `cohortSize`/`evidenceType` field.

### D. Metrics, scope, and process gaps

11. **The headline "≥400 articles" is fragile relative to the eligible population.** Ewing is rare;
    the *open-access, license-clear, English, full-text* slice of Ewing literature may be only a few
    hundred to low thousands of articles. The plan should estimate the realistic OA-eligible
    denominator (a quick PMC-OA count for the retrieval query) so 400 is shown to be both achievable
    *and* meaningful coverage, not an arbitrary number. Right now there is no denominator.

12. **Versioning/DOI model is named (Zenodo) but the *semantics* are unspecified.** For a living
    corpus, the plan needs: Zenodo **concept DOI** (all versions) vs **version DOI** (each release);
    a SemVer policy for *content* changes (article added/removed, license re-classified, extraction
    re-run); and a stable per-record ID that survives re-releases. "Released with a DOI" is necessary
    but not sufficient for reproducible meta-research citation.

13. **Redistribution rights of the *derived* corpus are asserted but not fully reasoned.** Even for
    CC-BY source text, the derived extraction inherits attribution obligations; for CC-BY-SA source,
    the derivative inherits **ShareAlike** (the plan notes this) — which **conflicts** with releasing
    the whole extraction layer as CC-BY-4.0. A single uniform output license cannot be correct across
    a mixed-license corpus. The plan needs a per-record output-license computation, not one global
    statement.

14. **No explicit handling of non-English literature or figures/tables/supplements.** Ewing
    research includes non-English OA articles and data-rich tables/figures. Scope should state these
    are in or out (recommend: English-only v1, text-only, no figure images — which also helps the
    no-identifiable-images guardrail).

15. **"Verified need / consumer" remains the project's central risk and is honestly flagged — good
    — but the Zenodo fallback partly defeats the "delivered, not merged" bar.** Self-publishing to a
    DOI you control is *release*, not *reuse*; the plan's own metric correctly says self-reported
    reuse doesn't count, yet M0's fallback lets the pilot "ship" via Zenodo. This tension should be
    named: Zenodo = released artifact exists; it is **not** a delivered-to-beneficiary outcome until
    the KG project or an external lab actually ingests it.

---

## 2. Competitive landscape (web-grounded)

No existing resource is a *Ewing-specific, license-tiered, provenance-per-assertion, redistributable*
corpus. The space is occupied by large general infrastructures, one canonical disease-specific
exemplar (CORD-19), and adjacent tools. Each below: what it does / strengths / weaknesses vs this
project.

**1. PMC Open Access Subset (NCBI)** — the upstream source itself.
[openftlist](https://pmc.ncbi.nlm.nih.gov/tools/openftlist/).
*Does:* ~3.4M+ full-text OA articles/preprints, bulk + OA Web Service + BioC, grouped into
**Commercial / Non-Commercial / Other** license tiers. *Strengths:* authoritative, machine-readable
license per article, BioC format, the legitimate redistribution base. *Weaknesses (= our gaps):* not
disease-curated; license tiers are coarse (ND sits inside "Commercial"); no Ewing retrieval, no
extraction, no dedup vs Europe PMC, no datasheet. It is raw material, not a corpus.

**2. Europe PMC + SciLite Annotations API + annotated full-text corpus** —
[Annotations](https://europepmc.org/Annotations),
[Sci Data 2023](https://www.nature.com/articles/s41597-023-02617-x).
*Does:* OA full text plus text-mined annotations (genes/proteins, diseases, organisms, chemicals,
GO, PPIs) served as W3C Web-Annotation JSON-LD; annotations provided **only** for CC-BY / CC-BY-NC /
CC0 / OA articles. Publishes a gold annotated corpus. *Strengths:* mature, license-aware annotation
delivery; community submission; a ready gold-set seed; standards-based. *Weaknesses:* annotations are
generic (not Ewing/fusion-aware), automated (not expert-verified per assertion), abstract-and-OA
scoped; no aggregate-only PII triage; no rare-cancer focus; no per-assertion verification status.

**3. PubTator Central / PubTator3 (NCBI)** —
[NAR 2019](https://academic.oup.com/nar/article/47/W1/W587/5494727),
[PMC6602571](https://pmc.ncbi.nlm.nih.gov/articles/PMC6602571/).
*Does:* automated bioconcept annotation (genes, diseases, chemicals, mutations, species, **cell
lines**) over 29M+ PubMed abstracts and 3M+ PMC full texts; BioC-XML/JSON + REST + FTP; daily sync.
*Strengths:* exactly the six entity types we need (incl. cell lines), free, standard formats, huge
scale, relation extraction in v3. *Weaknesses:* no provenance-grade quote spans tied to a
verification workflow; no license tiering of *redistribution* (it's an annotation service, not a
redistributable corpus); no Ewing fusion vocabulary; not expert-verified; no PII/case-report triage.
**This is our closest "annotation" overlap — and a feeder we should consume, not rebuild.**

**4. CORD-19 (Allen Institute / Semantic Scholar)** — the disease-specific exemplar to emulate.
[allenai.org/data/cord-19](https://allenai.org/data/cord-19),
[Zenodo](https://zenodo.org/records/3715506).
*Does:* curated COVID-19 corpus, ~47K+ articles (36K+ full text) from PubMed, WHO, bioRxiv/medRxiv,
one JSON object per paper, weekly updates, hugely used (TREC-COVID). *Strengths:* proves a
disease-scoped open corpus drives an entire research ecosystem; clean JSON; versioned. *Weaknesses
(our differentiators):* **CORD-19 was criticized for license heterogeneity and including
non-redistributable content**; no per-assertion grounding; no expert verification; coarse provenance.
We are "CORD-19 for a rare cancer, but license-clean and provenance-first."

**5. S2ORC (Allen Institute)** —
[arXiv 1911.02782](https://arxiv.org/abs/1911.02782),
[github allenai/s2orc](https://github.com/allenai/s2orc).
*Does:* 8.1M open-access full-text papers with structured full text + linked citations, released
**ODC-BY 1.0**. *Strengths:* huge, structured, single clear license, great for NLP pretraining.
*Weaknesses:* general (all fields), one blanket license (doesn't preserve per-article CC terms the way
a faithful mixed-license corpus must), no biomedical entity normalization, no Ewing focus, no PII
stance.

**6. OpenAlex (OurResearch)** —
[Wikipedia](https://en.wikipedia.org/wiki/OpenAlex),
[developers](https://developers.openalex.org/).
*Does:* fully open **CC0** catalog of 477M+ works (metadata, authors, institutions, concepts, OA
status), API + fortnightly dumps. *Strengths:* best-in-class *metadata*, CC0, OA-status flags, free,
no auth. *Weaknesses:* **metadata/abstracts, not full text**; no extraction; no license-tiered full
text. Ideal as our **discovery/metadata backbone** (find Ewing works, get OA status) — complement,
not competitor.

**7. LitCovid (NCBI)** — the curated-disease-hub model.
[ncbi.../coronavirus](https://www.ncbi.nlm.nih.gov/research/coronavirus/),
[NAR 2021](https://academic.oup.com/nar/article/49/D1/D1534/5964074).
*Does:* daily-updated curated COVID-19 literature hub (~300K articles), topic categories +
drug/geo extraction, automated search + human review. *Strengths:* proves automated-retrieval +
human-curation at scale with topic labels; freely available. *Weaknesses:* abstract-level
categorization, not a redistributable full-text corpus with per-assertion provenance; COVID-only;
huge-disease economics don't transfer to a rare cancer (but the *model* does — at smaller scale).

**8. BioC / PubTator BioC format** —
[arXiv 1804.05957](https://arxiv.org/pdf/1804.05957).
*Does:* community text-mining interchange (offsets, passages, annotations). *Strengths:* the right
interop standard; the plan correctly adopts it. *Weakness:* a format, not a corpus — no competitive
threat, a dependency.

**9. Hugging Face biomedical corpora (e.g., `pmc/open_access`, `allenai/cord19`)** —
[pmc/open_access](https://huggingface.co/datasets/pmc/open_access).
*Does:* hosts bulk PMC OA and disease corpora as ML-ready datasets, increasingly with **Croissant**
metadata. *Strengths:* distribution + discoverability + Croissant (which the plan targets).
*Weaknesses:* mirrors of upstream with the same coarse licensing; no curation, no verification.
A **distribution channel** for us, not a competitor.

**10. BioASQ** —
[BioASQ 2023 overview](https://arxiv.org/pdf/2307.05131).
*Does:* annual benchmark for biomedical semantic indexing + QA; gold Q/A with relevant snippets.
*Strengths:* the model for **gold-set + eval-harness** design (gold docs + gold snippets, expert-made).
*Weakness:* general benchmark, not a corpus; informs our eval methodology rather than competing.

**11. `pubmed_parser` (titipata)** —
[github](https://github.com/titipata/pubmed_parser).
*Does:* Python parser for PMC OA XML / MEDLINE / E-utils. *Strength:* exactly the parsing layer we
need (Python; we're TS — note interop). *Weakness:* a library, not a corpus; no licensing/PII logic.
Reference implementation for our ingestion.

**12. Adjacent disease/rare-disease corpora — RareDis, CANTEMIST, EBM-PICO.**
[RareDis / LLM-NER arXiv 2508.09323](https://arxiv.org/pdf/2508.09323),
[EBM-PICO arXiv 1806.04185](https://arxiv.org/pdf/1806.04185).
*Does:* **RareDis** = 1,041 docs from NORD (rare-disease entities/relations); **CANTEMIST** =
1,301 oncology clinical case reports (Spanish) with tumor-morphology coding; **EBM-PICO** =
PICO-annotated medical abstracts. *Strengths:* prove demand for disease-/rare-disease-specific
annotated corpora and offer schema precedent. *Weaknesses (our gap):* none is Ewing-specific; CANTEMIST
is **clinical case reports** (exactly the PII surface we exclude) and Spanish; RareDis is NORD prose,
not the primary OA literature with per-article license tiering and provenance. **Web search found no
existing Ewing-specific annotated, license-tiered literature corpus** — confirming the niche is open.

---

## 3. Gaps we can fill

The defensible, non-redundant position is the **intersection** no incumbent occupies:

- **Rare-cancer-specific depth.** General infra (PubTator, S2ORC, OpenAlex) is broad and shallow on
  Ewing; we are narrow and deep: EWSR1-ETS fusion typing, Ewing cell-line panel (A673/SK-N-MC/TC-71…),
  Ewing-relevant drugs/targets. None of the giants encode fusion-type semantics.
- **License-clean *and* faithfully mixed-license.** CORD-19/S2ORC chose blanket licenses and took
  heat for it; we keep each article under its true license and compute redistribution per record —
  the corpus a cautious lab or knowledge-base can actually reuse without re-doing licensing.
- **Provenance-per-assertion with enforced groundedness.** PubTator/SciLite give annotations; we give
  *verifiable* assertions (verbatim span + offset + automated re-location check + human/expert
  sign-off). That's the meta-research / KG-substrate gap.
- **Aggregate-only PII triage as a first-class gate.** No incumbent does rare-disease
  quasi-identifier triage; CANTEMIST is the opposite (clinical cases). For a pediatric/AYA cancer this
  is both an ethical differentiator and a trust signal.
- **Expert-verified biomarker/outcome assertions.** A small body of oncologist-signed, provenanced
  aggregate findings is something no automated pipeline offers.
- **Datasheet + Croissant + Zenodo DOI** as a citable, ML-ready, reproducible artifact tailored to
  one disease — turnkey for the sibling KG project and small labs.

---

## 4. Differentiators to win

1. **Provenance-first, not coverage-first.** "Every assertion re-locatable to a verbatim span or it's
   rejected" — structurally hallucination-resistant; the trust story incumbents can't match.
2. **Honest mixed-license corpus** with per-record redistribution computation (vs blanket-license
   competitors) — the *legally safe* Ewing corpus.
3. **Domain specialization** (fusion vocabulary, Ewing cell-line/drug panels) that general corpora
   lack and that the downstream KG actually needs.
4. **Ethics as a feature:** aggregate-only, quasi-identifier-aware PII triage for a pediatric cancer.
5. **Composable, standards-native** (BioC + Croissant + Zenodo DOI + Datasheet) so it slots into
   existing toolchains rather than inventing a silo.
6. **Built to be consumed by a named sibling project** — outcome-anchored, not artifact-anchored.

---

## 5. Claude API leverage

**Where Claude clearly helps (assistive, human-verified):**

- **Grounded NER + relation/assertion extraction.** Claude proposes entity mentions, EWSR1-ETS fusion
  types, cell lines, drugs, and aggregate gene–phenotype/gene–drug/model-finding assertions, each
  required to return the **verbatim supporting span**; the deterministic re-location check then
  accepts/rejects. Claude's long context lets it read full IMRaD sections at once and respect
  section structure. (Confirm current model IDs/pricing/limits via the `claude-api` skill before
  building the funded-lane runner — do not hardcode from memory.)
- **Metadata normalization & entity linking.** Map surface forms → HGNC / MONDO / Cellosaurus / ChEBI
  candidate IDs with a confidence and the evidence string; ambiguous/unmapped flagged for human
  review (Claude proposes, human disposes).
- **Dedup / near-duplicate adjudication.** Decide whether two records (preprint vs published, PMC vs
  Europe PMC) are the same work and which license governs — as a *suggestion* with rationale.
- **PII / case-report triage assistance.** First-pass detection of case-report markers and
  quasi-identifiers ("a 14-year-old…", dates, geography) to *raise flags* for the License+PII reviewer.
- **Gold-set bootstrapping.** Draft candidate annotations to accelerate human annotators (who then
  adjudicate to the gold standard) — reducing cost while IAA is still measured on human labels.
- **Summarization for *internal* navigation only** (e.g., section gisting to help reviewers), never
  shipped as synthesis or advice.
- **Schema/Datasheet/Croissant drafting** and retrieval-query expansion (MeSH + fusion synonyms),
  human-tuned against a seed set.

**Where Claude must NOT decide (hard human-verified boundaries):**

- **License & redistribution determinations stay human-verified.** Claude may *parse* a license URL
  and *propose* a bucket, but the binding redistribution decision (esp. ND/NC edge cases, NIHMS
  author-manuscript fair-use) is the License+PII reviewer's call. Default-to-conservative on any
  model uncertainty.
- **No assertion ships on Claude's authority.** The source paper is the only authority; every
  accepted assertion needs the automated re-location check *plus* human verification, and
  biomarker/outcome assertions need **domain-expert (oncology) sign-off** (risk-tier medium).
- **Annotations need IAA + expert validation.** Claude-drafted annotations cannot count as gold;
  the gold set is human-adjudicated and IAA is computed on humans.
- **No fabricated or "filled-in" citations/IDs.** Ungrounded assertions are rejected; Claude must
  never invent a PMCID, DOI, ontology ID, or quote. Re-location check is the backstop.
- **No clinical synthesis or patient-facing output**, ever, from the model — out of scope by guardrail.
- **Budget discipline:** any funded-lane extraction runs under `packages/runner` with a hard
  per-task escrow cap (per CLAUDE.md); prompt-cache the static schema/instructions to cut cost.

---

## 6. Ten concrete optimizations

1. **Fix the license model to two orthogonal axes** (derivatives-allowed × commercial-allowed),
   decided separately for verbatim full text vs the derived extraction; align ND/NC handling with
   NCBI's actual OA grouping; drop the global "CC-BY-4.0 for the whole corpus" claim in favor of a
   per-record computed output license.
2. **Add an explicit OA-Subset membership check** (is PMCID in the OA file list / OA Web Service?)
   as a hard gate distinct from license parsing, to exclude NIHMS author-manuscripts and non-OA hits
   by construction.
3. **Consume PubTator Central + Europe PMC SciLite as pre-annotators**, then verify, instead of
   building NER from zero — cuts cost and raises baseline F1; record their annotation provenance.
4. **Use OpenAlex (CC0) as the discovery/metadata backbone** for the Ewing query, retraction flags,
   and OA-status, reserving PMC OA strictly for redistributable full text.
5. **Replace "Cohen's κ ≥ 0.7" with task-appropriate IAA**: span-level F1 / Krippendorff's α for NER,
   a separate measure for assertions; define the agreement unit, double-annotated fraction, and
   adjudication protocol.
6. **Report per-entity-type P/R/F1 with per-type targets** (cell line/drug high; gene-symbol/fusion
   lower) instead of one blended 0.85.
7. **Make retraction/erratum and `cohortSize`/`evidenceType` first-class schema fields**; add an
   explicit small-n (n≈2–5) case-series policy to the aggregate-only rule.
8. **Specify Zenodo concept-DOI vs version-DOI + content-SemVer** and stable per-record IDs, so the
   living corpus is reproducibly citable.
9. **Estimate the OA-eligible denominator** (one PMC-OA count for the retrieval query) to justify the
   400 target as meaningful coverage, and report coverage % not just count.
10. **Tighten the dedup key hierarchy** (PMID > PMCID > normalized DOI > fuzzy title/author/year) with
    a documented canonical-copy and license-precedence rule when copies disagree.

---

## 7. Parallel & perpendicular spin-offs

- **`pmc-oa-cancer-corpus` (generalize the pipeline).** The license/PII gate + grounded-extraction +
  Croissant/Zenodo machinery is disease-agnostic. Promote it to a **reusable rare-disease corpus
  pipeline** (config = MeSH/synonyms + entity panels + cell-line list), then instantiate per cancer.
- **`ewsr1-fli1-knowledge-graph` (primary downstream).** The corpus is the licensed text substrate;
  co-design the assertion schema to its node/edge needs (gene–phenotype, gene–drug, model-finding) so
  the first reuse is frictionless.
- **`systematic-review-assist`.** The grounded, deduped, retraction-aware corpus + per-assertion
  provenance is the backbone of a PRISMA-style screening aid (search → screen → extract-with-citation),
  reusing the same retrieval and groundedness gates.
- **`biomarker-extraction` / `ewing-biomarker-evidence-cards`.** Expert-verified aggregate
  biomarker/outcome assertions become structured "evidence cards" (claim + span + reviewer +
  ontology IDs) — a high-value medium-risk extension.
- **`ewing-drug-target-evidence` / `ewing-research-landscape`.** Reuse the entity/assertion layer for
  target tractability maps and a landscape view; both consume, not duplicate, the corpus.
- **Rare-disease corpus pipeline as a community good.** Publish the gate + groundedness check as an
  MIT toolkit so NORD/rare-cancer foundations can spin up license-clean corpora for *their* disease.
- **MCP server for researchers.** A read-only **Model Context Protocol** server exposing the corpus
  (search, fetch record, fetch grounded assertions with spans/licenses) so any agent/IDE can query
  it with provenance attached — the natural "make it consumable" surface (MLCommons is already pairing
  Croissant with MCP: [Croissant+MCP](https://mlcommons.org/2025/10/croissant-mcp/)).

---

## 8. Open questions for the maintainer

1. **Is Europe PMC in scope for v1?** It has a larger OA full-text set, an annotations API, and a gold
   corpus that could seed ours — but adds cross-source dedup and per-article TDM-term capture.
2. **What is the binding ND/NC policy** once split into two axes — do we redistribute verbatim ND full
   text (arguably allowed) while withholding only the *derivative* extraction, or stay maximally
   conservative (metadata + capped quotes)? Needs the License+PII reviewer.
3. **One output license or per-record?** Confirm we abandon a single CC-BY-4.0 corpus license in
   favor of a per-record computed license (CC-BY-SA inheritance, NC preservation) — and what the
   *added* schema/annotation layer's license is when attached to NC/SA source content.
4. **Do we consume PubTator/SciLite as pre-annotators** (faster, cheaper, but inherits their
   provenance) **or annotate independently** (cleaner provenance, costlier)?
5. **What is the realistic OA-eligible Ewing denominator**, and does 400 represent strong coverage or
   a small fraction? (Run the count before committing the target.)
6. **IAA unit and measure** — entity span vs normalized ID vs assertion; κ vs span-F1/α; how many
   annotators and what double-annotated fraction?
7. **Versioning semantics** — Zenodo concept vs version DOI, content SemVer rules, per-record stable
   IDs across re-releases?
8. **Funded lane?** Will large extraction batches use `packages/runner` with a hard escrow cap, or
   stay donated-lane only? (Affects throughput vs the 400/100 targets.)
9. **Does the Zenodo fallback satisfy "delivered, not merged"?** If a real consumer isn't secured, is
   a self-published DOI an acceptable M0 outcome, or must the pilot block on the KG maintainer?
10. **Preprints (bioRxiv/medRxiv)** — confirmed out for v1, or in via Europe PMC where OA-licensed?
