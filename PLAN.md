# PLAN — ewing-literature-corpus

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated
>
> For the families who brought us here: Ewing sarcoma is a rare, aggressive bone and soft-tissue
> cancer that strikes mostly children, teenagers, and young adults. The science that will help them
> is scattered across thousands of papers that no one family — and few small labs — has the time to
> read, reconcile, and verify. This project does one careful, honest thing: it assembles the
> **openly-licensed** Ewing literature into a clean, fully-sourced, machine-readable corpus so that
> researchers can move faster. It makes **no** medical claims and gives **no** advice. Every
> assertion is traceable to a sentence in a real, openly-licensed paper. We would rather ship less
> and be right than ship more and mislead.

## Executive summary

Ewing sarcoma research is published across thousands of articles, but the knowledge is locked in
unstructured prose, inconsistent terminology, and paywalls. Researchers — especially the small labs
and trainees who do much of the rare-cancer work — spend disproportionate effort just *finding* and
*reconciling* what is already known about EWSR1-FLI1/ETS biology, cell-line models, candidate
targets, and aggregate clinical outcomes. There is no clean, openly-licensed, machine-readable,
fully-provenanced corpus of Ewing literature that downstream projects (e.g. a knowledge graph,
a reanalysis effort, or a systematic review) can build on without redoing the licensing and
extraction work from scratch.

This project produces exactly that: a **curated corpus of open-access Ewing sarcoma literature drawn
from the PubMed Central Open Access (PMC OA) Subset**, plus **grounded structured extraction** —
normalized entities (genes, fusion types, cell lines, drugs, disease concepts) and *aggregate*,
source-anchored research assertions — where **every extracted assertion links back to the exact
sentence (PMCID + section + character offsets + verbatim quote) it came from.** The corpus is
designed first as research infrastructure: its earliest, most concrete beneficiary is the sibling
Elyos project `ewsr1-fli1-knowledge-graph`, which needs a licensed, provenanced text substrate.

The deliverable is **a curated, licensed, provenanced corpus + extraction layer — not medical
advice, not patient-facing guidance, and not a republication of closed-access papers.** Two hard
constraints define the work and lead the compliance section below: (1) **only open-access / aggregate
/ de-identified data** — controlled-access genomic data (dbGaP, EGA, individual-level biobanks) and
**any identifiable patient data** are out of scope; and (2) **per-article license verification** with
conservative handling of NoDerivatives (ND) and NonCommercial (NC) licenses that exist *within* the
PMC OA Subset. Because Ewing primarily affects children and young adults, **case reports describing
individual patients are treated as a privacy hazard**: we extract aggregate research findings, never
identifiable individual-level detail.

Risk tier is **low** for the corpus-building and extraction *infrastructure*, escalating to
**medium** for any extraction of biomarker/clinical-outcome assertions (domain-expert verification
required). Anything **patient-facing** is **out of scope here** and would be a separate `high`-risk
project gated behind oncologist + patient-advocate review. The plan front-loads the license/PII gate
and a human + domain-expert verification protocol because the dominant risks here are not technical —
they are legal (license misclassification), privacy (re-identifying a child in a case report), and
factual (an extraction that misstates the science).

## Problem & beneficiaries

**Who is helped.**
- **Ewing sarcoma researchers and trainees**, especially in small or under-resourced labs, who need
  to find, read, and reconcile the existing literature quickly and reliably.
- **Downstream Elyos cancer projects** that need a licensed, provenanced text substrate —
  most directly `ewsr1-fli1-knowledge-graph` (knowledge graph of fusion biology),
  `ewing-drug-target-evidence`, `ewing-biomarker-evidence-cards`, and `ewing-research-landscape`.
- **Patient-advocacy and rare-cancer foundations** that fund and synthesize research and need a
  trustworthy, sourced map of the open literature (as an internal research aid, *not* patient-facing
  output).
- **Indirectly, families and patients** — but only via researchers and clinicians who use the corpus
  to do their work faster. This project never speaks to patients directly.

**The verified need.** The *general* need is well established: rare-cancer research is bottlenecked
by fragmented literature and the absence of clean, reusable, openly-licensed corpora; systematic
reviews and knowledge bases routinely re-do ingestion and licensing from zero. We treat that general
need as real. The **specific, per-consumer need is TO BE SECURED**: we have not yet confirmed a named
research group, advocacy organization, or maintainer who has committed to *consume* this corpus.
The strongest candidate consumer is the **internal sibling project `ewsr1-fli1-knowledge-graph`**,
whose requirements should drive the extraction schema; even so, until a named consumer (internal or
external) confirms they will use the output, individual tasks carry `verifiedNeed: false`. "Delivered,
not merged" requires the corpus to actually be *used by a beneficiary*, not merely produced.

**Partner / requestor.** TO BE SECURED. Candidate channels: the maintainer of
`ewsr1-fli1-knowledge-graph` (internal, highest-probability first consumer); Ewing/sarcoma research
labs; pediatric-oncology cooperative groups; rare-cancer foundations and patient-advocacy
organizations. M0 includes explicit consumer-outreach work; no partner is assumed.

## Goals and non-goals

**Goals**
- Define and publish a **corpus record schema** and an **extraction (assertion) schema** in which
  every assertion is anchored to a verbatim source span (PMCID + section + character offsets), with
  ontology-normalized entities and a human/expert-verification status.
- Build a reproducible **PMC OA ingestion pipeline** that retrieves Ewing-relevant open-access
  articles, captures each article's **machine-readable license**, and records full provenance.
- Make **per-article license verification and PII/case-report triage a non-skippable, auditable
  gate**, with conservative routing of ND/NC licenses.
- Produce **grounded structured extraction** (genes, fusion types, cell lines, drugs, disease
  concepts, and aggregate research assertions) normalized to public ontologies, evaluated against a
  human-annotated gold standard.
- Release the corpus + extraction as a **versioned, openly-licensed, citable artifact** (with a
  Datasheet and a Croissant/BioC-compatible export) and demonstrate **real downstream reuse**.

**Non-goals**
- We do **not** produce medical advice, diagnosis, prognosis, or treatment guidance, and we do
  **not** create patient-facing content. (Patient-facing Ewing education is a separate `high`-risk
  project gated behind oncologist + patient-advocate review.)
- We do **not** include or redistribute closed-access / paywalled full text, nor full text under
  NoDerivatives terms (metadata + grounding-quote snippets only for ND — see compliance).
- We do **not** ingest controlled-access data (dbGaP, EGA, individual-level biobanks) or **any**
  identifiable patient data, and we do **not** extract or republish identifiable individual-level
  case details.
- We do **not** generate new scientific conclusions, rankings, meta-analyses, or "the evidence
  says X" syntheses; we extract what specific papers state, with provenance. Synthesis is downstream.
- We do **not** invent, infer, or "fill in" assertions: an assertion with no verbatim source span is
  rejected (no ungrounded extraction).
- We do **not** auto-publish; a human reviews and releases.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("papers ingested", "assertions produced")
are explicitly excluded unless tied to verified quality and reuse.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| Ewing OA articles curated with **verified license + complete provenance** (corpus-eligible, gate-passed) | 0 | ≥ 400 |
| **License/PII gate correctness**: license classification + PII/case-report disposition correct on an audited sample | n/a | 100% (zero ND-text redistributions; zero identifiable patient details) measured by audit |
| **Extraction groundedness**: sampled assertions whose verbatim source span verifiably supports the claim | n/a | ≥ 98% (target 100% for the gold-checked set) |
| **Extraction quality vs. gold standard** (entity normalization F1 on the annotated eval set) | n/a | ≥ 0.85 F1; inter-annotator agreement (Cohen's κ) ≥ 0.7 on the gold set |
| **Downstream reuse** (a named project/researcher uses the corpus or extraction) | 0 | ≥ 1 verifiable reuse event (target: `ewsr1-fli1-knowledge-graph` consuming it) |
| Confirmed consumers/partners (named, committed to use the output) | 0 | ≥ 1 secured |
| Domain-expert-verified aggregate assertions (biomarker/outcome statements signed off by reviewer) | 0 | ≥ 100, each with provenance + reviewer attribution |

Notes on outcome attribution:
- A **reuse event** must be externally verifiable (a downstream repo/commit that ingests the corpus,
  a citation, or a maintainer's written confirmation of use). Self-reported reuse does not count.
- "**Groundedness**" is measured by sampling extracted assertions and checking that the cited span
  (PMCID + offsets + quote) actually supports the assertion; a single fabricated/unsupported
  assertion in the sample is a defect, not noise.
- **Corpus size is deliberately a quality-gated count**, not a raw count: only gate-passed,
  fully-provenanced articles count toward the 400 target. We would rather report 400 clean than
  3,000 unverified.

## Scope

**In scope**
- A curated corpus of **PMC OA Subset** Ewing sarcoma articles: bibliographic metadata, full text
  *where the license permits redistribution of derivatives*, section structure (IMRaD), and complete
  per-article provenance + machine-readable license.
- A **corpus record schema** and an **extraction (assertion) schema** (BioC-compatible where
  practical) with span-level provenance.
- **Grounded structured extraction**: entities (genes/proteins, EWSR1-ETS fusion types, cell lines,
  drugs/compounds, disease concepts, model organisms, assay/method) normalized to public ontologies;
  and **aggregate, source-anchored research assertions** (e.g. "in cell line X, knockdown of Y
  reduced Z" as stated by paper P) — never patient-level data.
- **License + PII/case-report triage** per article, recorded as a committed gate artifact.
- A human-annotated **gold-standard evaluation set** and an extraction **evaluation harness**.
- Reproducible ingestion/extraction tooling (small TypeScript packages) and a versioned, citable
  release with a Datasheet.

**Out of scope**
- Patient-facing content, medical advice, diagnosis, prognosis, treatment guidance, "what the
  evidence means for you" — all out of scope (separate `high`-risk project if ever pursued).
- Closed-access / paywalled full text; full-text redistribution of ND-licensed articles; any source
  outside the PMC OA Subset's redistribution terms (bibliographic metadata only for those).
- Controlled-access data (dbGaP, EGA, individual-level biobanks); **any** identifiable patient data;
  extraction or republication of identifiable individual-level case-report detail.
- New scientific synthesis, meta-analysis, conclusions, rankings, or evidence grading.
- Ungrounded / inferred assertions (no source span ⇒ rejected).
- Hosting of, or derivative redistribution from, **non-commercial/custom-licensed databases**
  (e.g. **COSMIC**, **OncoKB**) — these are *flagged and excluded* from redistribution; only an
  optional non-redistributed cross-reference by stable ID is permitted where a paper cites them.
- Automated, unattended publishing.

## Solution approach & architecture

This is a **content/data project with light supporting software** (ingestion + license/PII gate +
extraction + normalization + evaluation tooling). It is not a runtime service.

**Pipeline (per article).**
1. **Retrieve (OA only).** Query PMC for Ewing-relevant articles; for each candidate, fetch the
   record **only from the PMC OA Subset** via the OA Web Service / BioC API / OA bulk packages.
   Non-OA hits are reduced to bibliographic metadata only (no full text).
2. **License capture & classification.** Read the article's machine-readable license (PMC OA license
   field / JATS `<license>` / CC URL); classify into a fixed policy bucket (see compliance). Record
   the license id, URL, and a snapshot (committed copy + SHA-256 + Wayback URL).
3. **PII / case-report triage.** Detect case-report / individual-patient content; route to the
   appropriate handling (aggregate-only extraction; no identifiable detail). Record the disposition.
4. **Normalize & structure.** Parse to a canonical corpus record: metadata, sections (IMRaD),
   tokenized spans, and BioC-compatible offsets. Full text is included **only** for redistribution-
   permitted licenses; ND/restricted articles keep metadata + short grounding quotes only.
5. **Extract (grounded).** Run entity recognition + assertion extraction; **every output must carry a
   verbatim source span** (PMCID + section + char offsets + quote). Ungrounded outputs are dropped.
6. **Ontology-normalize.** Map entities to public ontologies (below); record the ontology id + version
   and a normalization confidence; unmapped entities are kept as surface forms flagged `unmapped`.
7. **Verify.** Human reviewer checks groundedness + license/PII; domain (oncology/sarcoma-biology)
   reviewer checks scientific accuracy for biomarker/outcome assertions. Verification status is
   recorded per assertion.
8. **Release.** Versioned corpus + extraction with Datasheet and Croissant/BioC export; a human
   performs the release.

**Corpus record schema (source of truth).** One canonical record per article:
`pmcid`, `pmid`, `doi`, `title`, `journal`, `pubDate`, `authors[]` (names only as published; no
contact PII retained), `license {id, spdx, url, redistributable:boolean, derivativesAllowed:boolean,
commercialUseAllowed:boolean, snapshotRef}`, `oaStatus`, `provenance {source:"PMC-OA", retrievedAt,
oaPackageVersion, attribution}`, `caseReport {isCaseReport:boolean, individualDetailPresent:boolean,
disposition}`, `sections[] {type, charStart, charEnd, text?}` (text omitted when not redistributable),
`fullTextIncluded:boolean`, `specVersions {bioc, croissant}`.

**Extraction (assertion) schema.** One record per assertion:
`id`, `pmcid`, `assertionText` (normalized statement), `source {section, charStart, charEnd, quote}`
(**required** — no span ⇒ invalid), `entities[] {surfaceForm, type, ontology, ontologyId,
ontologyVersion, normConfidence}`, `assertionType` (e.g. `entity-mention | gene-phenotype |
gene-drug | aggregate-outcome | model-finding`), `extractor {method, modelOrTool, version}`,
`extractionConfidence`, `verification {humanVerified:boolean, domainVerified:boolean, reviewer,
reviewedAt}`, `aggregateOnly:boolean` (must be `true`; individual-patient assertions are rejected).

**Ontologies / normalization targets (pinned, recorded in `specVersions`).**
- Genes/proteins → **HGNC** (+ optional UniProt for proteins).
- Disease concepts → **MONDO** / **NCI Thesaurus (NCIt)** (Ewing sarcoma = MONDO:0012817; record id).
- Cell lines → **Cellosaurus** (e.g. A673 = CVCL_0080; SK-N-MC, TC-71, etc.).
- Drugs/compounds → **ChEBI** (+ optional RxNorm/DrugBank-open identifiers, no closed data).
- Fusion types → a small **controlled vocabulary** maintained in-repo (EWSR1-FLI1 type 1/2,
  EWSR1-ERG, EWSR1-ETV1/ETV4/FEV, FUS-ERG, etc.) mapped to gene HGNC ids.
- Variants (where stated) → **HGVS** notation as published; no controlled-access variant data.

**Extraction approach (grounded, LLM-assisted, human-verified).** Extraction is LLM-assisted with a
**hard groundedness constraint**: the extractor must return the exact supporting span and quote, and
a deterministic post-check re-locates the quote in the source text (offset + string match) before the
assertion is accepted — an assertion whose quote cannot be re-found in the source is automatically
rejected. This makes hallucinated assertions structurally impossible to retain. The LLM is used as an
extraction/normalization aid, never as an authority; the **source paper is the only authority** and a
human verifies. (Per CLAUDE.md, the donated lane prepares workspaces and humans run their agent; the
funded lane, if ever used, runs under `packages/runner` with a hard budget cap.)

**Tech stack.** TypeScript, ESM, pnpm workspaces (Elyos conventions). Ingestion/normalization/eval
are small Node packages with minimal dependencies; no runtime service. Corpus + extraction stored as
**JSONL** (+ BioC JSON for interop, Croissant for dataset metadata). Everything runs locally or in CI.

**Key decisions.**
- **Canonical-record-first**: BioC/Croissant exports are *projections* of the canonical record, so we
  never hand-maintain parallel formats.
- **Groundedness is enforced in code** (quote re-location check), not just by reviewer goodwill.
- **License gate is blocking and per-article**, encoded as a committed artifact; ND/closed never get
  full text redistributed.
- **PMC OA Subset is the only full-text source** (it exists precisely to enable text-mining reuse);
  scope is widened beyond it only by a deliberate, separately-reviewed decision.

## Data, licensing & compliance

**THIS IS THE CRITICAL SECTION. The cancer-domain guardrails below are binding and lead everything
else.**

### Cancer-domain guardrails (binding)
1. **Open-access / aggregate / de-identified data ONLY.** Full text comes exclusively from the
   **PMC Open Access Subset**. **Controlled-access genomic/clinical data — dbGaP, EGA, individual-level
   biobanks — are OUT OF SCOPE** (they require authorized access + IRB, which donated AI tasks cannot
   and must not satisfy). **Any identifiable patient data is OUT OF SCOPE.**
2. **No identifiable individual-patient data, ever — including from open-access case reports.**
   Because Ewing sarcoma predominantly affects children and young adults, case reports are a real
   re-identification hazard (rare disease + age + sex + geography + imaging dates can identify a
   child). We extract **only aggregate / research-level findings**; we do **not** extract or
   republish identifiable individual-level detail (demographics tied to an individual, dates, free-
   text clinical narrative, identifiable images). `aggregateOnly` must be `true` for every assertion.
3. **No medical advice. Education-only is out of scope here.** This project produces **research
   infrastructure**, not patient-facing content. Any patient-facing derivative would be a separate
   project, **education only**, **sourced**, explicitly **"not medical advice"**, and **gated behind
   oncologist + patient-advocate review (`riskTier: high`)**. We do not produce it here.
4. **Provenance on every assertion.** Every extracted assertion links to a verbatim source span
   (PMCID + section + char offsets + quote). No span ⇒ the assertion is invalid and rejected.
5. **Per-source license verification, conservatively.** TCGA/GDC **open-tier** and **GEO** are open
   (relevant if a paper's data is referenced — we still don't ingest controlled tiers). **COSMIC and
   OncoKB are non-commercial / custom-licensed → flagged and excluded from redistribution**; we may
   record a non-redistributed cross-reference by stable ID only.

### PMC OA Subset license policy (per-article, fixed before triage)
The PMC OA Subset contains articles under a **range** of licenses; being in the OA Subset does **not**
mean "do anything." We classify each article's machine-readable license into fixed buckets and route
accordingly:

| License bucket | Examples | Full text in corpus? | Derivative extraction redistributed? | Notes |
| --- | --- | --- | --- | --- |
| Public domain / CC0 | CC0, US-gov works | Yes | Yes | Record provenance even when attribution not required |
| Attribution | CC-BY (4.0/3.0) | Yes | Yes | Attribution string required |
| Attribution-ShareAlike | CC-BY-SA | Yes | Yes (share-alike) | Our derivative output inherits SA for that content; recorded |
| NonCommercial (incl. NC-SA) | CC-BY-NC, CC-BY-NC-SA | Yes | Yes (non-commercial) | Our commons output is non-commercial; NC recorded; SA inherited |
| **NoDerivatives** | CC-BY-ND, CC-BY-NC-ND | **No (metadata + short grounding quotes only)** | **No** | Facts aren't copyrightable; we keep bibliographic metadata + minimal verbatim quote-spans for provenance only, never the full text or a text derivative |
| Unclear / all-rights-reserved / non-OA | — | No (metadata only) | No | Excluded from full text; biblio metadata only |

**Objective acceptance criterion.** Full text is included **only** if the article is in the PMC OA
Subset **and** `license.derivativesAllowed: true` is recorded from a cited license id/URL. Missing,
unparseable, or ND licenses ⇒ metadata-only (and grounding quotes within strict length limits for ND).
No default-allow. The whole-corpus output license is **CC-BY-4.0 for our added content** (schemas,
extraction layer, annotations, code is **MIT**); per-article redistributed text retains its **original
license**, recorded per record (mixed-license corpus, faithfully labeled — never relicensed).

**Grounding-quote limit for restricted articles.** For ND / metadata-only articles, provenance is
preserved by storing **only the minimal verbatim quote needed to evidence an assertion** (hard cap,
e.g. ≤ 25 words per quote, ≤ 200 words total per article), plus the offset and a link to the source —
never the section text or full text. This is the standard "facts + minimal citation" stance, applied
conservatively.

### Provenance model
Every record stores: source = PMC OA, PMCID/PMID/DOI, retrieval timestamp, OA package version,
license id + SPDX (where mappable) + license URL + license snapshot (committed copy + SHA-256 +
Wayback URL), and the required attribution string. Every assertion additionally stores its source
span + verbatim quote + extractor + verification status. Provenance is part of the committed
deliverable, not an afterthought.

### Privacy / PII stance
- **Mandatory PII / case-report triage before extraction.** Detect case-report and individual-patient
  content; never extract identifiable detail. We never attempt to re-identify anyone, and we treat
  rare-disease case material as high-sensitivity by default.
- **PII detection methodology** (repeatable, recorded in the gate artifact): article-type heuristics
  (JATS `article-type="case-report"`, "case report" / "a N-year-old" patterns); direct-identifier
  scan (names of non-authors, contact details, MRNs, exact dates of birth/admission, identifiable
  image references); rare-disease quasi-identifier flag (age + sex + precise geography + diagnosis
  date). Any flag ⇒ aggregate-only handling or exclusion; the methodology output is recorded.
- **No identifiable images.** We do not ingest, store, or describe identifiable patient images.
- We store author **names as published** for citation/attribution only — no author contact PII,
  affiliations beyond what's needed for citation, or re-contact data.

### Attribution
All redistributed content attributes the original authors/journal per its license and links to the
source. Our added schemas/extraction/annotations are **CC-BY-4.0**; tooling is **MIT**; per-article
text keeps its original license, labeled per record.

## Quality, review & risk gates

**Risk tier.** **low** for the corpus + extraction *infrastructure* (ingestion, schema, entity
normalization). **medium** for extraction of **biomarker / clinical-outcome assertions**, which
require domain-expert (oncology / sarcoma-biology) verification for accuracy and framing.
**high** (and **out of scope** for this project) for anything **patient-facing** — which, if ever
pursued, requires **oncologist + patient-advocate sign-off** and "not medical advice" framing.

**Required review before a deed is "done":**
- **License + PII reviewer** (mandatory, every article/batch): confirms the recorded license bucket
  and redistribution decision, and that no identifiable patient data / ND full text is present. Hard,
  non-skippable gate.
- **Technical reviewer**: confirms schema validity, that the groundedness re-location check passes for
  100% of accepted assertions, ontology-normalization correctness, and CI green.
- **Domain (oncology / sarcoma-biology) reviewer**: required for any biomarker/outcome/model-finding
  assertion (`riskTier: medium`); confirms the extraction faithfully represents what the paper states
  and does not drift into clinical claims or advice. Sign-off recorded per assertion/batch.

**Groundedness as an automated gate.** CI runs the deterministic quote re-location check on every
extraction artifact; any assertion whose `source.quote` cannot be re-found at the recorded offsets in
the (redistribution-permitted) source fails the build. "CI green" therefore means groundedness is
machine-verified, not asserted.

**Gold-standard evaluation.** A human-annotated gold set (≥ 2 annotators, κ recorded) measures
entity-normalization precision/recall/F1 and assertion groundedness; the eval harness reports these
per release. No extraction release ships without an eval run against the current gold set.

**Definition of Shipped.** The corpus + extraction is **shipped** when: (1) every included article
passed the license + PII gate with a committed artifact; (2) every accepted assertion passes the
automated groundedness check and carries required-tier review sign-off (domain reviewer for
medium-risk assertions); (3) an eval run against the gold set meets the quality targets; (4) the
versioned, openly-licensed artifact is **released and actually consumed by a named beneficiary**
(downstream project/researcher) with a recorded reuse-evidence artifact. Producing files is not
shipped; verified, licensed, *used* output is.

## Roadmap & milestones

**M0 — Foundation & cold-start (thin)**
- Goal: lock the schemas + license/PII gate, build a minimal OA ingestion path, and prove the whole
  flow on a small pilot set; begin consumer outreach (esp. the sibling KG project).
- **Cold-start de-risking.** The pilot is gated on a realistic *consumption* path before extraction
  begins, in priority order: (a) the **internal `ewsr1-fli1-knowledge-graph` maintainer** agreeing to
  consume the pilot extraction; failing that, (b) a **self-serve fallback** — publish the pilot corpus
  + extraction as a **Zenodo record/DOI** we control, so a real *released* (and self-citable) outcome
  exists even before an external consumer is secured.
- Exit criteria: (1) corpus record schema + extraction (assertion) schema published; (2) license +
  PII/case-report gate checklist published and applied to the pilot; (3) OA ingestion tool retrieves
  pilot articles and captures license + provenance, CI green; (4) **pilot of ~25 OA articles** fully
  curated end-to-end with **one extraction type** (entity mentions), every assertion grounded
  (automated check green) and human-verified, released via the KG consumer or Zenodo fallback with a
  recorded outcome artifact — or, if neither materializes, **submitted** with the blocker surfaced;
  (5) ≥ 1 consumer-outreach thread opened.

**M1 — Corpus build + license/PII gate hardened**
- Goal: scale the corpus with rigorous, partly-automated license classification + PII triage, dedup,
  and complete provenance.
- Exit criteria: (1) automated license classifier + ND/NC routing in place, validated against a
  hand-labeled license sample; (2) PII/case-report audit pass over the corpus with committed
  dispositions; (3) ≥ 150 OA articles curated (gate-passed, provenanced); (4) dedup/versioning by
  PMCID/DOI working; (5) ≥ 1 confirmed consumer/partner.

**M2 — Structured extraction at scale + evaluation**
- Goal: grounded entity + assertion extraction normalized to ontologies, with a gold standard and
  measured quality.
- Exit criteria: (1) entity extraction + ontology normalization (HGNC/MONDO/Cellosaurus/ChEBI)
  running with the groundedness check enforced in CI; (2) gold-standard eval set built (κ ≥ 0.7) and
  eval harness reporting F1 ≥ 0.85 entity normalization; (3) ≥ 400 articles curated cumulatively;
  (4) ≥ 100 domain-expert-verified aggregate biomarker/outcome assertions, each provenanced.

**M3 — Release, reuse & sustainability**
- Goal: a citable release, demonstrated downstream reuse, and a refresh model.
- Exit criteria: (1) versioned corpus + extraction released with a Datasheet + Croissant/BioC export
  (Zenodo DOI); (2) ≥ 1 verifiable external/downstream reuse event (target: KG project consuming it);
  (3) documented refresh/staleness process for new PMC OA articles; (4) maintainer + steward
  identified for ongoing upkeep.

Dependencies: M1 depends on M0 schemas + ingestion; M2 extraction depends on M1's curated corpus;
M3 release/reuse depends on a quality-passing extraction body from M2.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above, each with
a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`), acceptance
criteria for the most important tasks, and a milestone Definition of Done. A sized-but-unscheduled
backlog and one complete, schema-valid example Task JSON are included there. Per Elyos guardrails,
every per-article task still requires its own committed license + PII gate artifact before work
proceeds; listing a task does not pre-approve any article.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns schemas, ingestion, extraction tooling, and the backlog.
- **License + PII reviewer:** TBD (TO BE SECURED) — mandatory, **non-skippable** gatekeeper for
  license classification and PII/case-report disposition; no article ships without this sign-off. Must
  be filled **before the M0 pilot is reviewed** (blocking prerequisite, not a parallel hire). May
  rotate among ≥ 2 qualified reviewers, but at least one named reviewer must always exist or
  ingestion/extraction halts. Until named, all tasks remain `verifiedNeed: false`.
- **Domain (oncology / sarcoma-biology) reviewer:** TBD (TO BE SECURED) — required for medium-risk
  biomarker/outcome assertions; confirms scientific faithfulness and that no clinical advice creeps
  in. A credentialed oncologist/sarcoma researcher; sign-off recorded per assertion batch.
- **Technical reviewer(s):** rotation of contributors who verify schema validity, the groundedness
  check, normalization, and CI.
- **Steward (last-mile owner):** TBD — owns the relationship with the consuming project/partner and
  records the reuse/acceptance evidence (the "delivered" signal).
- **Partner / requestor / first consumer:** TO BE SECURED — candidate: `ewsr1-fli1-knowledge-graph`
  maintainer (internal); else a sarcoma lab / advocacy org / rare-cancer foundation.

## Dependencies & integrations

- **Data sources:** **PubMed Central Open Access Subset** (OA Web Service, BioC API / `pmc-oa`
  packages, Entrez E-utilities for discovery/metadata). PubMed for bibliographic metadata of non-OA
  hits (metadata only).
- **Ontologies / vocabularies (pinned, versioned in `specVersions`):** HGNC, MONDO, NCI Thesaurus,
  Cellosaurus, ChEBI; HGVS for variant notation; SPDX for license ids. Optional: UniProt, RxNorm
  (open identifiers only).
- **Standards/formats:** **BioC** (text-mining interchange), **Croissant ML** (dataset metadata),
  JATS (source article structure), schema.org/Dataset.
- **Flagged / excluded databases:** **COSMIC, OncoKB** (non-commercial/custom — cross-reference by
  ID only, no redistribution). **dbGaP, EGA, individual-level biobanks** (controlled-access — out of
  scope entirely).
- **Elyos pieces:** Task JSON schema (`packages/schema`); donated-lane CLI workspace/PR flow
  (`packages/cli`); good-deed definition + refusal guardrails; sibling project
  `ewsr1-fli1-knowledge-graph` (primary downstream consumer). No funded-lane/runner dependency unless
  a budgeted extraction batch is explicitly chosen (then `packages/runner` + hard `fundedBudgetUsd` cap).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Redistributing ND-licensed or closed full text (license violation) | Medium | High | Per-article license gate; full text only when `derivativesAllowed:true`; ND ⇒ metadata + capped grounding quotes; license snapshot recorded | License+PII reviewer |
| Extracting/republishing identifiable patient data from a case report | Medium | High | Mandatory PII/case-report triage before extraction; aggregate-only enforced (`aggregateOnly:true`); rare-disease quasi-identifier flag; exclude on any signal; never re-identify | License+PII reviewer |
| Hallucinated / ungrounded extracted assertions | Medium | High | Hard groundedness constraint + deterministic quote re-location check in CI; no span ⇒ rejected; human + domain verification | Technical + Domain reviewers |
| Misstating the science / drifting into clinical claims or advice | Medium | High | Domain-expert sign-off for biomarker/outcome assertions; "no synthesis, no advice" non-goal; extraction states only what the paper says, with quote | Domain reviewer |
| License misclassification by the automated classifier | Medium | High | Classifier validated against hand-labeled sample; human reviewer confirms; default to metadata-only on uncertainty | License+PII reviewer |
| Mixing controlled-access data into scope | Low | High | dbGaP/EGA/biobank explicitly out of scope; ingestion restricted to PMC OA; reviewers reject any controlled-access content | Maintainer |
| No consumer secured ⇒ corpus built but unused (fails "delivered") | Medium | High | M0 outreach to the sibling KG project; Zenodo self-publish fallback; `verifiedNeed:false` until secured; steward role | Steward |
| Cross-referencing COSMIC/OncoKB content beyond a bare ID (license breach) | Medium | Medium | Flag both as non-commercial; ID-only cross-reference, never redistributing their data | License+PII reviewer |
| Corpus staleness as new OA articles appear / are retracted | Medium | Medium | Versioned releases; refresh process; retraction check against PubMed; staleness tracked as maintenance | Maintainer |
| Ontology/spec drift (HGNC/MONDO/Cellosaurus/BioC version changes) | Medium | Low | Pin versions in `specVersions`; bump only via a deliberate task; canonical-record-first | Maintainer |
| Scope creep into synthesis/meta-analysis or patient-facing output | Medium | Medium | Explicit non-goals; reviewers reject; patient-facing is a separate high-risk project | Maintainer |

## Security & privacy

- **Small threat surface** (no runtime service, no data hosting beyond the released corpus). Main
  surfaces: CI, the released corpus files, and the upstream-content/PII risk.
- **Secrets handling:** ingestion uses public PMC/Entrez endpoints; an NCBI API key (rate limits) is
  optional and, if used, is supplied via env var by the human — never written to logs, receipts, or
  committed files (per Elyos rules). No credentials in the repo.
- **PII:** the dominant concern is *upstream* identifiable patient content in case reports. Handled by
  the mandatory triage + aggregate-only enforcement; we never download/store identifiable images,
  never re-identify, and exclude on any signal.
- **Abuse/misuse prevention:** refuse and flag any task that would steer the corpus toward
  re-identification, surveillance, clinical advice, laundering closed content as open, or primarily
  serving a for-profit. Extraction stays descriptive, sourced, and aggregate.

## Sustainability & maintenance

- **Post-delivery ownership:** the maintainer keeps ingestion/extraction tooling current with PMC OA,
  ontology, and BioC/Croissant spec changes; the steward maintains the consumer relationship and
  records reuse.
- **Refresh:** a documented process re-queries PMC OA for new/updated Ewing articles and checks
  PubMed for **retractions** (retracted articles are flagged/removed from the corpus); releases are
  versioned with a changelog.
- **Outcome tracking:** the steward records reuse/consumption events and expert-verification counts
  against the success metrics, reviewed each milestone.

## Open questions

- Who is the first **confirmed consumer** — the internal `ewsr1-fli1-knowledge-graph` maintainer, an
  external sarcoma lab, or an advocacy org? (Drives the extraction schema's priorities.)
- For NoDerivatives (ND) articles, what is the exact, defensible **grounding-quote length cap** (we
  propose ≤ 25 words/quote, ≤ 200 words/article) — confirm with the license reviewer before M1.
- Where is the corpus released and versioned — **Zenodo DOI** (proposed default), a GitHub release,
  or a consumer-specified location?
- What is the precise **Ewing retrieval query** (MeSH + EWSR1/FLI1/ETS terms + fusion synonyms) and
  how do we measure its recall/precision against a hand-curated seed set?
- Do we ever use the **funded lane** for large extraction batches (with a hard budget cap), or keep
  everything donated-lane?
- How do we treat **preprints** (bioRxiv/medRxiv) — out of scope for v1 (not PMC OA), revisit later?

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\elyos\planning\ROADMAP.md`
- Sibling project — `ewsr1-fli1-knowledge-graph` (primary downstream consumer)
- PubMed Central Open Access Subset (license terms; OA Web Service; BioC API)
- BioC text-mining interchange format; Croissant ML metadata specification; JATS
- Ontologies: HGNC, MONDO, NCI Thesaurus, Cellosaurus, ChEBI; HGVS; SPDX license list
- Creative Commons CC0 / CC-BY / CC-BY-SA / CC-BY-NC / CC-BY-ND families
- Flagged non-commercial DBs: COSMIC, OncoKB (cross-reference by ID only)

---

## Appendix A — Improvements applied

The 25 specific improvements below were identified against the first draft and **have been applied**
in the body above (each notes where).

1. **Led the compliance section with the binding cancer guardrails** as a numbered block (open-access
   only; no identifiable patient data; no advice; provenance on every assertion; per-source license
   verification) — applied in *Data, licensing & compliance*.
2. **Made case reports a named privacy hazard** because Ewing is a pediatric/AYA cancer (rare disease
   + age + geography ⇒ re-identification) — applied in guardrail #2 and the PII methodology.
3. **Concrete PMC OA license-bucket table** with explicit ND handling (metadata + capped quotes, no
   full text) — applied as the license policy table.
4. **Enforced groundedness in code** via a deterministic quote re-location check that auto-rejects any
   assertion whose quote can't be re-found — applied in architecture, quality gates, and risks.
5. **Quantified success metrics with baselines/targets** and made corpus size a *quality-gated* count
   (only gate-passed articles count) — applied in *Success metrics*.
6. **Named the sibling `ewsr1-fli1-knowledge-graph` as the first, internal candidate consumer**, giving
   a realistic cold-start path and a Zenodo self-publish fallback — applied in beneficiaries + M0.
7. **Pinned ontologies/specs with versions in `specVersions`** (HGNC/MONDO/NCIt/Cellosaurus/ChEBI/
   BioC/Croissant) and a version-bump-only-via-task rule — applied in architecture + dependencies.
8. **Concrete identifiers** (Ewing = MONDO:0012817; A673 = CVCL_0080) so normalization is checkable —
   applied in the ontology list.
9. **Added a controlled fusion-type vocabulary** (EWSR1-FLI1 type 1/2, EWSR1-ERG, ETV1/ETV4/FEV,
   FUS-ERG) mapped to HGNC — applied in normalization targets.
10. **COSMIC/OncoKB flagged non-commercial, ID-only cross-reference, no redistribution** — applied in
    guardrails #5, scope, dependencies, and risks.
11. **Mixed-license corpus stated honestly**: our added content CC-BY-4.0 / code MIT, per-article text
    keeps its original license, never relicensed — applied in compliance/attribution.
12. **Three-tier risk model made explicit**: infra = low, biomarker/outcome assertions = medium
    (domain reviewer), patient-facing = high & out of scope — applied in *Quality, review & risk gates*.
13. **Added a domain (oncology/sarcoma-biology) reviewer role** distinct from the license/PII reviewer
    — applied in governance + review gates.
14. **`aggregateOnly:true` is a schema-enforced field** (individual-patient assertions rejected) —
    applied in the extraction schema and guardrails.
15. **Required `source` span (offsets + quote) as a mandatory schema field** — no span ⇒ invalid —
    applied in the extraction schema.
16. **Defined "reuse event" as externally verifiable** (downstream commit/citation/written
    confirmation; no self-report) — applied in *Success metrics*.
17. **Added a gold-standard eval set with inter-annotator agreement (κ ≥ 0.7)** and an eval harness
    reporting F1 — applied in metrics, M2, and tasks.
18. **License snapshot format fixed** (committed copy + SHA-256 + Wayback URL), matching Elyos
    provenance norms — applied in provenance model.
19. **Added retraction handling** in the refresh process (check PubMed; flag/remove) — applied in
    *Sustainability*.
20. **ND grounding-quote cap proposed concretely** (≤ 25 words/quote, ≤ 200 words/article) and flagged
    as an open question for reviewer confirmation — applied in compliance + open questions.
21. **Restricted full-text source strictly to PMC OA Subset**; non-OA = metadata only; preprints
    out of scope for v1 — applied in scope + open questions.
22. **Compassionate, honest framing for families** in the header without over-promising (no advice,
    research infrastructure only) — applied in the header/executive summary.
23. **Explicit refusal/abuse stance** (no re-identification, no advice, no laundering closed content,
    no for-profit primary benefit) — applied in *Security & privacy*.
24. **Author PII minimization** (names-as-published for citation only; no contact/re-contact data) —
    applied in the corpus schema + PII stance.
25. **Funded-lane contingency** (large extraction batches may use `packages/runner` with a hard
    `fundedBudgetUsd` cap) made explicit, consistent with CLAUDE.md — applied in architecture +
    dependencies + open questions.

---

## Review sign-off

**Reviewed for completeness against the 17-section spec.** All required H2 sections are present and in
order: Executive summary; Problem & beneficiaries; Goals and non-goals; Success metrics (outcomes);
Scope; Solution approach & architecture; Data, licensing & compliance; Quality, review & risk gates;
Roadmap & milestones; Work breakdown; Governance, roles & stakeholders; Dependencies & integrations;
Risks & mitigations; Security & privacy; Sustainability & maintenance; Open questions; References.

**Reviewed for correctness against the cancer guardrails (binding):**
- Open-access / aggregate / de-identified ONLY — enforced; PMC OA Subset is the sole full-text source;
  dbGaP/EGA/individual biobanks and all identifiable patient data are out of scope. ✔
- Per-source license verification — PMC OA per-article bucket policy with ND/NC routing; COSMIC/OncoKB
  flagged non-commercial (ID-only); TCGA/GDC open-tier & GEO noted as open. ✔
- No medical advice; patient-facing = education-only, sourced, "not medical advice", oncologist +
  patient-advocate review, `riskTier: high` — and explicitly **out of scope** for this project. ✔
- Provenance on every assertion — mandatory `source` span + verbatim quote, enforced by an automated
  re-location check; no span ⇒ rejected. ✔

**Corrections made during review:**
- Reconciled risk tier: the project is **low-risk infrastructure** but biomarker/outcome extraction
  is **medium** (domain reviewer) — clarified the three-tier model so the roadmap "low" and the
  per-task `riskTier` values in TASKS.md are consistent.
- Ensured every success metric is quality- or reuse-anchored (removed raw-count vanity targets).
- Confirmed `verifiedNeed: false` everywhere a consumer is not yet secured, including the example JSON.
- Confirmed the deliverable enums used in TASKS.md (`document`/`dataset`/`pr`) match the schema and
  that no task emits patient-facing content.

**Outstanding human decisions (surfaced, not resolved):** name the License+PII reviewer and the
domain (oncology) reviewer; confirm the first consumer (internal KG project vs. external); confirm the
ND grounding-quote cap; choose the release venue (Zenodo proposed). These are tracked in *Open
questions* and gate `verifiedNeed`/shipping.

Status: **Draft v0.1.0 — ready for maintainer + reviewer review.**
