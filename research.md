# Research & Benchmark Track (Core ML/AI Artifact)

> Originated in the [2026-06-19 call with G.J.](Meeting_Notes/feedback_2026-06-19.md) (MLE).
> This is the *first* thing we build — before the product — and it ships as a **public,
> open-source GitHub repo with a results README** (not, initially, a paper). It is also the
> evaluation harness [inference.md §9](inference.md) already calls for, just built up front
> and in the open. See [roadmap.md](roadmap.md) for how this phase feeds the product.

## Why research first

Three reasons it leads the roadmap instead of trailing it:

1. **It solves the golden-set cold start.** We need labeled `attempt → misconception` pairs to
   know if inference works ([inference.md §9](inference.md)). Hand-labeling those with nursing
   experts is slow and expensive. A **synthetic student with a programmed misconception gives
   us ground-truth labels for free, at scale** — see §1.
2. **It de-risks the product before we build it.** The benchmark tells us which model to use per
   pipeline step and how good our inference actually is, so the MVP arrives with pre-tuned
   prompts instead of guesses.
3. **It is the cheapest artifact that throws off real ML signal *and* community recognition.**
   An open benchmark repo with a leaderboard earns GitHub stars and demonstrates ML/AI depth
   without a product launch or a paper deadline.

The whole track reuses **one engine** (synthetic student + judge) that later tunes the product
([roadmap.md Phase 2](roadmap.md)) and validates efficacy ([roadmap.md Phase 3](roadmap.md)).

---

## The two ground truths (the core design)

The benchmark closes a loop using two independent sources of truth:

- **Synthetic student → ground truth on the *input* side.** We *program* the misconception, so we
  know the right answer to "what was this student confused about?"
- **LLM judge → scalable scoring on the *output* side.** Validated against a small human-labeled
  set, then used to score thousands of runs cheaply.

Together they make the eval re-runnable on every prompt/model change — exactly the requirement in
[inference.md §9](inference.md).

---

## 1. The calibrated synthetic student (the centerpiece)

A "student agent" parameterized by two axes:

- **Knowledge level** — secondary / undergraduate / professional (G.J.'s calibration axis), tuning
  the agent from *terrible → ok → good → great* at the subject.
- **Seeded misconception set** — a specific, named confusion we plant (e.g. *"attributes the
  arrhythmia-driving role to calcium instead of potassium"*).

The agent takes an MCQ exam and emits **not just an answer but a cross-off trace consistent with
its competence** — which is what makes it useful for testing our keystone signal
([decisions.md §3.3](decisions.md), [scenario.md](scenario.md)):

| Agent behavior | Produces | Maps to |
|---|---|---|
| Sharp but imperfect: leaves correct answer in, picks the near-neighbor distractor | `discrimination_failure` trace | [scenario.md](scenario.md) case 1 |
| Genuinely confused: crosses off the *correct* answer, deliberates between two wrong ones | `deeper_gap` trace | [scenario.md](scenario.md) case 2 |
| Careless / rushed: large random stuck-between set, fast time | `likely_slip` trace | [inference.md §3](inference.md) |
| Confident-but-wrong: no cross-off, picks wrong | `no_signal` trace | [decisions.md §4.1](decisions.md) |

Because **we programmed the misconception**, every emitted attempt comes with a free ground-truth
label. We then run ConceptLoop's real inference pipeline ([inference.md §4-5](inference.md)) on the
attempt and measure whether it **recovered the planted misconception.**

### What this benchmarks (inference quality)

- **Misconception-recovery accuracy** — did we name the confusion we planted?
- **Confidence calibration** — does `confidence` track when it's actually right? (the
  anti-hallucination valve, [inference.md §5](inference.md))
- **False-misconception rate** — how often do we fabricate a confusion on a `likely_slip` or
  `no_signal` agent? (should be near zero)
- **miss_type classification accuracy** — `discrimination_failure` vs `deeper_gap` vs `slip` vs
  `no_signal` ([inference.md §3](inference.md)).

### Why it's novel (worth a writeup later)

No public benchmark evaluates **misconception inference from MCQ *elimination behavior*** using a
calibrated synthetic student. If we choose to write it up later (optional, not the goal of this
phase), the natural venues are workshop-tier: **BEA** (Building Educational Applications),
**AIED**, **EDM**, or **L@S**. Kept as an option, not a deadline — see
[roadmap.md](roadmap.md).

---

## 2. The regeneration leaderboard

Fix a Misconception Object; have N candidate models each generate the variant question; score and
rank them. Output a **cost-vs-quality Pareto leaderboard, open-source models vs frontier.**

Per generated variant, the judge (§3) scores:

- **Targets the misconception** — does the new question actually force the planted confusion?
- **Single defensible answer** given the user-supplied key ([decisions.md §4.2](decisions.md)).
- **Anti-duplication** — embedding distance from the original; reject near-rewordings
  ([decisions.md §3.4](decisions.md), [inference.md §8](inference.md)).
- **Difficulty match** — is the variant calibrated to the agent's level, not trivially easy/hard?

**Double duty:** the leaderboard is a star-earning public artifact *and* it directly resolves the
product's model-tiering cost question ([business.md "defining cost reality"](business.md)). It also
backs G.J.'s bring-your-own-key point ([feedback](Meeting_Notes/feedback_2026-06-19.md)): we can
tell a self-hosting user "this open model gets you ~85% of frontier quality at $0," which is both a
research finding and a sales line. Benchmark both **locally hosted** and **frontier (cloud API)**
models.

---

## 3. The LLM judge

The connective tissue, and a method we've run before (the Schedulater Bot paper). Two-step:

1. **Validate the judge against a small human-labeled set** — our pilot humans
   ([target.md](target.md): the owner's sister, then UAlberta nursing students) label a few dozen
   cases; we measure judge-vs-human agreement to establish the judge is trustworthy.
2. **Then scale scoring with the judge** across thousands of synthetic runs the judge can score but
   a human never could.

This is what makes §1 and §2 cheap to run continuously instead of once.

---

## 4. What ships in this phase

A standalone open-source repo (can live in this repo or a sibling) containing:

- The synthetic-student generator (knowledge-level × seeded-misconception → attempt + cross-off
  trace).
- A harness that runs the [inference.md](inference.md) pipeline over generated attempts and scores
  recovery.
- The regeneration leaderboard runner + results tables (open-source vs frontier, cost vs quality).
- The judge, with its human-agreement validation numbers.
- A **results README** — the public face: leaderboards, accuracy/calibration tables, method writeup.

Success for this phase is **a credible public results README**, not a paper and not a product.

---

## 5. How this feeds everything downstream

- **Product MVP** ([roadmap.md Phase 2](roadmap.md)) — inherits tuned prompts and a chosen
  per-step model tier; the eval harness becomes the regression guard on every prompt change.
- **Efficacy study** ([roadmap.md Phase 3](roadmap.md)) — the same judge + metrics infrastructure
  measures real pilot users vs non-users.
- **B2B2C credibility** ([business.md](business.md)) — "here is our measured inference accuracy and
  false-misconception rate" is the pitch to nursing programs, per
  [inference.md §9](inference.md).
