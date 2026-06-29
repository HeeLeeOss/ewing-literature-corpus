# TASKS — ewing-literature-corpus

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID from the tables (e.g. `ewing-literature-corpus-schema-002`).
- `title` — the table's Title.
- `project` — `ewing-literature-corpus`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all current tasks (no funded escrow). A funded extraction batch would set
  `lane: funded` and add `fundedBudgetUsd` (hard cap; see PLAN architecture).
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","ewing-sarcoma","open-access","text-mining"]`.
- `riskTier` — `low | medium | high`. Infra/ingestion/normalization = `low`; biomarker/outcome
  assertion extraction = `medium` (domain reviewer); patient-facing = `high` and **out of scope**.
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. Code → `pr`; schemas/docs/gate artifacts →
  `document`; corpus/extraction/gold-set data → `dataset`. We never deliver patient-facing content.
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until a named consumer (internal KG maintainer or external) confirms
  they will use the output.
- `verifiedNeed` — **`false`** until a named consumer confirms use (general need is real; per-output
  delivery need is unproven).
- `outputLicense` — `CC-BY-4.0` for our added content (schemas, extraction, annotations, datasheets);
  `MIT` for code. Per-article redistributed text retains its **original** license, recorded per record.

**Binding cancer guardrails apply to every task** (see PLAN *Data, licensing & compliance*): open-access
/ aggregate / de-identified data only; no identifiable patient data; no medical advice; provenance on
every assertion. Every per-article task requires its own committed license + PII gate artifact before
work proceeds — listing a task does not pre-approve any article.

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-literature-corpus-reviewers-001 | Name/secure License+PII reviewer and domain (oncology) reviewer (blocking gate roles) | research | small | low | document | — | Maintainer |
| ewing-literature-corpus-schema-002 | Corpus record schema + provenance model (BioC-compatible) | design-spec | small | low | document | — | Technical |
| ewing-literature-corpus-extract-schema-003 | Extraction (assertion) schema with mandatory source spans + ontology normalization | design-spec | medium | low | document | schema-002 | Technical |
| ewing-literature-corpus-gate-004 | License-bucket + PII/case-report triage gate (blocking) | design-spec | small | medium | document | — | License+PII |
| ewing-literature-corpus-query-005 | Ewing PMC retrieval query + recall/precision against a seed set | research | small | low | document | — | Domain |
| ewing-literature-corpus-ingest-006 | PMC OA ingestion tool (OA Web Service/BioC API) + license/provenance capture | code | medium | low | pr | schema-002, gate-004 | Technical |
| ewing-literature-corpus-pilot-007 | Pilot: ~25 OA articles curated end-to-end + entity-mention extraction, grounded + verified | data | medium | medium | dataset | schema-002, extract-schema-003, gate-004, query-005, ingest-006, reviewers-001 | License+PII, Technical, Domain |
| ewing-literature-corpus-outreach-008 | Consumer outreach (KG project, sarcoma labs, advocacy) + shortlist | research | small | low | document | — | Steward |

**Acceptance criteria — key tasks**

- **schema-002 (corpus record schema + provenance)**
  - [ ] Canonical record documents every field: `pmcid/pmid/doi`, `title/journal/pubDate/authors[]`
        (names as published; no contact PII), `license {id, spdx, url, redistributable,
        derivativesAllowed, commercialUseAllowed, snapshotRef}`, `oaStatus`, `provenance
        {source:"PMC-OA", retrievedAt, oaPackageVersion, attribution}`, `caseReport {isCaseReport,
        individualDetailPresent, disposition}`, `sections[] {type, charStart, charEnd, text?}`,
        `fullTextIncluded`, `specVersions {bioc, croissant}`.
  - [ ] Full text is represented only when `derivativesAllowed:true`; `sections[].text` is omitted
        otherwise (metadata-only records are valid).
  - [ ] BioC-compatible offset model documented; Croissant export defined as a projection of the record.
  - [ ] States our added content is CC-BY-4.0 and per-article text keeps its original license.

- **gate-004 (license + PII/case-report gate)**
  - [ ] Encodes the PMC OA license-bucket policy (CC0/CC-BY/CC-BY-SA/NC → full text + derivatives;
        ND → metadata + capped grounding quotes; unclear/non-OA → metadata only) as a fixed,
        pre-decided rule, not ad-hoc judgement.
  - [ ] Objective criterion: full text included only if in PMC OA **and** `derivativesAllowed:true`
        recorded from a cited license id/URL; missing/unparseable/ND ⇒ metadata-only. No default-allow.
  - [ ] PII/case-report methodology: article-type heuristic (`case-report`, "a N-year-old"),
        direct-identifier scan, rare-disease quasi-identifier flag (age+sex+geography+diagnosis date);
        any flag ⇒ aggregate-only or exclude; never re-identify.
  - [ ] Requires license snapshot (committed copy + SHA-256 + Wayback URL) and produces a committed
        PASS/METADATA-ONLY/EXCLUDE artifact per article recording which checks ran and what fired.
  - [ ] COSMIC/OncoKB cross-references limited to bare stable IDs (no redistribution); flagged.

- **ingest-006 (PMC OA ingestion tool)**
  - [ ] Retrieves articles **only** from the PMC OA Subset (OA Web Service / BioC API / OA packages);
        non-OA hits reduced to bibliographic metadata only.
  - [ ] Captures and records the machine-readable license, license URL, and snapshot per article.
  - [ ] Emits canonical records valid against schema-002; ships golden fixtures (synthetic/public)
        exercised in CI; no NCBI API key committed (env var only).
  - [ ] Code MIT-licensed; `pnpm build && pnpm test && pnpm lint` green; DCO signed-off.

- **pilot-007 (end-to-end pilot, ~25 articles)**
  - [ ] Pilot gated on a realistic consumption path first: the `ewsr1-fli1-knowledge-graph` maintainer
        agreeing to consume the output, or a Zenodo self-publish fallback (so a real released outcome
        exists).
  - [ ] All ~25 articles passed gate-004 with committed artifacts (license bucket + PII disposition).
  - [ ] Entity-mention extraction produced; **100% of accepted assertions pass the automated
        groundedness re-location check** (quote re-found at recorded offsets) and are human-verified;
        every assertion has `aggregateOnly:true`.
  - [ ] Provenance complete per article (PMCID/DOI, retrieval date, license id+URL+snapshot, attribution).
  - [ ] Released via the KG consumer or Zenodo fallback with a recorded outcome artifact
        (`outcomes/pilot.json`) — or submitted with the blocker surfaced if neither materializes.

**M0 Definition of Done:** License+PII reviewer and domain reviewer named (blocking roles filled before
pilot review); corpus + extraction schemas and the license/PII gate published; OA ingestion tool green
in CI with golden fixtures; ~25-article pilot curated end-to-end with grounded, verified entity-mention
extraction (automated groundedness check green, every assertion aggregate-only) and released via the KG
consumer or Zenodo fallback (outcome artifact recorded) — or submitted with the blocker surfaced; ≥ 1
consumer-outreach thread opened.

---

## Milestone M1 — Corpus build + license/PII gate hardened

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-literature-corpus-license-clf-009 | Automated license classifier + ND/NC routing | code | medium | medium | pr | ingest-006, gate-004 | License+PII, Technical |
| ewing-literature-corpus-dedup-010 | Dedup + versioning by PMCID/DOI | code | small | low | pr | ingest-006 | Technical |
| ewing-literature-corpus-pii-audit-011 | Case-report/PII audit pass over the corpus | research | medium | medium | document | pilot-007, gate-004 | License+PII |
| ewing-literature-corpus-build-012 | Curate corpus to ≥ 150 articles (gate-passed, provenanced) | data | large | medium | dataset | license-clf-009, dedup-010, pii-audit-011 | License+PII, Technical |
| ewing-literature-corpus-partner-013 | Secure first confirmed consumer/partner | research | small | low | document | outreach-008 | Steward |

**Acceptance criteria — key tasks**

- **license-clf-009 (automated license classifier)**
  - [ ] Classifies each article into the fixed PMC OA license bucket from its machine-readable license;
        routes ND/closed to metadata-only and CC0/BY/SA/NC to full text per gate-004.
  - [ ] Validated against a hand-labeled sample (≥ 50 articles); reports accuracy; any disagreement
        defaults to the more conservative bucket (metadata-only) pending human review.
  - [ ] Never upgrades a license bucket automatically; human reviewer confirms borderline cases.
  - [ ] Code MIT-licensed; tests + CI green; no credentials committed.

- **pii-audit-011 (case-report/PII audit)**
  - [ ] Every corpus article has a committed `caseReport` disposition (none / aggregate-only / excluded).
  - [ ] Rare-disease quasi-identifier methodology applied; any identifiable individual detail ⇒
        excluded or reduced to aggregate; no identifiable images ingested.
  - [ ] Audit output recorded; zero identifiable patient details remain in the corpus (verified by sample).

- **build-012 (corpus to ≥ 150 articles)**
  - [ ] ≥ 150 OA articles curated, each gate-passed with a committed artifact and complete provenance.
  - [ ] License mix recorded honestly (per-article original license retained; redistributable vs
        metadata-only counts reported).
  - [ ] Corpus validates against schema-002; dedup applied; CI green.

- **partner-013 (first confirmed consumer)**
  - [ ] A named consumer (KG maintainer or external lab/advocacy) confirms they will use the corpus.
  - [ ] Consumption mechanism documented (repo ingestion / DOI / direct handoff).
  - [ ] Tasks for that consumer updated to `verifiedNeed: true` with `requestor` set.

**M1 Definition of Done:** automated license classifier validated and routing ND/NC correctly; PII/case-
report audit complete with committed dispositions and zero identifiable detail; ≥ 150 articles curated
(gate-passed, provenanced, schema-valid); dedup/versioning working; ≥ 1 confirmed consumer.

---

## Milestone M2 — Structured extraction at scale + evaluation

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-literature-corpus-ner-014 | Grounded entity extraction (genes/fusions/cell lines/drugs/disease) | code | large | medium | pr | build-012, extract-schema-003 | Technical, Domain |
| ewing-literature-corpus-normalize-015 | Ontology normalization (HGNC/MONDO/Cellosaurus/ChEBI) + fusion vocab | code | medium | medium | pr | ner-014 | Technical, Domain |
| ewing-literature-corpus-gold-016 | Gold-standard annotated eval set + inter-annotator agreement | data | medium | medium | dataset | extract-schema-003 | Domain, Technical |
| ewing-literature-corpus-eval-017 | Extraction evaluation harness (P/R/F1 vs gold + groundedness check) | code | medium | low | pr | gold-016, ner-014, normalize-015 | Technical |
| ewing-literature-corpus-biomarker-018 | Aggregate biomarker/outcome assertion extraction (expert-verified, not advice) | data | large | medium | dataset | normalize-015, eval-017 | Domain, License+PII |

**Acceptance criteria — key tasks**

- **ner-014 (grounded entity extraction)**
  - [ ] Extracts entities (genes/proteins, EWSR1-ETS fusion types, cell lines, drugs, disease concepts,
        model organism, assay/method); **every output carries a verbatim source span** (PMCID +
        section + offsets + quote).
  - [ ] The deterministic groundedness re-location check passes for 100% of accepted outputs; ungrounded
        outputs are dropped (CI enforces this).
  - [ ] All assertions `aggregateOnly:true`; no individual-patient content extracted.
  - [ ] Code MIT-licensed; tests + CI green.

- **normalize-015 (ontology normalization)**
  - [ ] Maps entities to pinned ontologies (HGNC, MONDO/NCIt, Cellosaurus, ChEBI) recording
        `ontologyId + ontologyVersion + normConfidence`; fusion types mapped via the in-repo controlled
        vocabulary to HGNC ids; unmapped entities flagged `unmapped` (kept as surface forms).
  - [ ] Spot-check examples documented (e.g. Ewing sarcoma → MONDO:0012817; A673 → CVCL_0080).
  - [ ] Ontology versions recorded in `specVersions`; bumped only via a deliberate task.

- **gold-016 (gold-standard eval set)**
  - [ ] ≥ 2 annotators independently annotate a held-out article sample; inter-annotator agreement
        (Cohen's κ) computed and ≥ 0.7; adjudication process recorded.
  - [ ] Gold set covers entity types + assertion groundedness; released CC-BY-4.0 with provenance.

- **biomarker-018 (aggregate biomarker/outcome assertions)**
  - [ ] Only **aggregate** research/biomarker/outcome statements extracted, each with a source span and
        `aggregateOnly:true`; no patient-level data, no clinical advice, no synthesis.
  - [ ] ≥ 100 assertions **domain-expert verified** for scientific faithfulness, each with reviewer
        attribution + timestamp recorded in `verification`.
  - [ ] License/PII reviewer confirms no ND full text or identifiable detail used.

**M2 Definition of Done:** grounded entity extraction + ontology normalization running with the
groundedness check enforced in CI; gold set built (κ ≥ 0.7) and eval harness reporting entity-
normalization F1 ≥ 0.85; ≥ 400 articles curated cumulatively; ≥ 100 domain-expert-verified aggregate
assertions, each provenanced and aggregate-only.

---

## Milestone M3 — Release, reuse & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-literature-corpus-release-019 | Versioned corpus + extraction release (Zenodo DOI + Datasheet + Croissant/BioC) | data | medium | low | dataset | build-012, eval-017, biomarker-018 | License+PII, Technical |
| ewing-literature-corpus-reuse-020 | Track and verify downstream reuse events | research | small | low | document | release-019, partner-013 | Steward |
| ewing-literature-corpus-refresh-021 | Refresh/staleness + retraction-check process | maintenance | small | low | document | ingest-006, build-012 | Maintainer |

**Acceptance criteria — key tasks**

- **release-019 (versioned release)**
  - [ ] Corpus + extraction released as a versioned, citable artifact (Zenodo DOI) with a Datasheet
        (motivation, composition, collection, license mix, PII stance, limitations) and Croissant/BioC
        export.
  - [ ] Mixed-license labeling correct per record; our added content CC-BY-4.0; code MIT; no ND full
        text or identifiable detail present (final license+PII sign-off recorded).
  - [ ] Eval report for the release attached (F1, κ, groundedness pass rate).

- **reuse-020 (downstream reuse tracking)**
  - [ ] ≥ 1 verifiable external/downstream reuse event recorded (e.g. `ewsr1-fli1-knowledge-graph`
        commit ingesting the corpus, a citation, or written consumer confirmation).
  - [ ] Each event links to externally verifiable evidence (no self-reported reuse).

- **refresh-021 (refresh + retraction check)**
  - [ ] Documented process to re-query PMC OA for new/updated Ewing articles and to check PubMed for
        retractions; retracted articles flagged/removed.
  - [ ] Staleness flagged against recorded versions; updates handled as versioned releases.

**M3 Definition of Done:** versioned corpus + extraction released with Datasheet + Croissant/BioC export
and a DOI; ≥ 1 verifiable downstream reuse event; documented refresh/retraction process; maintainer +
steward identified for ongoing upkeep.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| ewing-literature-corpus-relation-022 | Relation extraction (gene-drug, fusion-phenotype), grounded | code | large | medium | pr | After entity extraction stable; needs domain review |
| ewing-literature-corpus-section-023 | IMRaD/section structure parser + per-section indexing | code | medium | low | pr | Improves extraction targeting + grounding |
| ewing-literature-corpus-api-024 | Read-only query interface over the corpus (no PII, no advice) | code | medium | low | pr | Helps consumers query by entity/assertion |
| ewing-literature-corpus-datasheet-i18n-025 | Translate the corpus Datasheet/summary (domain reviewer) | translation | small | medium | translation | Widens reuse; needs language + domain reviewer |
| ewing-literature-corpus-funded-batch-026 | Funded large-batch extraction run (hard budget cap) | data | large | medium | dataset | `lane: funded` + `fundedBudgetUsd`; only if donated throughput insufficient |

---

## Example task JSON

The first build task in M0 (`schema-002`), as a complete, schema-valid Task JSON:

```json
{
  "id": "ewing-literature-corpus-schema-002",
  "title": "Corpus record schema + provenance model (BioC-compatible)",
  "project": "ewing-literature-corpus",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "ewing-sarcoma", "open-access", "text-mining", "open-science"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Ewing sarcoma is a rare, aggressive pediatric/AYA cancer whose research is scattered across thousands of papers. Before ingesting or extracting anything, the project needs one canonical corpus record schema that captures bibliographic metadata, per-article machine-readable license, PII/case-report disposition, IMRaD section structure with BioC-compatible offsets, and complete provenance. Binding guardrails: open-access (PMC OA Subset) data only; no identifiable patient data; no medical advice; provenance on every record. Full text is represented only when the license permits derivatives; ND/closed articles are metadata-only.",
  "objective": "Define the canonical corpus record schema and provenance model that all corpus outputs (BioC export, Croissant metadata, extraction layer) are projections of.",
  "acceptanceCriteria": [
    "Canonical record documents every field: pmcid/pmid/doi, title/journal/pubDate/authors[] (names as published; no contact PII), license {id, spdx, url, redistributable, derivativesAllowed, commercialUseAllowed, snapshotRef}, oaStatus, provenance {source, retrievedAt, oaPackageVersion, attribution}, caseReport {isCaseReport, individualDetailPresent, disposition}, sections[] {type, charStart, charEnd, text?}, fullTextIncluded, specVersions {bioc, croissant}.",
    "Full text (sections[].text) is represented only when license.derivativesAllowed is true; metadata-only records (text omitted) are valid and explicitly supported for ND/closed articles.",
    "BioC-compatible character-offset model is documented; Croissant export is defined as a projection of the canonical record.",
    "License snapshot format specified (committed copy + SHA-256 + Wayback URL) and required in provenance.",
    "Schema states our added content is licensed CC-BY-4.0 while per-article redistributed text retains its original license, recorded per record; at least one worked example record is included.",
    "pnpm build && pnpm test && pnpm lint pass for any committed tooling/JSON-schema; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\elyos\\planning\\projects\\ewing-literature-corpus\\PLAN.md",
    "C:\\code\\elyos\\planning\\ROADMAP.md",
    "PubMed Central Open Access Subset documentation (license terms; OA Web Service; BioC API)",
    "BioC text-mining interchange format specification",
    "Croissant ML metadata specification"
  ],
  "output": "A corpus record schema definition plus provenance model, committed to the project repo and ready for reuse by the ingestion and extraction tasks.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Task rollup

- **21 scheduled tasks** across M0–M3 (M0: 8 · M1: 5 · M2: 5 · M3: 3) + **5 backlog** tasks = 26 total.
- By type: design-spec 3 · code 8 · data 6 · research 5 · writing 0 · maintenance 1 · translation 1
  (+ funded-batch data in backlog).
- Risk: most tasks `low` (infra/ingestion/normalization); `medium` on license classification,
  PII audit, and all biomarker/outcome assertion extraction (domain-reviewer-gated). No `high`-risk
  task — patient-facing output is out of scope.
- Every task starts `verifiedNeed: false`, `requestor: TO BE SECURED`; flips to `true` only when a
  named consumer (KG maintainer or external) confirms use (`partner-013`).
- Hard gates before any per-article work: License+PII reviewer named (`reviewers-001`) and a committed
  `gate-004` artifact per article.
