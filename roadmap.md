# Roadmap — Research-Led, Product-Bound

> Direction chosen 2026-06-19 after the [call with G.J.](Meeting_Notes/feedback_2026-06-19.md).
> The goal is to hit three career markers — **ML/AI experience, community recognition, and
> startup experience** — with one coherent line of work rather than three separate bets.
> The unifying insight: the research/eval track is a *prerequisite* for the product, so we build
> it first and in public, and each phase produces a shareable artifact that feeds the next.

## The decision

We considered three paths and how each serves the three goals:

| Path | Startup exp | ML/AI exp | Community recognition | Speed to artifact | Keeps the cross-off IP? |
|---|---|---|---|---|---|
| Standalone product (B2C/B2B2C) | ★★★ | ★ | ★ | Slow (months) | Yes |
| Plugin (e.g. Anki) | ★ | ★ | ★★ | Medium | **No** — see note |
| Research / benchmark | ★ | ★★★ | ★★★ | **Fast (weeks)** | Yes |

**Chosen: research-led, product-bound** — sequence all three so each phase ships a public artifact
and de-risks the next. We lead with research because it is fastest to a shareable result, it *is*
the ML experience, and the product can't be built well without it anyway.

> **Plugin caveat (why it's not a primary path).** Anki is a *flashcard* tool with no cross-off /
> elimination interaction — but the cross-off trace is our keystone signal
> ([decisions.md §3.3](decisions.md), [scenario.md](scenario.md)). An Anki plugin would force us to
> discard the one thing that makes ConceptLoop different. The plugin is therefore an *optional*
> distribution experiment (Phase 2.5), only worthwhile if a host with real MCQ + elimination
> semantics exists — not a substitute for the standalone product.

---

## The throughline

One engine — the **synthetic student + LLM judge** ([research.md](research.md)) — pays out in all
three currencies:

```
Phase 1  Public benchmark  ──►  ML/AI exp + GitHub stars + de-risked product
   │        (synthetic student + leaderboard + judge)
   ▼
Phase 2  Product MVP       ──►  startup exp begins; prompts arrive pre-tuned by the benchmark
   │        (the existing 1–2 week plan)
   ▼
Phase 3  Efficacy study    ──►  startup proof + 2nd research output + B2B2C sales evidence
            (uni-club pilots, users vs non-users)
```

---

## Phase 1 — Public benchmark & eval harness  ·  ~2–4 weeks

**Goal:** a credible open-source repo with a results README. Detailed design in
[research.md](research.md).

- Build the calibrated synthetic student (knowledge level × seeded misconception → attempt +
  cross-off trace).
- Run the [inference.md](inference.md) pipeline over generated attempts; measure
  misconception-recovery accuracy, confidence calibration, false-misconception rate, and
  `miss_type` classification accuracy.
- Build the regeneration leaderboard (open-source vs frontier; cost vs quality).
- Build + human-validate the judge (agreement against a small hand-labeled set).
- Ship the **results README**.

**Markers hit:** ML/AI experience, community recognition (stars). **Output artifact:** the repo.
**Exit criteria:** published repo with leaderboard + accuracy/calibration tables, and a chosen
per-step model tier to carry into Phase 2.

> Optional, not a deadline: a workshop/arXiv writeup (BEA / AIED / EDM / L@S). We chose the
> GitHub-repo ambition for now ([research.md §1](research.md)); revisit only if traction warrants.

## Phase 2 — Product MVP  ·  the existing 1–2 week plan

**Goal:** get one real nursing student through the full loop ([decisions.md §3.8](decisions.md)).

- Build per [decisions.md §3.8](decisions.md) scope: cross-off exam UI → navigation → submit/grade
  → missed-question follow-up page (inference + regenerated variant).
- Prompts and model tiers arrive **pre-tuned from Phase 1**; the Phase 1 harness becomes the
  regression guard on every prompt change.
- Run the first pilot ([target.md](target.md): the owner's sister, NCLEX prep).

**Markers hit:** startup experience begins (real product, real user). **Exit criteria:** the
[target.md](target.md) pilot questions answered — does cross-off get used, does a regenerated
question feel like it tested understanding, is the inferred misconception useful?

### Phase 2.5 (optional) — Plugin distribution experiment

Only if a host with real MCQ + elimination semantics exists (Anki does **not** — see caveat above).
Treat as a growth hack outside the critical path, not a pivot.

## Phase 3 — Distribution + efficacy study → B2B2C

**Goal:** turn the product into a startup by proving impact and converting to institutional buyers.

- Pilot with the uni clubs G.J. named — **UAlberta, MacEwan, Concordia, NAIT**
  ([feedback](Meeting_Notes/feedback_2026-06-19.md)) — plus the
  [target.md](target.md) pilot pipeline.
- Run a **users-vs-non-users correlation study**; log per-item response times and other engagement
  signals ([feedback](Meeting_Notes/feedback_2026-06-19.md)). This study is simultaneously a 2nd
  research output, B2B2C sales proof, and the startup-validation milestone.
- Convert proof into the B2B2C motion in priority order ([business.md](business.md)): nursing
  programs → smaller NCLEX prep companies → professional licensing bodies.

**Markers hit:** startup experience (validated), community recognition (efficacy data), ML/AI
experience (study). **Exit criteria:** a measured efficacy signal credible enough to open a B2B2C
conversation.

---

## How each marker gets hit (summary)

| Marker | Where it comes from |
|---|---|
| **ML/AI experience** | Phase 1 benchmark engine; Phase 2 pipeline tuning; Phase 3 study design |
| **Community recognition** | Phase 1 public repo + leaderboard (stars); Phase 3 efficacy data; optional writeup |
| **Startup experience** | Phase 2 real product + pilot; Phase 3 efficacy proof → B2B2C |
