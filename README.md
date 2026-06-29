# ewing-literature-corpus

> Ewing sarcoma research is published across thousands of articles, but the knowledge is locked in unstructured prose, inconsistent terminology, and paywalls. Researchers — especially the small labs and  ·  **Risk tier:** med  ·  **Status:** planning

Ewing sarcoma research is published across thousands of articles, but the knowledge is locked in unstructured prose, inconsistent terminology, and paywalls. Researchers — especially the small labs and trainees who do much of the rare-cancer work — spend disproportionate effort just *finding* and *reconciling* what is already known about EWSR1-FLI1/ETS biology, cell-line models, candidate targets, and aggregate clinical outcomes. There is no clean, openly-licensed, machine-readable, fully-provenanced corpus of Ewing literature that downstream projects (e.g. a knowledge graph, a reanalysis effort, or a systematic review) can build on without redoing the licensing and extraction work from scratch.

**Definition of shipped:** passed the license + PII gate with a committed artifact; (2) every accepted assertion passes the automated groundedness check and carries required-tier review sign-off (domain reviewer for medium-risk assertions); (3) an eval run against the gold set meets the quality targets; (4

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/ewing-literature-corpus --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
